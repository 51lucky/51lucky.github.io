---
title: String、StringBuffer与StringBuilder的区别
abbrlink: 272f3f11
date: 2018-07-12 16:56:31
tags:
---
## String类

### String类基础知识

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```
通过查看String类源码（jdk1.8）可以归纳出：
1. String类被final修饰符修饰，所以String类不能被继承。它的成员方法都默认是final方法。
2. 对String对象的任何改变都不影响到原对象， 相关的任何change操作都会生成新的对象。

  String类是通过char数组来保存字符串的，而String类的一些方法（substring,replace,concat）等操作都不是在原有的字符串上进行的，而是重新生成了一个新的字符串对象。也就是说进行这些操作后，最原始的字符串并没有被改变。

<!-- more -->

### 深入理解String类

```java
public class Main {     
    public static void main(String[] args) {
        String str1 = "hello world";
        String str2 = new String("hello world");
        String str3 = "hello world";
        String str4 = new String("hello world");

        System.out.println(str1==str2); //false
        System.out.println(str1==str3); //true
        System.out.println(str2==str4); //false
    }
}
```

在上述代码中，`String str1 = "hello world";`和`String str3 = "hello world";`都在编译期间生成了`字面常量`和`符号引用`，运行期间字面常量`"hello world"`被存储在运行时常量池（当然只保存了一份）。通过这种方式来将String对象跟引用绑定的话，JVM执行引擎会先在运行时常量池查找是否存在相同的字面常量，如果存在，则直接将引用指向已经存在的字面常量；否则在运行时常量池开辟一个空间来存储该字面常量，并将引用指向该字面常量。

通过new关键字来生成对象是在堆区进行的，而在堆区进行对象生成的过程是不会去检测该对象是否已经存在的。因此通过new来创建对象，创建出的一定是不同的对象，即使字符串的内容是相同的。

## String、StringBuffer以及StringBuilder的区别

既然在Java中已经存在了String类，那为什么还需要StringBuilder和StringBuffer类呢？那么看下面这段代码：

```java
public class StringDemo {
    public static void main(String[] args) {
        String string = "";
        for(int i=0;i<10000;i++){
            string += "hello";
        }
    }
}
```

这句 string += "hello";的过程相当于将原有的string变量指向的对象内容取出与"hello"作字符串相加操作再存进另一个新的String对象当中，再让string变量指向新生成的对象。如果大家还有疑问可以反编译其字节码文件便清楚了：

```java
public class com.kotlin.StringDemo {
  public com.kotlin.StringDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String
       2: astore_1
       3: iconst_0
       4: istore_2
       5: iload_2
       6: sipush        10000
       9: if_icmpge     38
      12: new           #3                  // class java/lang/StringBuilder
      15: dup
      16: invokespecial #4                  // Method java/lang/StringBuilder."<init>":()V
      19: aload_1
      20: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      23: ldc           #6                  // String hello
      25: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      28: invokevirtual #7                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      31: astore_1
      32: iinc          2, 1
      35: goto          5
      38: return
}
```

从这段反编译出的字节码文件可以很清楚地看出：从第12行开始到第35行是整个循环的执行过程，并且每次循环会new出一个StringBuilder对象，然后进行append操作，最后通过toString方法返回String对象。也就是说这个循环执行完毕new出了10000个对象，试想一下，如果这些对象没有被回收，会造成多大的内存资源浪费。

从上面还可以看出：string+="hello"的操作事实上会自动被JVM优化成：

```java
StringBuilder str = new StringBuilder(string);
str.append("hello");
str.toString();
```

再看下面这段代码

```java
public class StringDemo {
    public static void main(String[] args) {
        StringBuilder stringBuilder = new StringBuilder();
        for(int i=0;i<10000;i++){
            stringBuilder.append("hello");
        }
    }
}
```

反编译字节码文件得到：

```java
public class com.kotlin.StringDemo {
  public com.kotlin.StringDemo();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class java/lang/StringBuilder
       3: dup
       4: invokespecial #3                  // Method java/lang/StringBuilder."<init>":()V
       7: astore_1
       8: iconst_0
       9: istore_2
      10: iload_2
      11: sipush        10000
      14: if_icmpge     30
      17: aload_1
      18: ldc           #4                  // String hello
      20: invokevirtual #5                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      23: pop
      24: iinc          2, 1
      27: goto          10
      30: return
}
```

从这里可以明显看出，这段代码的for循环式从17行开始到27行结束，并且new操作只进行了一次，也就是说只生成了一个对象，append操作是在原有对象的基础上进行的。因此在循环了10000次之后，这段代码所占的资源要比上面小得多。

那么有人会问既然有了StringBuilder类，为什么还需要StringBuffer类？查看源代码便一目了然，事实上，StringBuilder和StringBuffer类拥有的成员属性以及成员方法基本相同，区别是StringBuffer类的成员方法前面多了一个关键字：synchronized，不用多说，这个关键字是在多线程访问时起到安全保护作用的,也就是说StringBuffer是线程安全的。

下面摘了2段代码分别来自StringBuffer和StringBuilder，insert方法的具体实现：

StringBuilder的insert方法

```java
public StringBuilder insert(int index, char[] str, int offset,
                                int len)
    {
        super.insert(index, str, offset, len);
        return this;
    }
