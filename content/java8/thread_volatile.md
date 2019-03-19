---
title: "Thread volatile 关键字 & synchronized"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# volatile & synchrinized

## 回顾并发的三大性质

### 原子性

一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其它线程干扰.

`volatile` 不能保证原子性，
`synchronized` 在作用对象的作用范围内，依赖JVM实现操作的原子性。
`Lock` 依赖特殊的CPU指令，代码实现，如`ReentrantLock`

### 可见性

当多个线程访问同一个变量的时候，一旦线程修改了这个变量的值，其他线程能够立即看到修改的值。

导致共享变量在线程间不可见的原因：

1. 线程交叉执行
2. 代码重排序结合线程交叉执行
3. 共享变量更新后的值没有在工作内存与主内存之间及时更新

`volatile` 通过加入内存屏障和禁止重排序优化来实现可见性
`synchronized` monitor enter exit 确保可见性

### 有序性

程序执行的顺序按照代码的先后顺序执行

* Java内存模型中，允许编译器和处理器对指令进行重排序，但重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性

`volatile`、`syncronized`、`Lock`都可保证有序性。

参考：https://blog.csdn.net/qq_30948019/article/details/80193392

## 回顾 Java 内存模型

CPU -> 缓存 -> 主存 -> 线程工作内存

![](../java_jvm/jvm_mem_model.md)

## volatile 使用场景

volatile 无`原子性`，需要充分利用其的`顺序性`和`可见性`

### 利用可见性 进行开关控制

```java
class MyThread extends Thread{
    private volatile boolean started = true;

    @Override
    public void run() {
        while (started){
            // Todo
        }
    }

    public void shutdown(){
        this.started = false;
    }

}
```

### 利用顺序性

```java
线程A：
content = initContent();    //(1)
isInit = true;              //(2)
```

```java
线程B
while (isInit) {            //(3)
    content.operation();    //(4)
}
```

### Singleton 设计模式的 double-check 也是利用了属性性的特点
