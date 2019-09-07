---
title: "Reactor 模式"
layout: page
date: 2019-06-07 00:00
---

[TOC]

# Reactor 模式

参考：https://www.cnblogs.com/doit8791/p/7461479.html

## 传统做法回顾

### 服务器一个while，单线程处理客户端所有的请求

```java
while(true){
    socket = accept();
    handle(socket)
}
```

### 多线程/线程池，每个请求分配一个线程处理

```java
while(true){
    socket = accept();
    new thread(socket);
}
```

线程同步的粒度太大了，限制了吞吐量。应该把一次连接的操作分为更细的粒度或者过程，这些更细的粒度是更小的线程。整个线程池的数目会翻倍，但是线程更简单，任务更加单一

### 事件驱动

事件驱动程序的基本结构是由一个事件收集器、一个事件发送器和一个事件处理器组成。

* 事件收集器专门负责收集所有事件
* 事件发送器负责将收集器收集到的事件分发到目标对象中
* 事件处理器做具体的事件响应工作

### Reactor

在Reactor中，这些被拆分的小线程或者子过程对应的是`handler`，每一种`handler`会出处理一种`event`。

这里会有一个全局的管理者`selector`，我们需要把`channel`注册感兴趣的事件，那么这个`selector`就会不断在`channel`上检测是否有该类型的事件发生，如果没有，那么主线程就会被阻塞，否则就会调用相应的事件处理函数即`handler`来处理。
