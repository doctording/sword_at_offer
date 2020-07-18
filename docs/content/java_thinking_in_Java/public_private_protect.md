---
title: "《Java编程思想》第6章：访问权限控制（重要）"
layout: page
date: 2018-12-12 00:00
---

[TOC]

# 访问权限控制

## 访问权限修饰词

Java访问控制符的含义和使用情况(不注明权限，默认是default)

- | 类内部 | 本包 | 子类 | 外部包
- | -| - | - | -
public | ✓ | ✓ | ✓ | ✓
protected | ✓ | ✓ | ✓ | x
default | ✓ | ✓ | x | x
private | ✓ | x | x | x

* package

* public: 接口访问权限

* private: 你无法访问

* protected: 继承访问权限

## 接口和实现

访问权限的控制常被称为是具体实现的隐藏。把数据和方法包装进类中，以及具体实现的隐藏，常共同被称作是**封装**。其结果是一个同时带有特征和行为的数据类型。

1. 设定客户端程序员可以使用和不可以使用的界限
2. 接口和具体实现进行分离

## 例子

```java
package other;

/**
 * @Author mubi
 * @Date 2019/6/1 9:15 PM
 */
public class Bowl {
    private int private_a;
    // 默认权限，defalt
    int b;
    public int public_c;
    protected int protected_d;

    public Bowl(int marker) {
        System.out.println(String.format("Bowl(marker:%d)", marker));
    }
}
```

* 外部包子类

```java
package com;

import other.Bowl;

public class BowlSonOther extends Bowl{

    public BowlSonOther(int marker) {
        super(marker);
        System.out.println(String.format("BowlSon(marker:%d)", marker));
    }

    public void funA(){
        System.out.println(super.public_c);
        System.out.println(super.protected_d);
    }
}
```

* 同一个包的子类

```java
package other;

public class BowlSon extends Bowl{

    public BowlSon(int marker) {
        super(marker);
        System.out.println(String.format("BowlSon(marker:%d)", marker));
    }

    public void funA(){
        System.out.println(super.b);
        System.out.println(super.public_c);
        System.out.println(super.protected_d);
    }
}
```

* 外部包Main

```java
package com;

import other.Bowl;

class Main {
    public static void main(String[] args) throws Exception {
        Bowl bowl = new Bowl(1);
        System.out.println(bowl.public_c);

        BowlSonOther bowlSonOther = new BowlSonOther(2);
        bowlSonOther.funA();
        System.out.println(bowlSonOther.public_c);
    }

}
```

* 同包的main

```java
package other;

public class Main2 {

    public static void main(String[] args) throws Exception {
        Bowl bowl = new Bowl(1);
        System.out.println(bowl.b);
        System.out.println(bowl.public_c);
        System.out.println(bowl.protected_d);

        BowlSon bowlSon = new BowlSon(2);
        bowlSon.funA();
        System.out.println(bowlSon.b);
        System.out.println(bowlSon.public_c);
        System.out.println(bowlSon.protected_d);
    }
}

```

* 不同包，子类方法能访问`protected`的成员，子类方法不能访问`default`的成员
