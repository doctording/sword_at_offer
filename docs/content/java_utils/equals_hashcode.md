---
title: "== & equals方法 & hashcode方法"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# 操作符`==`

## 作用于基本数据类型的变量，则直接比较其存储的`值`是否相等

基本数据类型 | 包装类
-|-
byte   | Byte
short  | Short
int | Integer
long  |  Long
char   |     Char
float   |     Float
double  |   Double
boolean  | Boolean

## 作用于引用类型的变量，则比较的是所指向的对象的地址

Java8程序验证

```java
int a = 1;
int b = 1;
Integer ia = new Integer(a);
Integer ib = new Integer(b);
//  true
System.out.println(a == b);
// false, idea编译器插件提示包装类使用equals比较
// Integer类的equals方法重写就是比较的 int value
System.out.println(ia == ib);
```

```java
String s0 = "helloworld";
String s1 = "helloworld";
String s2 = "hello" + "world";
// true s0跟s1是指向同一个对象,字符串常量
System.out.println(s0 == s1);
// true s2也指向同一字符串常量
System.out.println(s0 == s2);
String s3 = new String(s0);
String s4 = new String(s0);
// false s3, s4是不同的对象，地址不同, 只是内容相同
System.out.println(s3 == s4);
// true，用String的`equals`方法是判断的字符串内容是否相等
System.out.println(s3.equals(s4));
```

# `equals`方法

equals() 的作用是 用来判断两个对象是否相等。

equals() 定义在JDK的Object.java中。通过判断两个对象的地址是否相等(即，是否是同一个对象)来区分它们是否相等。源码如下：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

## `equals`方法是基类`Object`中的方法，因此对于所有的继承于`Object`的类都会有该方法

* equals方法不能作用于基本数据类型的变量

* 如果没有对`equals`方法进行重写，则比较的是`引用类型的变量所指向的对象的地址`

* 诸如`String`、`Date`等类对`equals`方法进行了`override`的话，比较的是所指向的对象的内容

* equals默认实现是`==`, 不过大部分是要重写的类的equals方法的

## 对象的hashcode方法

`hashCode()`在散列表中才有用，在其它情况下没用。在散列表中`hashCode()`的作用是获取对象的散列码，进而确定该对象在散列表中的位置(散列表存储的是键值对(key-value)，它的特点是：能根据“键”快速的检索出对应的“值”。这其中就利用到了散列码！)

```java
 /**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@code equals(Object)}
     *     method, then calling the {@code hashCode} method on each of
     *     the two objects must produce the same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link java.lang.Object#equals(java.lang.Object)}
     *     method, then calling the {@code hashCode} method on each of the
     *     two objects must produce distinct integer results.  However, the
     *     programmer should be aware that producing distinct integer results
     *     for unequal objects may improve the performance of hash tables.
     * </ul>
     * <p>
     * As much as is reasonably practical, the hashCode method defined by
     * class {@code Object} does return distinct integers for distinct
     * objects. (This is typically implemented by converting the internal
     * address of the object into an integer, but this implementation
     * technique is not required by the
     * Java&trade; programming language.)
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object#equals(java.lang.Object)
     * @see     java.lang.System#identityHashCode
     */
    public native int hashCode();
```

说对于两个对象，如果调用equals方法得到的结果为true，则两个对象的hashcode值必定相等；

* 如果equals方法得到的结果为false，则两个对象的hashcode值**不一定不同**

* 如果两个对象的hashcode值不等，则equals方法得到的结果必定为false；

* 如果两个对象的hashcode值相等，则equals方法得到的**结果未知**。

设计`hashCode()`时最重要的因素就是：无论何时，对同一个对象调用`hashCode()`都应该产生同样的值。如果在将一个对象用put()添加进HashMap时产生一个hashCdoe值，而用get()取出时却产生了另一个hashCode值，那么就无法获取该对象了。所以如果你的hashCode方法依赖于对象中易变的数据，用户就要当心了，因为此数据发生变化时，hashCode()方法就会生成一个不同的散列码

### 例子，需要重写 hashcode 方法

```java
class People{
    private String name;
    private int age;

    public People(String name,int age) {
        this.name = name;
        this.age = age;
    }

    public void setAge(int age){
        this.age = age;
    }

    @Override
    public boolean equals(Object obj) {
        return this.name.equals(((People)obj).name) && this.age == ((People)obj).age;
    }

    @Override
    public int hashCode() {
        return name.hashCode()*101+age;
    }

}

public static void main(String[] args) throws Exception {
    People p1 = new People("Jack", 12);
    System.out.println(p1.hashCode());

    HashMap<People, Integer> hashMap = new HashMap<People, Integer>();
    hashMap.put(p1, 1);
    System.out.println(hashMap.get(p1)); // 1
    People p2 = new People("Jack", 12);
    System.out.println(p2.hashCode());
    System.out.println(hashMap.get(p2)); // 1

    p2.setAge(99);
    System.out.println(hashMap.get(p2)); // null
}
```

### hash函数

特点

1. 单向函数，反向运算无法完成
2. 任务一长度输入，得到 固定长度输出
3. 输入不变，输出就不会变（Whenever it is invoked on the same object more than once during an execution of a Java application, the {@code hashCode} method must consistently return the same integer）

常见的有数百种hash函数，eg: MD5加密 + salt

### Java 对象 hashcode的作用

1. 记录在markword中
2. 与hash table密切相关
