---
title: "《Java编程思想》第5章：初始化和清理(重点)"
layout: page
date: 2018-12-12 00:00
---

[TOC]

# 初始化和清理

## 构造函数

调用构造函数是编译器的责任，必须要让编译器知道调用的是哪个方法；分配内存空间后，就会调用构造函数，确保使用对象前，对象已经被初始化了。

* 函数名是类名
* 无参数的构造函数称为默认构造器
* 没有返回类型(不是返回类型为void，而是没有返回类型）（new返回了新建对象的引用）
* this(所操作对象的引用) 能在类的内部方法使用（static不能使用），表示"调用方法的那个对象"的引用，编译器默认隐式的将this作为普通方法的第一个参数
* 尽管可以用this调用一个构造器，但是不能调用两个且必须置于起始的语句，否则会报错,类似`Error:(25, 13) java: 对this的调用必须是构造器中的第一个语句`

## 方法重载

名字相同，有不同的参数类型列表

* 不以返回值作区分
* 基本类型作为参数，编译器会选择部分类型提升和窄化

## static 与 this

* static方法是没有this的方法
* static方法不能调用普通的方法，但普通的方法能调用static方法

## 清理： 终结处理和垃圾回收

1. 对象可能不被垃圾回收
2. 垃圾回收并不等于`析构`
3. 垃圾回收只与内存有关：垃圾回收只对程序不再使用对内存进行回收

JVM未面临内存耗尽对情形，它是不会浪费时间去执行垃圾回收以恢复内存的

## finalize()

## 成员初始化

Java尽力保证： 所有变量在使用前都能得到恰当的初始化。对于方法的局部变量，Java以编译时错误的形式来贯彻这种保证

```java
void f(){
    int i;
    i++; // Error, i not initiallized
}
```

对于类数据成员(即字段)是基本类型，即便没有初始化，会被默认为`0`值, 如果定义一个对象引用，却没有赋予初始值，则会出现`null`的情况

### 静态数据与初始化

无论有多少个实例对象，静态数据都只占用一份存储，static关键字不能应用于局部变量，只能作用于域，其数据没有被初始化则会称为`0`或者`null`

### 初始化顺序（包含构造函数）

单个类的初始化顺序： `静态数据成员 -> 静态代码块 -> 非静态数据成员 -> 非静态代码块 -> 构造方法`

代码执行从上到下（当然，不同类型的成员不受顺序影响，需要遵循初始化顺序），不合理的顺序会导致某些错误

### 例子代码理解

```java
class Bowl{
    Bowl(int marker) {
        System.out.println(String.format("Bowl(marker:%d)", marker));
    }
    void f1(int marker) {
        System.out.println(String.format("f1(marker:%d)", marker));
    }
}

class Table{
    static Bowl bowl1 = new Bowl(1);

    public Table() {
        System.out.print("Table()");
        bowl2.f1(1);
    }
    void f2(int marker){
        System.out.println(String.format("f2(marker:%d)", marker));
    }

    static Bowl bowl2 = new Bowl(2);
}

class Cupboard{
    Bowl boel3 = new Bowl(3);
    static Bowl bowl4 = new Bowl(4);

    public Cupboard() {
        System.out.println("Cupboard()");
        bowl4.f1(2);
    }
    void f3(int marker){
        System.out.println(String.format("f3(marker:%d)", marker));
    }
    static Bowl bowl5 = new Bowl(5);
}

class Main {
    static Table table = new Table();
    static Cupboard cupboard = new Cupboard();

    public static void main(String[] args) throws Exception {
        System.out.println("Creating new Cupboard() in main()");
        new Cupboard();
        System.out.println("Creating new Cupboard() in main()");
        new Cupboard();
        table.f2(1);
        cupboard.f3(1);
    }

}
```

```java
// output
Bowl(marker:1) // Main的静态成员Table对象的静态成员`Bowl bowl1`
Bowl(marker:2) // 同上，与代码位置无关的able对象的静态成员`Bowl bowl2`
Table()f1(marker:1) // Table的构造函数

Bowl(marker:4) // Cupboard对象的静态成员`Bowl bowl4`
Bowl(marker:5) // Cupboard对象的静态成员`Bowl bowl4`
Bowl(marker:3) // Cupboard对象的普通成员`Bowl bowl3`
Cupboard() // Cupboard构造函数
f1(marker:2)

Creating new Cupboard() in main() // main方法
Bowl(marker:3)
Cupboard()
f1(marker:2)

Creating new Cupboard() in main()
Bowl(marker:3)
Cupboard()
f1(marker:2)

f2(marker:1)
f3(marker:1)
```

### 总结一个对象的创建过程(class & class实例 => 对象)（实例化的顺序问题？）

假设有一个名为Dog的类

1. 即使没有显示地使用static关键字，构造器其实也可以看成静态方法。首次创建Dog类的对象时，或者Dog类的静态方法/静态域首次被访问时，Java解释器必须查找类路径，以定位`Dog.class`文件
2. 然后载入`Dog.class`(这将创建一个Dog对象)，有关静态初始化的所有动作都会执行。因此，**静态初始化只在Class对象首次加载的时候执行一次**
3. 当用new Dog()创建对象的时候，首先将在`堆`上为Dog对象分配足够的存储空间
4. 这块存储空间会被清零，这就自动地将Dog对象中的所有基本类型都设置成了默认值（`0`，`flase`，`null`等默认值）
5. 执行所有出现于字段定义外的初始化动作
6. 执行构造器

初始化顺序如下：

`静态数据成员 -> 静态代码块 -> 非静态数据成员 -> 非静态代码块 -> 构造方法`

按照代码中的顺序，所以也可能是：

`静态代码块 -> 静态数据成员 -> 非静态代码块  -> 非静态数据成员 -> 构造方法`

<font color='red'>总是先`静态`，再`非静态`，最后`构造方法`</font>

### 数组的初始化

```java
int[] a1; // int a1[];
```

编译器不允许制定数组的大小，只是个引用，并为该引用分配了足够的存储空间；为了给数组创建相应的存储空间，必须写初始化表达式。

数组的初始化动作可以出现在代码的任何地方，但也可以使用特殊的初始化表达式，但必须是在创建数组的地方出现，这种特殊的表达式是用`{}`扩起来的值组成的。

```java
// 正确写法
int[] a1 = {1, 2, 3, 4, 5}
// 错误写法
int[] a1;
a1 = {1, 2, 3, 4, 5}
```

如果忘记了创建对象，并且试图使用数组中的空引用，就会出现runtime exception

由于所有的类都继承自`Object`类，所以也可以用Object数组

## 抽象类初始化问题

可以初始化,但是不能直接初始化(即new，即不能实例化),要么通过多态,要么通过静态的内存来指向一块区域让你去调用抽象类的某个方法(即局部初始化)

```java
abstract class Abs_A{
    int a;
    //这是一个抽象方法，
    public abstract void run();
    Abs_A(){
        System.out.println("Abs_A()");
    }
    void test(){
        System.out.println("test");
    }
}

class A extends Abs_A{

    @Override
    public void run() {
        System.out.println("run");
    }
}

A a = new A();
a.test();
/*
Abs_A()
test
*/
```
