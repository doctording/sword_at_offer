---
title: "《Java编程思想》第10章：内部类"
layout: page
date: 2019-02-13 00:00
---

[TOC]

# 概述

可以将一个类的定义放到另一类的定义的内部，这就是`内部类`

## 为什么需要内部类

一般来说，内部类继承自某个类或实现某个接口，内部类的代码操作创建它的外围类的对象。所以可以认为内部类提供了某种进入其外围类的窗口。

内部类必须要回答的一个问题是： 如果知识需要一个对接口的引用，为什么部通过外围类实现那个接口呢？答案是：“如果这能满足需求，那么句应该这样做。” 那么内部类实现一个接口和外围类实现这个接口有什么区别吗？答案是：后者不是总能享用到接口带来的方便，有时需要用到接口的实现。所有，使用内部类最吸引人的原因是：

```bash
每个内部类都能独立的继承自一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对内部类都没有影响
```

### 一种隐藏和组织代码的模式

```java
interface Contents {
    int value();
}

interface Destination {
    String readLabel();
}
class Goods {
    private class Content implements Contents {
        private int i = 11;
        @Override
        public int value() {
            return i;
        }
    }
    protected class GDestination implements Destination {
        private String label;
        private GDestination(String whereTo) {
            label = whereTo;
        }
        @Override
        public String readLabel() {
            return label;
        }
    }
    public Destination dest(String s) {
        return new GDestination(s);
    }
    public Contents cont() {
        return new Content();
    }
}

public class MainTest {

    public static void main(String[] args) {
        Goods p = new Goods();
        Contents c = p.cont();
        System.out.println(c.value());
        Destination d = p.dest("Beijing");
        System.out.println(d.readLabel());
    }
}
/* Output:
11
Beijing
 *///:~
```

### 匿名内部类

匿名内部类没有构造器的行为（因为没有类名），但能够通过实例初始化，达到构造行为

```java
interface Contents {
    int value();
}

interface Destination {
    String readLabel();
}

abstract class Destination2 {
    Destination2(String s){
        System.out.println("Constructor s=" + s);
    }
    abstract String readLabel();
}

class Goods3 {
    public Contents cont() {
        // 匿名内部类
        return new Contents() {
            private int i = 11;
            @Override
            public int value() {
                return i;
            }
        };
    }

    public Destination dest(String s) {
        // 匿名内部类使用参数，外围必须是final的
        // String dest 不是final 也ok(Java8) ?
        return new Destination(){
            private String label = s;
            @Override
            public String readLabel(){
                return label;
            }
        };
    }

    public static Destination2 dest2(String dest) {
        return new Destination2(dest){
            @Override
            public String readLabel(){
                return dest;
            }
        };
    }
}

public class MainTest {

    public static void main(String[] args) {
        Goods3 p = new Goods3();
        Contents c = p.cont();
        System.out.println(c.value());
        Destination d = p.dest("shanghai");
        System.out.println(d.readLabel());
        Destination2 d2 = Goods3.dest2("beijing");
        System.out.println(d2.readLabel());
    }
}
/* Output:
11
shanghai
Constructor s=beijing
beijing
 *///:~
```

### 闭包和回调

`闭包`(closure)是一个可调用的对象，它记录了一些信息，这些信息用来创建它的作用域。通过这个定义，可以看出内部类是面向对象的闭包，因为它不仅包含外围类对象（创建内部类的作用域）的信息，还自动拥有一个指向此外围类对象的引用，在此作用域内，内部类有权操作所有的成员，包括private成员。

`回调`(callback), 通过回调，对象能够携带一些信息，这些信息允许它在稍后的某个时刻调用初始的对象。

简单来说，就是我调用你的函数，你调用我的函数。正规一点的说法就是类A的a()函数调用类B的b()函数，当类B的b()函数的执行时又去调用类A里的函数。是一种双向的调用方式。一般情况下，回调分两种，分别是同步回调和异步回调。

#### 同步回调

一种双向调用模式，被调用方在函数被调用时也会调用对方的函数。

参考：
作者：Bro__超
地址：https://www.cnblogs.com/heshuchao/p/5376298.html

