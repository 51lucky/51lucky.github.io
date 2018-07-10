---
title: hashCode方法和equals方法
abbrlink: 142beca2
date: 2018-07-10 10:56:01
tags:
---
`hashCode`方法和`equals`方法都是`java.lang.Object`类的方法。`equals`方法是判断两个对象是否等价的方法，而`hashCode`方法则是为散列数据结果服务的计算散列值的方法。

<!-- more -->

## equals方法

`equals`方法注重**两个对象在逻辑上是否相等**。重写`equals`方法需要遵循以下规则：
* 自反性: 一个对象与自身相等，即x = x。 对于任何非空对象x，x.equals(x)必定为true。
* 对称性: 对象之间的等价关系是可交换的，即a = b ⇔ b = a。对于任何非空对象x,y。x.equals(y)为true,则y.equals(x)一定也为true。
* 传递性: (a = b)∧( b = c) ⇒ (a = c)。对于任何非空对象x、y、z,若x.equals(y)为true且y.equals(z)为true,则x.equals(z)为true.
* 一致性: 对于任何非null对象x,y，只要所比较的信息未变，则连续调用x.equals(y)总是得到一致的结果。
* 对于任何非空对象x,x.equals(null)必定为false。

### 重写规则

1. 首先判断传入的对象与自身是否为同一对象，如果是的话直接返回`true`。这相当于一种性能优化，尤其是在对象比较操作代价高昂的时候，这种优化非常有效。
2. 判断对象是否为正确的类型。若此方法接受子类，即子类判断等姐的逻辑与父类相同，则可以用`instanceof`操作符；若逻辑不同，即仅接受当前类型，则可以用`getClass`方法获取`Class`对象来判断。注意使用`getClass`方法时，必须保证非空，而用`instanceof`操作符则不用非空验证(null instanceof object 的值为false)。
3. 将类型转换为相应的类型，由于前面已经做过校验，因此这里做类型转换的时候不应当抛出`ClassCastException`异常。
4. 编写相关的判断逻辑。简单的示例如下：

```Java
class Fucker {
    private int id;
    private String name;
    public Fucker(int id, String name) {
        this.id = id;
        this.name = name;
    }
    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        //使用instanceof来判断类型，不需要非空验证
        if (!(o instanceof Fucker))
            return false;

        //使用getClass方法来判断类型，需要做非空验证
        /*
        if (o == null || getClass() != o.getClass())
            return false;
        */
        Fucker fucker = (Fucker) o;
        return id == fucker.id && !(name != null ? !name.equals(fucker.name) : fucker.name != null);
    }
    @Override
    public int hashCode() {
        int result = id;
        result = 31 * result + (name != null ? name.hashCode() : 0);
        return result;
    }
}
```
### 注意事项
1. 我们无法在扩展一个可实例化的类的同时，**即增加新的成员变量**，同时又保留**原先的equals约定**
2. 不要写错equals方法的参数类型，标准的应该是`public boolean equals(Object o)`,若写错就变成重载而不是重写。
3. 如果重写了equals方法，则一定要重写`hashCode`方法。

### equals和==的区别
`equals`方法用来判断**两个对象在逻辑上是否相等**，而`==`用来判断两个引用对象是否指向同一个对象（是否为同一个对象,即两个对象地址是否相同），如果两个对象是基本数据类型（byte,short,char,int,long,float,double,boolean）则比较的是基本类型的字面值。
```Java
String str1 = "Fucking Scala";
String str2 = new String("Fucking Scala");
String str3 = new String("Fucking Scala");
String str4 = "Fucking Scala";
System.out.println(str1 == str2); // false
System.out.println(str2 == str3); // false
System.out.println(str2.equals(str3)); // true
System.out.println(str1 == str4); // true
str4 = "Fuck again!";
String str5 = "Fuck again!";
System.out.println(str1 == str4); // false
System.out.println(str4 == str5); // true
```
## hashCode方法
如果重写了`equals`方法，则一定要重写`hashCode`方法。
重写hashCode方法的原则如下：

1. 在程序执行期间，只要`equals`方法的比较操作用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法必须始终如一地返回同一个整数。
2. 如果两个对象通过`equals`方法比较得到的结果是相等的，那么对这两个对象进行hashCode得到的值应该相同。
3. 两个不同的对象，hashCode的结果可能是相同的，这就是哈希表中的冲突，为了保证哈希表的效率，哈希算法应尽可能的避免冲突。

建议：

* 永远不要让哈希算法返回一个常值，这时哈希表将退化成链表，查找时间复杂度也从***O(1)***退化到***O(N)***。
* 如果参数是`boolean`类型，计算`(f ? 1 : 0)`
* 如果参数是`byte`，`char`，`short`或者`int`类型，计算`(int)f`
* 如果参数是`long`类型，计算`(int)(f ^ (f >>> 32))`
* 如果参数是`float`类型，计算`Float.FloatToIntBits(f)`
* 如果参数是`double`类型，计算`Double.doubleToLongBits(f)`得到long类型的值，再根据公式计算出相应的hash值
* 如果参数是`Object`类型，那么应计算其有用的成员变量的hash值，并按照下面的公式计算最终的hash值
* 如果参数是个数组，那么把数组中的每个值都当作单独的值，分别按照上面的方法单独计算hash值，最后按照下面的公式计算最终的hash值

> 组合公式 result = 31 * result + c

例子: `String`类的`hashCode`方法如下（jdk 1.8）
```Java
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```
自定义类的hashCode
```Java
class Duck {
    private int id;
    private String name;
    private double weight;
    private float height;
    private String note;
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Duck duck = (Duck) o;
        if (id != duck.id) return false;
        if (Double.compare(duck.weight, weight) != 0) return false;
        if (Float.compare(duck.height, height) != 0) return false;
        if (name != null ? !name.equals(duck.name) : duck.name != null) return false;
        return !(note != null ? !note.equals(duck.note) : duck.note != null);
    }
    @Override
    public int hashCode() {
        int result;
        long temp;
        result = id;
        result = 31 * result + (name != null ? name.hashCode() : 0);
        temp = Double.doubleToLongBits(weight);
        result = 31 * result + (int) (temp ^ (temp >>> 32));
        result = 31 * result + (height != +0.0f ? Float.floatToIntBits(height) : 0);
        result = 31 * result + (note != null ? note.hashCode() : 0);
        return result;
    }
}
```
 hashCode在集合类(HashMap，HashSet等）操作中使用，为了提高查询速度。将对象放入集合中，首先判断要放入对象的hashcode值与集合中任意一个元素的hashCode值是否相等，如果不相等直接将该对象放入集合中。如果hashCode值相等，然后再通过equals方法判断要放入对象与该对象是否相等，如果equals判断不相等，直接将该元素放入到集合中，否则不放入。
