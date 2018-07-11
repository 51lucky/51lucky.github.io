---
title: java基本类型与包装类
abbrlink: 61d7e5c9
date: 2018-07-11 16:23:33
tags:
---
## 基本数据类型

java中的基本数据类型有`byte`,`char`,`boolean`,`short`,`int`,`long`, `float`,`double`
* `byte`：1个字节，最大存储数据量是255，存放的数据范围是-128~127之间。
* `short`：2个字节，最大数据存储量是65536，数据范围是-32768~32767之间。
* `int`：4个字节，最大数据存储容量是2的32次方减1，数据范围是负的2的31次方到正的2的31次方减1。
* `long`：8个字节，最大数据存储容量是2的64次方减1，数据范围为负的2的63次方到正的2的63次方减1。
* `float`：4个字节，数据范围在3.4e-45~1.4e38，直接赋值时必须在数字后加上f或F，**float的精度为6-7位有效数字**。
* `double`：8个字节，数据范围在4.9e-324~1.8e308，赋值时可以加d或D也可以不加（默认double类型），**double的精度位15-16位有效数字**。
* `boolean`：1个字节 只有true和false两个取值。
* `char`：2个字节，存储Unicode码，用单引号赋值。

数据类型转换：

  ![类型转换](/images/java/java_basic_data_type.png)

<!-- more -->

## 基本数据类型包装类

java为每一个基本数据类型都引入了对应的包装类型（wrapper class）,并从java 5开始引入了自动装箱/拆箱机制，使得二者可以相互转换。

包装类类型：`Byte`,`Character`,`Boolean`,`Short`,`Integer`,`Long`,`Float`,`Double`

### 自动装箱:将基本数据类型重新转化为对象
```java
public class Test {  
    public static void main(String[] args) {  
        //声明一个Integer对象
        Integer num = 9;

        //以上的声明就是用到了自动的装箱：解析为:Integer num = new Integer(9);
    }  
}
```
9是属于基本数据类型的，原则上它是不能直接赋值给一个对象Integer的，但jdk1.5后你就可以进行这样的声明。自动将基本数据类型转化为对应的封装类型，成为一个对象以后就可以调用对象所声明的所有的方法。

### 自动拆箱:将对象重新转化为基本数据类型

```java
public class Test {  
    public static void main(String[] args) {  
        //声明一个Integer对象
        Integer num = 9;

        //进行计算时隐含的有自动拆箱
        System.out.print(num--);
    }  
}  
```

### 使用场景

* 一般POJO类中使用包装类型，而在本地变量中推荐使用基本类型
* 使用容器时，只能存储对象，不能存储基本数据类型（List,Set,Map）
* 考试成绩为0和缺考的区别（用Integer可以，int不行）

## Integer与int的区别

###  Integer与int的基本使用对比

1. Integer是int的包装类，int则是java的一种基本数据类型
2. Integer变量必须实例化后才能使用，而int变量不需要
3. Integer实际是对象的引用，指向new出的Integer对象，而int则直接存储数据值
4. Integer的默认值是null，int的默认值是0

### Integer与int的深入对比
1. 由于Integer变量实际上是对一个Integer对象的引用，所以两个通过new生成的Integer变量永远是不相等的（因为new生成的是两个对象，其内存地址不同）。
```java
Integer i = new Integer(100);
Integer j = new Integer(100);
System.out.print(i == j); //false
```

2. Integer变量和int变量比较时，只要两个变量的值时相等的，则结果为true(因为包装类Integer和基本数据类型int比较时，java会自动拆包为int，然后进行比较，实际上就变为两个int变量的比较)。
```java
Integer i = new Integer(100);
int j = 100；
System.out.print(i == j); //true
```

3. 非new生成的Integer变量和new Integer()生成的变量比较时，结果为false。（因为非new生成的Integer变量指向的是`java常量池中的对象`,而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同）
```java
Integer i = new Integer(1000);
Integer j = 1000;
System.out.print(i == j); //false
```
4. 对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间-128到127之间，则比较结果为true,如果两个变量的值不在此区间，则比较的结果为false。
```java
public static void main (String[] args) {
    Integer i = 100;
    Integer j = 100;

    System.out.print(i == j); //true
    Integer i = 128;
    Integer j = 128;
    System.out.print(i == j); //false
}
```
  原因:java在编译Integer i = 100 ;时，会翻译成为Integer i = Integer.valueOf(100)。而java API中对Integer类型的valueOf的定义如下，对于-128到127之间的数，会进行缓存，Integer i = 127时，会将127进行缓存，下次再写Integer j = 127时，就会直接从缓存中取，就不会new了。  
```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

### Integer源码分析

给一个Integer对象赋一个int值的时候，会调用Integer类的静态方法valueOf，源码如下：
```java
public static Integer valueOf(String s, int radix) throws NumberFormatException {
    return Integer.valueOf(parseInt(s,radix));
}

public static Integer valueOf(String s) throws NumberFormatException {
    return Integer.valueOf(parseInt(s, 10));
}

public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```
java对于Integer与int的自动装箱与拆箱的设计，是一种模式：叫享元模式（flyweight）。

加大对简单数字的重利用，Java定义在自动装箱时对于值从–128到127之间的值，它们被装箱为Integer对象后，会存在内存中被重用，始终只存在一个对象。

而如果超过了从–128到127之间的值，被装箱后的Integer对象并不会被重用，即相当于每次装箱时都新建一个 Integer对象。

IntegerCache是Integer的内部类，源码如下:
```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {                  // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```
