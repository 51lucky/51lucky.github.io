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

**HttpStack**:处理Http请求，返回请求结果。目前Volley中有基于HttpUrlConnection的`HurlStack`和基于Apache HttpClient的`HttpClientStack`。

**Network**:调用`HttpStack`处理请求，并将结果转换为可被`ResponseDelivery`处理的`NetworkResponse`。

**Cache**:缓存请求结果，Volley默认使用的是基于Cache目录的`DiskBasedCache`。`NetworkDispatcher`得到请求结果后判断是否需要存储在Cache，`CacheDispatcher`会从Cache中取缓存结果。

## 请求流程图

![请求流程图](/images/volley/Volley-run-flow-chart.png)

## 详细设计

### 类关系图
