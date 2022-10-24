---
title: "String交换问题"
layout: page
date: 2018-11-24 00:00
---

[TOC]

# Java字符串相关

## 不用反射，能否实现一个方法，调换两个 String 对象的实际值？

答案：否

```java
void swapStringTest(){
    String yesA = "a";
    String yesB = "b";
    //能否实现这个 swap 方法
    // 让yesA=b，yesB=a？
    swap(yesA, yesB);
    System.out.println("外部swap:" + yesA + "--" + yesB);
}

void swap(String a, String b){
    System.out.println("内部before swap:" + a + "--" + b);
    String tmp = a;
    a = b;
    b = tmp;
    System.out.println("内部after swap:" + a + "--" + b);
}
```

输出如下

```java
内部before swap:a--b
内部after swap:b--a
外部swap:a--b
```

<font color='red'>Java 只有值传递</font>,所以改成如下

```java
void swapStringTest(){
    int yesA = 1;
    int yesB = 2;
    //能否实现这个 swap 方法
    // 让yesA=b，yesB=a？
    swap(yesA, yesB);
    System.out.println("外部swap:" + yesA + "--" + yesB);
}

void swap(int a, int b){
    System.out.println("内部before swap:" + a + "--" + b);
    int tmp = a;
    a = b;
    b = tmp;
    System.out.println("内部after swap:" + a + "--" + b);
}
```

输出如下：

```java
内部before swap:1--2
内部after swap:2--1
外部swap:1--2
```

swap 里的 yesA 和 yesB 实际上是副本，不影响外部变量本身
