---
title: Volley源码分析
tags: Volley
abbrlink: 584afc42
date: 2018-05-15 14:56:22
---
## Volley 简介
Volley是Google推出的Android异步网络请求框架和图片加载框架。在Google I/O 2013大会上发布。

Volley的特点:

1. 支持多个并发网络请求。
2. 支持请求优先级。
3. 支持网络请求缓存。
4. 支持取消网络请求（单个or一组）。
5. 容易自定义/扩展。

所以，Volley特别适合数据量小，通信频繁的网络应用开发,由于Volley会将Response解析到内存中，所以不适合大数据下载。

<!-- more -->

## 总体设计

### 总体设计图

![设计图](/images/volley/design.png)

Volley主要是通过两种`Dispatch Thread`不断从`Request Queue`中取出请求，根据是否已缓存调用`Cache`或`Network`这两个类的数据获取接口，从内存缓存或是服务器取得请求的数据，然后交由`ResponseDelivery`去做结果分发及回调处理。

### Volley中关键类介绍

Volley的调用比较简单，通过newRequestQueue(...)函数新建并启动一个请求队列`RequestQueue`后，只需要往这个`RequestQueue`不断add Request即可。

**Volley**:volley对外暴露的API，通过newRequestQueue(...)函数新建并启动一个请求队列`RequestQueue`。

**Request**:表示一个请求的抽象类。`StringRequest`,`JsonRequest`,`ImageRequest`都是它的子类，表示某种类型的请求。

**RequestQueue**:表示请求队列，里面包含一个`CacheDispatcher`(用于处理走缓存请求的调度线程)、`NetworkDispatcher`数组(用于处理走网络请求的调度线程)，一个`ResponseDelivery`(返回结果分发接口),通过start()函数启动时会启动`NetworkDispatcher`，`CacheDispatcher`。

**CacheDispatcher**:一个线程，用于调度处理走缓存的请求。启动后会不断从缓存请求队列中取请求处理，队列为空则等待，请求处理结束则将结果传递给`ResponseDelivery`去执行后续处理。当结果未缓存过，缓存失效或缓存需要刷新的情况下，该请求都需要重新进入`NetworkDispatcher`去调度请求。

**NetworkDispatcher**:一个线程，用于调度处理走网络的请求。启动后不断从网络请求队列中取请求处理，队列为空则等待，请求处理结束则将结果传递给`ResponseDelivery`去执行后续处理，并判断结果是否要进行缓存。

**ResponseDelivery**:返回结果分发，目前只有基于`ExecutorDelivery`的，在入参handler对应线程内进行分发。

**BaseHttpStack**:处理Http请求，返回请求结果。目前Volley中有基于HttpUrlConnection的`HurlStack`和基于Apache HttpClient的`HttpClientStack`。由于目前不需要考虑API<9的情况，故`HttpClientStack`不需要考虑。`HttpClientStack`通过`AdaptedHttpStack`转换为`BaseHttpStack`。

**Network**:调用`HttpStack`处理请求，并将结果转换为可被`ResponseDelivery`处理的`NetworkResponse`。

**Cache**:缓存请求结果，Volley默认使用的是基于Cache目录的`DiskBasedCache`。`NetworkDispatcher`得到请求结果后判断是否需要存储在Cache，`CacheDispatcher`会从Cache中取缓存结果。

## 请求流程图

![请求流程图](/images/volley/Volley-run-flow-chart.png)

## 详细设计

### 核心类功能介绍

#### Volley.java

这个和Volley框架同名的类，其实是个工具类，作用是构建一个可用于添加网络请求的RequestQueue对象。

Volley.java有两个重载的静态方法

```java
public static RequestQueue newRequestQueue(Context context)

public static RequestQueue newRequestQueue(Context context, BaseHttpStack stack)
```
第一个方法的实现调用了第二个方法，通过BaseHttpStack创建一个代表网络(Network)的具体实现`BaseNetwork`，接着构造一个代表缓存(Cache)的基于Disk的具体实现`DiskBasedCache`。最后将网络(Network)对象和缓存(Cache)对象传入构建一个`RequestQueue`，启动这个RequestQueue，并返回。

```java
File cacheDir = new File(context.getCacheDir(),DEFAULT_CACHE_DIR);
RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
queue.start();
return queue;
```

我们平时大多采用`Volley.newRequestQueue(context)`的默认实现，构建RequestQueue。通过源码可以看出，我们可以抛开Volley工具类构建自定义的RequestQueue，采用自定义的`BaseHttpStack`，采用自定义的`Network`实现，采用自定义的Cache实现等来构建`RequestQueue`。**优秀框架的高可拓展性的魅力来源于此啊**

