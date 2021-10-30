---
title: "《Java编程思想》第8章：多态"
layout: page
date: 2019-02-12 00:00
---

[TOC]

# 面向对象的3个基本特征

多态，封装(抽象)，继承

## 向上转型

```java
class Cycle {
    public void play(){
        Sxystem.out.println("Cycle");
    }
}

class Unicycle extends Cycle {
    @Override
    public void play() {
        System.out.println("Unicycle");
    }
}

class Bicycle extends Cycle {
    @Override
    public void play() {
        System.out.println("Bicycle");
    }
}

public class MainTest {
    public static void ride(Cycle c) {
        System.out.println(c.getClass());
        c.play();
    }

    public static void main(String args[]) {
        ride(new Cycle());    // No upcasting
        ride(new Unicycle()); // Upcast
        ride(new Bicycle());  // Upcast
    }
}
/*output

class com.test.Cycle
Cycle
class com.test.Unicycle
Unicycle
class com.test.Bicycle
Bicycle
 */
```

## 方法调用绑定

将一个方法调用同一个方法主体关联起来被称作`绑定`，当绑定发生在程序运行之前时（如果有的话，由编译器和连接器负责，`C`就是一种前绑定的语言）称为`前期绑定`（early binding ）。在运行时，根据对象的类型来决定运行哪个方法称为`后期绑定`（late binding)，后期绑定也被称为`动态绑定`（dynamic binding ）或运行时绑定(run-time binding).

Java中除了static方法和final方法（`private`方法也属于`final`方法）之外，其它所有方法都是`后期绑定`。当方法声明为final类型时，因为方法不会被继承或改变，也就无谓多态啦，这时就是使用的`前期绑定`。

## 缺陷: 域与静态方法

只有普通的方法调用可以是多态的；如果直接访问某个域，这个访问将在编译期进行解析，不是多态

```java
class Super {
    public int field = 0;

    public int getField() {
        return field;
    }
}

class Sub extends Super {
    public int field = 1;

    @Override
    public int getField() {
        return field;
    }

    public int getSuperField() {
        // 显示的说明为：super.field
        return super.field;
    }
}

public class MainTest {

    public static void main(String[] args) {
        Super sup = new Sub(); // Upcast
        System.out.println("sup.field:" + sup.field + " sup.getField():" + sup.getField());
        Sub sub = new Sub();
        System.out.println("sub.field:" + sub.field + " sub.getField():" + sub.getField()
            + " sub.getSuperField():" + sub.getSuperField());
    }
}
/*output

sup.field:0 sup.getField():1
sub.field:1 sub.getField():1 sub.getSuperField():0
 */
```

```java
class Super {
    public static String staticGet() {
        return "Super static get()";
    }
    public String dynamicGet() {
        return "Super dynamic get()";
    }
}

class Sub extends Super {

    public static String staticGet() {
        return "Sub static get()";
    }

    @Override
    public String dynamicGet() {
        return "Sub dynamic get()";
    }
}

public class MainTest {

    public static void main(String[] args) {
        Super sup = new Sub(); // Upcast
        System.out.println(sup.staticGet()); // 这里会警告：不应该通过类实例访问静态成员
        System.out.println(sup.dynamicGet());
    }
}
/*output

Super static get()
Sub dynamic get()
 */
```

## 向下转型

向上转型可能会丢失信息，但向上转型是安全的，因为基类不会出现大于导出类的接口。

向下转型可能是不安全的，java在运行期间会检查转型，必要时返回`ClassCastException`，这种运行期间对类型进行检查的行为称为"运行是类型识别（RTTI）"

```java
class Useful {
    public void f() {
    }
    public void g() {
    }
}

class MoreUseful extends Useful {

    @Override
    public void f() {
        super.f();
    }

    @Override
    public void g() {
        super.g();
    }
    public void u() {
    }
    public void v() {
    }
}

public class MainTest {

    public static void main(String[] args) {
        Useful[] x = {new Useful(), new MoreUseful()};
        x[0].f();
        x[1].g();
        ((MoreUseful)x[1]).u(); // DownCast / RTTI
        ((MoreUseful)x[0]).u(); // java.lang.ClassCastException throw
    }
}
```

## Java多态和例子

Java多态的三个必要条件：`继承`,`重写`,`向上转型`

```java
C
  \
   \
     -> B ----> A
   /
  / 
D
```

eg:

```java

class A {
    public String show(D obj){
       return "A & D";
    }
    public String show(A obj){
        return "A & A";
    }
}

class B extends A {

    public String show(B obj){
        return "B & B";
    }

    @Override
    public String show(A obj){
        return "B & A";
    }
}

class C extends B {

}

class D extends B {

}


public class Solution {

    public static void main(String[] args) throws InterruptedException {
        A a1 = new A();
        A a2 = new B();
        B b = new B();
        C c = new C();
        D d = new D();

        /**
         * 优先级
         *  this.show(O)
         *      super.show(O)
         *          this.show((super)O)
         *              super.show((super)O))
         */

        System.out.println("1---" + a1.show(b)); // A & A
        System.out.println("2---" + a1.show(c)); // A & A
        System.out.println("3---" + a1.show(d)); // A & D

        System.out.println("3---" + a2.show(b)); // B & A
        System.out.println("4---" + a2.show(c)); // B & A
        System.out.println("5---" + a2.show(d)); // A & D

        System.out.println("6---" + b.show(b)); // B & B
        System.out.println("7---" + b.show(c)); // B & B
        System.out.println("8---" + b.show(d)); // A & D
    }

}
```

分析如下代码，按照`this.show(O)`, `super.show(O)`,`this.show((super)O)`,`super.show((super)O))`

```java
A a2 = new B()
a2.show(b) // B & A
```

那么`a2.show(b)`翻译过来是`A.show(B)`，`this`是`A`, 但是A并没有show(B)的方法，且A的父类是`Object`类，所以：

```java
this.show(O), 不满足
super.show(O), 不满足
this.show((super)O), 来到 A.show(A)方法了，但是a2是子类B的实例，而B重写了show(A)方法，所以本质会调用B.show(A)
super.show((super)O))
```

可以看到是：实例化对象类型(`new B()`)决定了方法的调用,而不是引用类型（`A a2`）

----

再分析如下代码，按照`this.show(O)`, `super.show(O)`,`this.show((super)O)`,`super.show((super)O))`

```java
B b = new B()
b.show(c) // B & B
```

那么`b.show(c)`翻译过来是`B.show(C)`，`this`是`B`

```java
this.show(O), 不满足，B没有show(C)方法
super.show(O), 不满足，A没有show(C)方法
this.show((super)O), 来到 B.show(B)方法，有了，则返回B.show(B)了
super.show((super)O))
```