```java
class Calculator
{
    public int add(int a, int b)
    {
        return a + b;
    }
}

class SuperCalculator
{
    public void add(int a, int b, Student xiaoming)
    {
        int result = a + b;
        xiaoming.fillBlank(a, b, result);
    }
}

class Student
{
    private String name = null;

    public Student(String name)
    {
        // TODO Auto-generated constructor stub
        this.name = name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    @SuppressWarnings("unused")
    private int calcADD(int a, int b)
    {
        return a + b;
    }

//    public void fillBlank(int a, int b)
//    {
//        int result = calcADD(a, b);
//        System.out.println(name + "心算:" + a + " + " + b + " = " + result);
//    }

//    private int useCalculator(int a, int b)
//    {
//        return new Calculator().add(a, b);
//    }
//
//    public void fillBlank(int a, int b)
//    {
//        int result = useCalculator(a, b);
//        System.out.println(name + "使用计算器:" + a + " + " + b + " = " + result);
//    }

    public void callHelp(int a, int b)
    {
        new SuperCalculator().add(a, b, this);
    }

    // 类似回调函数
    public void fillBlank(int a, int b, int result)
    {
        System.out.println(name + "求助小红计算:" + a + " + " + b + " = " + result);
    }
}

public class MainTest {

    public static void main(String[] args) {
        int a = 1;
        int b = 1;
        Student s = new Student("小明");
        s.callHelp(a, b);
    }
}
```

`Student`调用了`SuperCalculator`的`add`方法, `add`方法在执行的时候，反过来调用了Studdent的`fillBlank`方法

#### 异步回调

一种类似消息或事件的机制，被调用方在函数在收到某种讯息或发生某种事件时，才去调用对方的函数,即通过异步消息进行通知。简单来说，类A的a()函数调用类B的b()函数，但是b()函数很耗时，不确定什么时候执行完毕，如果是同步调用的话会等b()执行完成后才往下执行回调类A中的函数，如果是异步回调的话调用了b()函数，虽然b()函数没有执行完,但仍然继续往下执行，为了完成这点，就需要另开一个线程了。

参考：
作者：O水冰O
来源：CSDN
原文：https://blog.csdn.net/o15277012330o/article/details/79271385

```java
interface doJob {
    void fillBlank(int a, int b, int result);
}

class SuperCalculator {
    public void add(int a, int b, doJob  customer) {
        int result = a + b;

        //让线程等待3秒
        try {
            Thread.sleep(3 * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        customer.fillBlank(a, b, result);
    }
}

class Student {
    private String name = null;

    public Student(String name) {
        this.name = name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public class DoHomeWork implements doJob {

        @Override
        public void fillBlank(int a, int b, int result) {
            System.out.println(name + "求助小红计算:" + a + " + " + b + " = " + result);
        }

    }

    public void callHelp (int a, int b) {
        //开启另一个子线程
        new Thread(new Runnable() {
            @Override
            public void run(){
                new SuperCalculator().add(a, b, new DoHomeWork());
            }
        }).start();

    }
}

class Seller {
    private String name = null;

    public Seller(String name) {
        this.name = name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public class DoHomeWork implements doJob {

        @Override
        public void fillBlank(int a, int b, int result)
        {
            System.out.println(name + "求助小红算账:" + a + " + " + b + " = " + result + "元");
        }

    }

    public void callHelp (int a, int b) {
        new SuperCalculator().add(a, b, new DoHomeWork());
    }
}


public class MainTest {

    public static void main(String[] args) {
        int a = 56;
        int b = 31;
        int c = 26497;
        int d = 11256;
        Student s1 = new Student("小明");
        Seller s2 = new Seller("老婆婆");

        s1.callHelp(a, b);
        System.out.println("/========================1/");
        s2.callHelp(c, d);
        System.out.println("/========================2/");

    }
}
/* output, 其中一次的输出如下， 小明的执行耗时，导致回调是迟输出的
/========================1/
老婆婆求助小红算账:26497 + 11256 = 37753元
小明求助小红计算:56 + 31 = 87
/========================2/
 */

/* output, 其中一次的输出如下， 小明的执行耗时，导致回调是迟输出的
/========================1/
小明求助小红计算:56 + 31 = 87
老婆婆求助小红算账:26497 + 11256 = 37753元
/========================2/
 */
```

### 事件驱动系统

### 内部类标识符

每个类都会产生`.class`文件

内部类： 外围类名字 + $ + 内部类但名字

```java
Contents.class
Destination.class
Destination2.class
Goods3$1.class
Goods3$2.class
Goods3$3.class
Goods3.class
```