```

StringBuffer的insert方法：

```java
@Override
public synchronized StringBuffer insert(int index, char[] str, int offset,
                                          int len)
  {
      toStringCache = null;
      super.insert(index, str, offset, len);
      return this;
  }
```

### 性能测试

```java
public class StringDemo {
    private static int time = 50000;

    public static void main(String[] args) {
        testString();
        testStringBuffer();
        testStringBuilder();
        test1String();
        test2String();
    }

    public static void testString () {
        String s="";
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            s += "java";
        }
        long over = System.currentTimeMillis();
        System.out.println("操作"+s.getClass().getName()+"类型使用的时间为："+(over-begin)+"毫秒");
    }

    public static void testStringBuffer () {
        StringBuffer sb = new StringBuffer();
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            sb.append("java");
        }
        long over = System.currentTimeMillis();
        System.out.println("操作"+sb.getClass().getName()+"类型使用的时间为："+(over-begin)+"毫秒");
    }

    public static void testStringBuilder () {
        StringBuilder sb = new StringBuilder();
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            sb.append("java");
        }
        long over = System.currentTimeMillis();
        System.out.println("操作"+sb.getClass().getName()+"类型使用的时间为："+(over-begin)+"毫秒");
    }

    public static void test1String () {
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            String s = "I"+"love"+"java";
        }
        long over = System.currentTimeMillis();
        System.out.println("字符串直接相加操作："+(over-begin)+"毫秒");
    }

    public static void test2String () {
        String s1 ="I";
        String s2 = "love";
        String s3 = "java";
        long begin = System.currentTimeMillis();
        for(int i=0; i<time; i++){
            String s = s1+s2+s3;
        }
        long over = System.currentTimeMillis();
        System.out.println("字符串间接相加操作："+(over-begin)+"毫秒");
    }

}
```
　
测试结果

> 操作java.lang.String类型使用的时间为：10645毫秒
操作java.lang.StringBuffer类型使用的时间为：4毫秒
操作java.lang.StringBuilder类型使用的时间为：3毫秒
字符串直接相加操作：1毫秒
字符串间接相加操作：9毫秒

由上面的执行结果可以看出：
1. 对于直接相加字符串，效率很高，因为在编译器便确定了它的值，也就是说形如"I"+"love"+"java"; 的字符串相加，在编译期间便被优化成了"Ilovejava"。这个可以用javap -c命令反编译生成的class文件进行验证。
2. 对于间接相加（即包含字符串引用），形如s1+s2+s3; 效率要比直接相加低，因为在编译器不会对引用变量进行优化。
3. String、StringBuilder、StringBuffer三者的执行效率

  StringBuilder > StringBuffer > String

  当然这个是相对的，不一定在所有情况下都是这样。

  比如String str = "hello"+ "world"的效率就比 StringBuilder st  = new StringBuilder().append("hello").append("world")要高。

**总结：**

> * 当字符串相加操作或者改动较少的情况下，建议使用 String str="hello"这种形式；
* 当字符串相加操作较多的情况下，建议使用StringBuilder，如果采用了多线程，则使用StringBuffer。

## 常见的关于String、StringBuffer的面试题

1. 下面这段代码的输出结果是什么？

  String a = "hello2"; 　　String b = "hello" + 2; 　　System.out.println((a == b));

  输出结果为:true。原因很简单，"hello"+2在编译期间就已经被优化成"hello2"，因此在运行期间，变量a和变量b指向的是同一个对象。
2. 下面这段代码的输出结果是什么？

  String a = "hello2"; 　　String b = "hello"; 　　String c = b + 2 　　System.out.println((a == b));

  输出结果为:false。由于有符号引用的存在，所以String c = b + 2;不在在编译期间被优化，不会把b + 2当作字面常量来处理，因此这种方式生成的对象事实上是保存在堆上的。因此a和c指向的不是同一个对象。

3. 下面这段代码的输出结果是什么？

  String a = "hello2";   　 final String b = "hello";       String c = b + 2;       System.out.println((a == c));

  输出结果为：true。对于被final修饰的变量，会在class文件常量池中保存一个副本，也就是说不会通过连接而进行访问，对final变量的访问在编译期间都会直接被替代为真实的值。那么String c = b + 2;在编译期间就会被优化成：String c = "hello" + 2;

4. 下面这段代码输出结果为：

  ```java
  public class StringDemo {
      public static void main(String[] args) {
          String a = "hello2";
          final String b = getHello();
          String c = b + 2;
          System.out.println((a == c));
      }

      public static String getHello() {
          return "hello";
      }
  }
  ```

  输出结果为false。这里面虽然将b用final修饰了，但是由于其赋值是通过方法调用返回的，那么它的值只能在运行期间确定，因此a和c指向的不是同一个对象。

5. 下面这段代码的输出结果是什么？

  ```java
  public class StringDemo {
      public static void main(String[] args) {
          String a = "hello";
          String b =  new String("hello");
          String c =  new String("hello");
          String d = b.intern();

          System.out.println(a==b);
          System.out.println(b==c);
          System.out.println(b==d);
          System.out.println(a==d);
      }
  }
  ```

  输出结果为

  >false
  false
  false
  true  

  这里面涉及到的是String.intern方法的使用。在String类中，intern方法是一个本地方法，intern方法会在运行时常量池中查找是否存在内容相同的字符串，如果存在则返回指向该字符串的引用，如果不存在，则会将该字符串入池，并返回一个指向该字符串的引用。因此，a和d指向的是同一个对象。

6. String str = new String("abc")创建了多少个对象？

  首先必须弄清楚创建对象的含义，创建是什么时候创建的？这段代码在运行期间会创建2个对象么？毫无疑问不可能，用javap -c反编译即可得到JVM执行的字节码内容：

  ```java
    public class com.kotlin.StringDemo {
    public com.kotlin.StringDemo();
      Code:
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return

    public static void main(java.lang.String[]);
      Code:
         0: new           #2                  // class java/lang/String
         3: dup
         4: ldc           #3                  // String abc
         6: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
         9: astore_1
        10: return
    }
  ```
  很显然，new只调用了一次，也就是说只创建了一个对象。

  而这道题目让人混淆的地方就是这里，这段代码在运行期间确实只创建了一个对象，即在堆上创建了"abc"对象。而为什么大家都在说是2个对象呢，这里面要澄清一个概念  该段代码执行过程和类的加载过程是有区别的。在类加载的过程中，确实在运行时常量池中创建了一个"abc"对象，而在代码执行过程中确实只创建了一个String对象。

  因此，这个问题如果换成 String str = new String("abc")涉及到几个String对象？合理的解释是2个。

  个人觉得在面试的时候如果遇到这个问题，可以向面试官询问清楚”是这段代码执行过程中创建了多少个对象还是涉及到多少个对象“再根据具体的来进行回答。
