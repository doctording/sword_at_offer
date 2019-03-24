---
title: "== & equals方法"
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

程序验证

```java
int a = 1;
int b = 1;
Integer ia = new Integer(a);
Integer ib = new Integer(b);
//  true
System.out.println(a == b);
// false
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

## `equals`方法是基类`Object`中的方法，因此对于所有的继承于`Object`的类都会有该方法。

* equals方法不能作用于基本数据类型的变量

* 如果没有对equals方法进行重写，则比较的是`引用类型的变量所指向的对象的地址`

* 诸如`String`、`Date`等类对`equals`方法进行了`override`的话，比较的是所指向的对象的内容
