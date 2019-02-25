---
title: "Java 堆内存结构"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# Java7堆内存划分：

##《深入理解java虚拟机》堆的描述

1. Java堆（Java Heap）是java虚拟机所管理的内存中最大的一块
2. Java堆被所有线程共享的一块内存区域
3. 虚拟机启动时创建java堆
4. Java堆的唯一目的就是存放对象实例。
5. Java堆是垃圾收集器管理的主要区域。
6. 从内存回收的角度来看， 由于现在收集器基本都采用分代收集算法， 所以Java堆可以细分为：新生代（Young）和老年代（Old）。 新生代又被划分为三个区域Eden、From Survivor， To Survivor等。无论怎么划分，最终存储的都是实例对象， 进一步划分的目的是为了更好的回收内存， 或者更快的分配内存。
7. Java堆的大小是可扩展的， 通过-Xmx和-Xms控制。
8. 如果堆内存不够分配实例对象， 并且对也无法在扩展时， 将会抛出outOfMemoryError异常。

## 堆区域

* 堆大小 = 新生代 + 老年代。堆大小设置参数：`–Xms`（堆的初始容量）、`-Xmx`（堆的最大容量）
* 其中，新生代 (`Young`) 被细分为`Eden`和两个`Survivor`区域，这两个`Survivor`区域分别被命名为`from`和`to`，以示区分。默认的，`Edem : from : to = 8 : 1 : 1`。(可以通过参数 –XX:SurvivorRatio 来设定 。即： Eden = 8/10 的新生代空间大小，from = to = 1/10 的新生代空间大小。
* JVM 每次只会使用`Eden`和其中的一块`Survivor`区域来为对象服务，所以无论什么时候，总是有一块`Survivor`区域是空闲着的。
* 新生代实际可用的内存空间为 9/10 ( 即90% )的新生代空间。

`jstat`查看gc相关的堆信息

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
  0.00   85.81  77.33  56.33      �    161    1.624    14    0.603    2.227
```

* E    — Heap上的 Eden space 区已使用空间的百分比(`Edem`)
* S0   — Heap上的 Survivor space 0 区已使用空间的百分比(`From`)
* S1   — Heap上的 Survivor space 1 区已使用空间的百分比(`To`)
* O    — Heap上的 Old space 区已使用空间的百分比(`Old`)
* P    — Perm space 区已使用空间的百分比（`Java7中的Perm Generation`）
* YGC  — 从应用程序启动到采样时发生 Young GC 的次数
* YGCT – 从应用程序启动到采样时 Young GC 所用的时间(单位秒)
* FGC  — 从应用程序启动到采样时发生 Full GC 的次数
* FGCT – 从应用程序启动到采样时 Full GC 所用的时间(单位秒)
* GCT  — 从应用程序启动到采样时用于垃圾回收的总时间(单位秒)

# 常见的垃圾回收算法

参考学习：

https://blog.csdn.net/qq_26437925/article/details/53728388

# 垃圾收集器

## 三个问题 ？

1. 哪些内存需要回收

2. 什么时候回收

3. 如何回收

## `可达性分析` 判定对象是否存活

算法的基本思路就是通过一系列的称为`GC Roots`的对象作为起始点，从这些节点向下搜索，搜索所走过的路径称为`引用链(Reference Chain)`,当一个对象到`GC Roots`没有任何引用链相连(用图论的话来说，就是`GC Roots`到这个对象不可达)时，则证明此对象是不可用的。

### Java中可作为`GC Roots`的对象包括：

* 虚拟机栈（栈帧中的本地变量表）中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 本地方法栈中JNI(即一般说的Native方法)引用的对象

## 引用`Reference`

强引用(Strong)，软引用(Soft)，弱引用(Weak)，虚引用(Phantom)；引用强度依次减弱

* 强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。如

```java
Object o=new Object();
```

* 软引用是用来描述一些还有用但并非必需的对象，对于软引用关联者的对象，在系统将要发生内存溢出异常之前，将会吧这些对象列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。`SoftReference`

```java
String str=new String("abc");                                     // 强引用
SoftReference<String> softRef=new SoftReference<String>(str);     // 软引用
```

* 弱引用也是用来描述非必需对象的。但是它的强度比弱引用更弱一些，被弱引用关联的对象只能生成到下一次垃圾收集之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。`WeakReference`

```java
String str=new String("abc");
WeakReference<String> abcWeakRef = new WeakReference<String>(str);
str=null;
```

* 虚引用也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的就是能够在这个对象被收集器回收时收到一个系统通知。`PhantomReference`

参考：https://www.cnblogs.com/fengbs/p/7019687.html

## 对象是生存还是死亡的？(两次标记)

对对象进行可达性分析 ？
=》没有GC Roots的引用链 =》 判断对象是否有必要执行`finalize()`方法

1. 对象没有覆盖`finalize()`方法，或者`finalize()`方法已经被虚拟机调用过,虚拟机将这两种情况都视为"没有必要执行"
2. 如果这个对象被判定为有必要执行`finalize()`方法,则这个对象会放置在一个叫做`F-Queue`的队列之中，并在稍后由一个虚拟机自动建立的，低优先级的`Finalizer线程`去执行它。

如果对象对象要在`finalize()`中成功拯救自己--只需要重新与引用链上的任何一个对象建立关联即可，GC对`F-Queue`中对对象进行第二次标记时，会将它移出"即将回收"的集合，否则会被回收

* 自救代码

```java
package com.java7;

/**
 * 此代码说明如下两点：
 * 1. 对象可以在GC时自我拯救
 * 2. 这种自救的机会只有一次，因为一个对象的`finalize()`方法最多只会被系统自动调用一次
 */
public class FinalizeEscapeGC {

    public static FinalizeEscapeGC SAVE_HOOK = null;
    public String name;
    public FinalizeEscapeGC(String name) {
        this.name = name;
    }
    public void isAlive() {
        System.out.println("yes, i am still alive");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed!");
        System.out.println(this.name);
        FinalizeEscapeGC.SAVE_HOOK = this;
    }
    public static void main(String[] args) throws Throwable {
        SAVE_HOOK = new FinalizeEscapeGC("abc");
        // 对象第一次成功拯救自己
        SAVE_HOOK = null;
        System.gc();
        // finalize方法的优先级很低，暂停0.5秒等待它
        Thread.sleep(500);
        if(SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("no, i am dead");
        }
        // 下面这段代码与上面完全相同，但是这次自救却失败了
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if(SAVE_HOOK != null) {
            SAVE_HOOK.isAlive();
        } else {
            System.out.println("no, i am dead");
        }
    }
}
```

* output

```bash
finalize method executed!
abc
yes, i am still alive
no, i am dead
```

* 最后，不推荐`finalize()`

## 回收方法区

堆中，新生代进行一次垃圾收集一般可以回收70%-95%的空间，而永久代的垃圾收集效率远低于此。

永久代主要回收两部分内容： 废弃常量和无用的类， 判断"无用的类"：

1. 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例
2. 加载该类的ClassLoader已经被回收
3. 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

是否对类进行回收，HotSpot虚拟机提供了`-Xnoclassgc`参数进行控制，还可以使用`-verbose:class`以及`-XX:+TraceClassLoading`,`-XX:+TraceClassUnLoading`查看类加载和卸载信息，其中`-verbose:class`和`-XX:+TraceClassLoading`可以在Product版的虚拟机中使用，`-XX:+TraceClassUnLoading`参数需要FastDebug版的虚拟机支持

在大量使用反射，动态代理，CGLib等ByteCode框架，动态生成JSP以及OSGi这类频繁自定义ClassLoader的场景都需要虚拟机具备类卸载的功能，以保证永久代不会溢出

## HotSpot 算法实现

可达性分析的"一致性"分析，GC进行时必须停顿所有Java执行线程(Sun将这件事情称为`Stop The World`)

虚拟机有办法直接得知哪些地方存放着对象的引用，HotSpot中使用一组称为`OopMap`的数据结构来达到这个目的：在类加载完成的时候，HotPot就把对象内什么偏移量是什么类型的数据计算出来，在JIT编译过程中，也会在特定的位置记录下栈和寄存器中哪些位置是引用。这样，GC在扫描时就可以直接得知这些信息了。

借助OopMap，HotSpot可以快速且准确地完成GC Roots枚举，问题：

* 可能导致引用关系变化，或者说OopMap内容变化的指令特别多，如果为每一条指令都生成对应的OopMap，那将会需要大量的额外空间，这样GC的空间成本将会变得很高

实际上，HotSpot没有为每条指令都生成`oopMap`, 安全点(`Sagepoint`) GC， 选举以"是否具有让程序长时间执行的特征"为标准进行选定，如方法调用，循环跳转，异常跳转等，这些功能的指令会产生安全点

# 收集器
