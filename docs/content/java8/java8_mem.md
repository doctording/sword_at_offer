---
title: "Java8 jvm内存结构"
layout: page
date: 2019-02-16 00:00
---

[TOC]

# Java8内存变化

```java
^Cmubi@mubideMacBook-Pro Home $ pwd
/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
mubi@mubideMacBook-Pro Home $ bin/jstat -gcutil 62850 1000 10
Warning: Unresolved Symbol: sun.gc.generation.2.space.0.capacity substituted NaN
Warning: Unresolved Symbol: sun.gc.generation.2.space.0.used substituted NaN
Warning: Unresolved Symbol: sun.gc.generation.2.space.0.capacity substituted NaN
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
  0.00  85.81  76.97  56.33      �    161    1.624    14    0.603    2.227
  0.00  85.81  77.08  56.33      �    161    1.624    14    0.603    2.227
  0.00  85.81  77.22  56.33      �    161    1.624    14    0.603    2.227
  0.00  85.81  77.33  56.33      �    161    1.624    14    0.603    2.227
^Cmubi@mubideMacBook-Pro Home $ jstat -gcutil 62850 1000 10
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00  85.81  78.86  56.33  93.46  89.80    161    1.624    14    0.603    2.227
  0.00  85.81  78.93  56.33  93.46  89.80    161    1.624    14    0.603    2.227
  0.00  85.81  79.04  56.33  93.46  89.80    161    1.624    14    0.603    2.227
  0.00  85.81  79.17  56.33  93.46  89.80    161    1.624    14    0.603    2.227
  0.00  85.81  79.29  56.33  93.46  89.80    161    1.624    14    0.603    2.227
^Cmubi@mubideMacBook-Pro Home $
```

java7:

* S0  — Heap上的 Survivor space 0 区已使用空间的百分比
* S1  — Heap上的 Survivor space 1 区已使用空间的百分比
* E   — Heap上的 Eden space 区已使用空间的百分比
* O   — Heap上的 Old space 区已使用空间的百分比
* P   — Perm space 区已使用空间的百分比
* YGC — 从应用程序启动到采样时发生 Young GC 的次数
* YGCT– 从应用程序启动到采样时 Young GC 所用的时间(单位秒)
* FGC — 从应用程序启动到采样时发生 Full GC 的次数
* FGCT– 从应用程序启动到采样时 Full GC 所用的时间(单位秒)
* GCT — 从应用程序启动到采样时用于垃圾回收的总时间(单位秒)

## Java8: 没有了永久代的概念

少了一个 Perm space（永久代）

Java7中方法区（Method Area）,与Javad堆一样，是各个线程共享的内存区域，主要用于存储已经被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。在HotSpot虚拟机上，通常把方法区称为"永久代"(Permanent Generation), 本质上两者并不等价，仅仅是因为HotSpot虚拟机的设计团队选择把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已。这样HotSpot的垃圾收集器可以像管理Java堆一样管理这部分内存，能够省去专门为方法区编写内存管理代码的工作。

java7中，符号引用(Symbols)转移到了native heap；字面量(interned strings)转移到了java heap；类的静态变量(class statics)转移到了java heap。

Java7的`Perm space`在堆中，设置小了，容易`OutOfMemory`；设置大了比较浪费堆内存

## Java8 新增了 Metaspace（元空间）

* M   - 元空间（Metaspace）： Klass Metaspace, NoKlass Metaspace
* CCS - 表示的是NoKlass Metaspace的使用率

元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制，但可以通过参数来指定元空间的大小。

### Klass Metaspace

存放klass的，klass是我们熟知的class文件在jvm里的运行时数据结构，这个空间的默认大小是1G

### NoKlass Metaspace

专门来存klass相关的其他的内容，比如method，constantPool（常量池）等，这块内存是由多块内存组合起来的，所以可以认为是不连续的内存块组成的。这块内存是必须的

* `-XX:MetaspaceSize`，初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize时，适当提高该值。

* `-XX:MaxMetaspaceSize`，最大空间，默认是没有限制的。

* `-XX:MinMetaspaceFreeRatio`，在GC之后，最小的Metaspace剩余空间容量的百分比，减少为分配空间所导致的垃圾收集

* `-XX:MaxMetaspaceFreeRatio`，在GC之后，最大的Metaspace剩余空间容量的百分比，减少为释放空间所导致的垃圾收集

### 替换理由 //TODO

1. <a href="http://openjdk.java.net/jeps/122">http://openjdk.java.net/jeps/122</a>

2. <a href="https://blog.csdn.net/wodewutai17quiet/article/details/80746103">Java8中的metaspace</a>

### 元空间溢出

元空间存放什么：
  * 虚拟机加载的类信息
  * 常量池
  * 静态变量
  * 即时编译后的代码

导致元空间溢出的可能原因：
  * 大量类加载
  * 代理，cglib导致的大量类生成
