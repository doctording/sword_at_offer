---
title: "Java 堆外内存"
layout: page
date: 2019-05-05 00:00
---

[TOC]

# 堆外内存

## 问题

`metaspace`没有限制，堆内存使用正常(没有`full gc`)，然后遇到`OOMKilled(程序因为内存使用超过限额被 kill -9 杀掉)`

## 概念

除了堆内存，Java 还可以使用堆外内存，也称直接内存（Direct Memory）。

例如：在通信中，将存在于堆内存中的数据 flush 到远程时，需要首先将堆内存中的数据拷贝到堆外内存中，然后再写入 Socket 中；如果直接将数据存到堆外内存中就可以避免上述拷贝操作，提升性能。类似的例子还有读写文件。

很多 NIO 框架 （如 netty，rpc） 会采用 Java 的 DirectByteBuffer 类来操作堆外内存，DirectByteBuffer 类对象本身位于 Java 内存模型的堆中，由 JVM 直接管控、操纵。DirectByteBuffer 中用于分配堆外内存的方法 unsafe.allocateMemory(size) 是个 native 方法，本质上是用 C 的 malloc 来进行分配的。

**堆外内存并不直接控制于JVM，因此只能等到full GC的时候才能垃圾回收！**（direct buffer归属的的JAVA对象是在堆上且能够被GC回收的，一旦它被回收，JVM将释放direct buffer的堆外空间。前提是没有关闭DisableExplicitGC）。堆外内存包含线程栈，应用程序代码，NIO缓存，JNI调用等.例如`ByteBuffer bb = ByteBuffer.allocateDirect(1024)`，这段代码的执行会在堆外占用`1k`的内存，Java堆内只会占用一个对象的指针引用的大小，堆外的这1k的空间只有当bb对象被回收时，才会被回收，这里会发现一个明显的不对称现象，就是堆外可能占用了很多，而堆内没占用多少，导致还没触发GC，那就很容易出现**Direct Memory造成物理内存耗光**

### 堆外内存溢出

1. 最大的堆外内存设置的太小了
2. 没有full gc， 堆外内存没有及时被清理掉

### 堆外内存更适合：

* 存储生命周期长的对象
* 可以在进程间可以共享，减少 JVM 间的对象复制，使得 JVM 的分割部署更容易实现。
* 本地缓存，减少磁盘缓存或者分布式缓存的响应时间。