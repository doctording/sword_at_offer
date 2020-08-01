---
title: "Java8 interface defalt static"
layout: page
date: 2019-03-21 13:00
---

[TOC]

# Java8 interface default & static

Java8中为接口新增了一项功能：定义一个或者更多个静态方法。用法和普通的static方法一样。

## 实例1

* interface Animal

```java
package com.mb;

/**
 * @Author mubi
 * @Date 2019/3/21 12:41 PM
 */
public interface Animal {

    void play();

    /**
     * 静态方法
     */
    static void sleep() {
        System.out.println("static sleep");
    }

    /**
     * 默认方法
     */
    default void eat() {
        System.out.println("default eat");
    }

}
```

* Cat

```java
package com.mb;

/**
 * @Author mubi
 * @Date 2019/3/21 12:45 PM
 */
public class Cat implements Animal{
    @Override
    public void play() {
        System.out.println("cat play");
    }
}
```

* Dog

```java
package com.mb;

/**
 * @Author mubi
 * @Date 2019/3/21 12:45 PM
 */
public class Dog implements Animal{
    @Override
    public void play() {
        System.out.println("dog play");
    }

    @Override
    public void eat() {
        System.out.println("dog eat");
    }
}
```

* main

```java
package com.mb;

/**
 * @Author mubi
 * @Date 2019/3/17 7:56 PM
 */
public class Main {

    public static void main(String[] args) throws Exception {
        Dog dog = new Dog();
        Cat cat = new Cat();

        dog.eat();
        dog.play();

        cat.play();
        cat.eat();

        Animal.sleep();
    }

}
/*output

dog eat
dog play
cat play
default eat
static sleep

 */
```

## 实例2

```java
package com.mb;

interface A {

    String what();

    default void say() {
        System.out.println("before:" + System.currentTimeMillis());
        System.out.println(what());
        System.out.println("after:" + System.currentTimeMillis());
    }
}

class C implements A{

    @Override
    public String what() {
        return "I'm C";
    }
}

class D implements A{

    @Override
    public String what() {
        return "I'm D";
    }

    @Override
    public void say() {
        System.out.println("D:" + what());
    }

}

public class Main {

    public static void main(String[] args) throws Exception {
       new C().say();
       new D().say();
    }

}
/*output

before:1553145296092
I'm C
after:1553145296093
D:I'm D

 */
```

## `interface`的`default`方法 与 继承

```java
package com.mb;

interface A {
    default void say() {
        System.out.println("A");
    }
}

class B {
    public void say() {
        System.out.println("B");
    }
}


class C implements A{

}
class D extends B{

}
class E extends B implements A{

}


public class Main {

    public static void main(String[] args) throws Exception {
       new C().say(); // A
       new D().say(); // B
       new E().say(); // B
    }

}
```

子类**优先继承父类的方法**，如果父类没有相同签名的方法，才继承接口的`default`方法。

## `interface`继承`interface`

```java
package com.mb;

interface A {

    String what();

    default void say() {
        System.out.println("A:" + what());
    }
}

/**
 * interface 的继承 可以重写 default 方法
 */
interface B extends A{

    String how();

    @Override
    default void say() {
        System.out.println("B:" + what());
    }

}

class C implements A{

    @Override
    public String what() {
        return "I'm C";
    }

    @Override
    public void say() {
        System.out.println("C:" + what());
    }
}

class D implements B{

    @Override
    public String what() {
        return "I'm D";
    }

    @Override
    public String how() {
        return "how I' m D";
    }

}
class E implements A{

    @Override
    public String what() {
        return "I'm E";
    }
}

public class Main {

    public static void main(String[] args) throws Exception {
       new C().say();
       new D().say();
       new E().say();
    }

}
/*output

A:I'm C
B:I'm D
A:I'm E
 */
```
