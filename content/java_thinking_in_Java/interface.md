---
title: "《Java编程思想》第9章：接口 补充：抽象类"
layout: page
date: 2019-02-12 00:00
---

[TOC]

# 概述

接口和内部类为我们提供了一种将接口与实现分离的更加结构化的方法

## 抽象方法和抽象类

包含`抽象方法`的类叫做`抽象类`。如果一个类包含一个或者多个抽象方法，该类必须被限定为`抽象的`

抽象方法：只有声明，没有方法体, eg:

```java
abstract void f();
```

抽象类和普通类的主要有三点区别：

1. **抽象方法必须为public或者protected**（因为如果为private，则不能被子类继承，子类便无法实现该方法），缺省情况下默认为public。
2. 抽象类不能用来创建对像，即**无法实例化**
3. 如果一个类继承于一个抽象类，则**子类必须实现父类的抽象方法**。如果子类没有实现父类的抽象方法，则必须将子类也定义为abstract类

## 接口（interface）

接口，是对行为的抽象，一般为实现xx接口; Java继承是单继承, 而实现接口是可以实现多个接口的

```java
class ClassName implements Interface1,Interface2,[....]{
}
```

* 与`抽象类`语法层面上的区别

1. 抽象类可以提供成员方法的实现细节，而接口中只能存在`public abstract`方法(默认就是`public abstract`，前面加上修饰词会提示多余)

2. 抽象类中的成员变量可以是各种类型的，而**接口中的成员变量**只能是`public static final`类型的(默认就是`public static final`，前面加上修饰词会提示多余)

3. 接口中不能含有静态代码块以及静态方法，而抽象类可以有静态代码块和静态方法；

4. 一个类只能继承一个抽象类，而一个类却可以实现多个接口。

* 与`抽象类`设计层面上的区别

1. 抽象类是对一种事物的抽象，即对类抽象，而接口是对行为的抽象。抽象类是对整个类整体进行抽象，包括属性、行为，但是接口却是对类局部（行为）进行抽象

2. 设计层面不同，抽象类作为很多子类的父类，它是一种模板式设计。而接口是一种行为规范，它是一种辐射式设计

参考1 ：
作者：海子
出处：http://www.cnblogs.com/dolphin0520/

## 接口与工厂

接口是实现多重继承的途径，而生成遵循某个接口的对象的典型方式就是`工厂方法`设计模式。这与直接调用构造器不同，我们在工厂对象上调用的是创建方法，而该工厂对象将生成接口的某个实现的对象。理论上，通过这种方式，我们的代码将完全与接口的实现分离

```java
interface Cycle {
    int wheels();
}
interface CycleFactory {
    Cycle getCycle();
}
class Unicycle implements Cycle {
    @Override
    public int wheels() { return 1; }
}
class UnicycleFactory implements CycleFactory {
    @Override
    public Unicycle getCycle() { return new Unicycle(); }
}
class Bicycle implements Cycle {
    @Override
    public int wheels() { return 2; }
}
class BicycleFactory implements CycleFactory {
    @Override
    public Bicycle getCycle() { return new Bicycle(); }
}
class Tricycle implements Cycle {
    @Override
    public int wheels() { return 3; }
}
class TricycleFactory implements CycleFactory {
    @Override
    public Tricycle getCycle() {
        return new Tricycle();
    }
}

public class MainTest {

    public static void ride(CycleFactory fact) {
        Cycle c = fact.getCycle();
        System.out.println("Num. of wheels: " + c.wheels());
    }
    public static void main(String[] args) {
        ride(new UnicycleFactory());
        ride(new BicycleFactory ());
        ride(new TricycleFactory ());
    }
}
/* Output:
 Num. of wheels: 1
 Num. of wheels: 2
 Num. of wheels: 3
 *///:~
```

如果不用工厂方法，就必须在某处指定将要创建的Cycle的确切类型，以便调用合适的构造器

参考：
http://www.runoob.com/design-pattern/factory-pattern.html

1. 您需要一辆汽车，可以直接从工厂里面提货，而不用去管这辆汽车是怎么做出来的，以及这个汽车里面的具体实现

2. Hibernate 换数据库只需换方言和驱动就可以

## 抽象类`null`的问题

```java
abstract class A{
    protected String description;

    public abstract void setDescription();

    public void show(){
        System.out.println("show:" + description);
    }
}

class B extends A{

    @Override
    public void setDescription() {
        this.description = "B";
    }
}

public class Main {

    public static void main(String[] args) throws Exception {
        B b = new B();
        System.out.println(b.description);
        b.show();
    }
}
/* output

null
show:null

 */
```

只需要改成

```java
B b = new B();
b.setDescription();
System.out.println(b.description);
b.show();
```

## 抽象类有构造函数，子类也必须有一个默认构造函数

```java
abstract class A{
    private String name = "A";
    protected String description;

    public A(){}

    public A(String description){
        this.description = description;
    }

    abstract void setDescription();

    public void show(){
        System.out.println("show:" + name + " " + description);
    }
}

class B extends A{

    public B(){
    }

    public B(String description){
        super(description);
        this.description = description;
    }

    @Override
    public void setDescription() {
        description = "B";
    }

}
```

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_thinking_in_Java/imgs/abstract_class.png)