#### Request.java

代表一个网络请求的抽象类。我们通过构建一个`Request`类的非抽象子类(`StringRequest`,`JsonRequest`,`ImageRequest`或自定义)对象，并将其加入到`RequestQueue`中来完成一次网络请求操作。Volley支持8种Http请求方式**GET, POST, PUT, DELETE, HEAD, OPTIONS, TRACE, PATCH**。Request类中包含了请求url,请求方式，请求Header，请求Body，请求的优先级等信息。

**因为是抽象类，子类必须重写的两个方法。**

```java
abstract protected Response<T> parseNetworkResponse(NetworkResponse response);
```
子类重写此方法，将网络返回的原生字节内容，转换成合适的类型。此方法会在**工作线程**中被调用。
```java
abstract protected void deliverResponse(T response);
```
子类重写此方法，将解析成合适类型的内容传递给他们的监听回调。

**以下两个方法也经常会被重写**

```java
public byte[] getBody()
```
重写此方法，可以构建用于**POST, PUT, PATCH**请求方式的Body内容。
```java
protected Map<String, String> getParams()
```
在上面`getBody`函数没有被重写情况下，此方法的返回值会被key，value分别编码后拼接起来转换为字节码作为Body内容。

#### RequestQueue.java

Volley框架的核心类，将请求Request加入到一个运行的`RequestQueue`中，来完成请求操作。

* 主要成员变量

RequestQueue中维护了两个**基于优先级**的Request队列，缓存请求队列和网络请求队列。

```java
private final PriorityBlockingQueue<Request<?>> mCacheQueue = new PriorityBlockingQueue<>();
private final PriorityBlockingQueue<Request<?>> mNetworkQueue = new PriorityBlockingQueue<>();
```

维护了一个正在进行中，尚未完成的请求集合

```java
private final Set<Request<?>> mCurrentRequests = new HashSet<>();
```

* 启动队列

创建出RequestQueue以后，调用start()方法，启动队列。

```java
/** Starts the dispatchers in this queue. */
public void start() {
    stop(); // Make sure any currently running dispatchers are stopped.
    // Create the cache dispatcher and start it.
    mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
    mCacheDispatcher.start();

    // Create network dispatchers (and corresponding threads) up to the pool size.
    for (int i = 0; i < mDispatchers.length; i++) {
        NetworkDispatcher networkDispatcher =
                new NetworkDispatcher(mNetworkQueue, mNetwork, mCache, mDelivery);
        mDispatchers[i] = networkDispatcher;
        networkDispatcher.start();
    }
}
```

start()方法中，开启一个**缓存调度线程`CacheDispatcher`** 和4个 **网络调度线程`NetworkDispatcher`**。
缓存调度线程不断的从缓存请求队列中取出Request去处理，网络调度线程不断的从网络请求队列中取出Rqeust去处理。

* 处理流程图

![RequestQueue流程图](/images/volley/request_queue.png)

#### CacheDispatcher.java

一个线程，用于调度处理走缓存的请求。启动后会不断从缓存请求队列中取请求处理，队列为空则等待，请求处理结束则将结果传递给`ResponseDelivery`去执行后续处理。当结果未缓存过、缓存失效或缓存需要刷新的情况下，该请求都需要重新进入`NetworkDispatcher`去调度处理。

* 主要成员变量

`BlockingQueue<Request<?>> mCacheQueue;`缓存请求队列

`BlockingQueue<Request<?>> mNetworkQueue;`网络请求队列

`Cache mCache;`缓存类，代表了一个可以获取请求结果，存储请求结果的缓存

`ResponseDelivery mDelivery;`请求结果传递类

`WaitingRequestManager mWaitingRequestManager`

* 处理流程图

![CacheDispatcher流程图](/images/volley/cache_dispatcher.png)

#### NetworkDispatcher.java

一个线程，用于调度处理走网络的请求。启动后会不断从网络请求队列中请求处理，队列为空则等待，请求处理结束则将结果传递给`RespondeDelivery`去执行后续处理。并判断结果是否要进行缓存。

* 主要成员变量

`BlockingQueue<Request<?>> mQueue` 网络请求队列

`Network mNetwork` 网络类，代表了一个可以执行请求的网络

`Cache mCache` 缓存类，代表了一个可以获取请求结果，存储请求结果的缓存

`ResponseDelivery mDelivery` 请求结果传递类，可以传递请求的结果或者错误到调用者

* 处理流程

![NetworkDispatcher流程图](/images/volley/NetworkDispatcher-run-flow-chart.png)
