---
title: "Java变量类型"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# Java变量类型

static
* 变量属于类，类加载会初始化
* 位于方法区，为所有对象共享，共享一份内存

非static
* 属于某个对象实例，也叫实例变量
* 在虚拟机的堆中分配，非单例模式模式下，是自己内部的变量
* 单例模式因为只有一个对象实例singleton存在，多线程同时操作时是不安全的；而非单例模式下多线程（一个实例一个线程）操作是安全的。

## 静态代码块、构造代码块、构造方法的执行顺序是什么

* 静态代码块：用staitc声明，jvm加载类时执行，仅执行一次
* 构造代码块（非静态代码块）：类中直接用{}定义；每次调用构造函数，均会先调用构造代码块，多个构造函数重复代码可提取到构造代码块。

执行顺序优先级：静态块,构造块（非静态代码块）,构造方法

附：类加载的过程

```bash
1.加载（由类加载器负责根据一个类的全限定名来读取此类的二进制字节流到JVM内部，并存储在运行时内存区的方法区，然后将其转换为一个与目标类型对应的java.lang.Class对象实例）

2.验证（class格式验证，语义验证，操作验证）
    准备（为类中的所有静态变量分配内存空间，并为其设置一个初始值；被final修饰的static变量（常量），则会直接赋值）
        解析（将常量池中的符号引用转为直接引用；解析需要静态绑定的内容）

3.初始化（先父后子）（为静态变量赋值；执行static代码块）
```

## 为什么静态方法无法调用非静态成员（方法和变量）?

类加载，将类的静态方法会加载到方法区，静态方法数据类，而非具体对象

new 对象，对象在堆只能够创建，this关键字指向该对象

<font color='red'>在一个类的静态成员中去访问非静态成员会出错：显然因为在类的非静态成员不存在的时候静态成员就已经存在了，访问一个内存中不存在的东西当然会出错。</font>

## 静态代码和静态变量的执行顺序

（静态）变量和（静态）代码块的也是有执行顺序的，与代码书写的顺序一致。在（静态）代码块中可以使用（静态）变量，但是被使用的（静态）变量必须在（静态）代码块前面声明。

如下编译错误

```java
class Test{
  
    static{
       j= 2; // 这里会提示无法从静态上下文中引用非静态 变量 j
    }

    public int j = 1;
    
    public Test() {
    }

}
```

如下输出：

* 有父子类，先执行父类的静态块；先父类的构造方法
* 静态块只加载一次

```java

class B {

    public B(){
        System.out.println("父类B的构造函数");
    }

    static {
        System.out.println("父类B的中的静态代码块");
    }

    {
        System.out.println("父类B的中的非静态代码块 sya()");
    }
}

class A extends B {
    public A() {
        System.out.println("子类A的构造函数");
    }

    static {
        System.out.println("子类A的中的静态代码块");
    }

    {
        System.out.println("子类A的中的非静态代码块 sya()1");
    }
}

public class MainTest {

    public static void main(String[] args) {
        A a = new A();
        System.out.println("A!");
        A a2 = new A();
        System.out.println("启动完成");
    }
}
/**
父类B的中的静态代码块
子类A的中的静态代码块
父类B的中的非静态代码块 sya()
父类B的构造函数
子类A的中的非静态代码块 sya()1
子类A的构造函数
A!
父类B的中的非静态代码块 sya()
父类B的构造函数
子类A的中的非静态代码块 sya()1
子类A的构造函数
启动完成
 */
```
