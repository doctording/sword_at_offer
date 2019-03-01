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

<a href='https://blog.csdn.net/qq_26437925/article/details/53728388'>几种垃圾回收算法</a>

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

垃圾收集器，并发&并行

* 并行(Parallel)： 指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态

* 并发(Concurrent): 只用户线程与垃圾收集线程同时执行（但不一定是并行但，可能会交替执行），用户程序在继续运行，而垃圾收集程序运行于另一个CPU上

## CMS(Concurrent Mark Sweep)

* 并发收集，低停顿

一种以获取最短回收停顿时间为目标但收集器：希望系统停顿时间最短，给用户带来较好但体验，4个步骤如下：

1. 初始标记（CMS initial mark）
2. 并发标记（CMS concurrent mark）
3. 重新标记（CMS remark）
4. 并发清除（CMS concurrent sweep）

1,3两个步骤仍需要"Stop The World"

* 缺点

1. CMS收集器对CPU资源非常敏感。在并发阶段，虽然不会导致用户线程停顿，但是会因为占用了一部分线程使应用程序变慢，总吞吐量会降低，为了解决这种情况，虚拟机提供了一种"增量式并发收集器"(Incremental Concurrent Mark Sweep/i-CMS)的CMS收集器变种，所做对事情就是在并发标记和并发清除的时候让GC线程和用户线程交替运行，尽量减少GC线程独占资源的时间，这样整个垃圾收集的过程会变长，但是对用户程序的影响会减少。（效果不明显，已经不推荐）

2. CMS处理器无法处理浮动垃圾（Floating Garbage）。由于CMS在并发清理阶段有用户线程还在运行这，伴随着程序的运行自然也会产生新的垃圾，这一部分垃圾产生在标记过程之后，CMS无法再当次手机中处理掉它们，所以只有等到下次gc时候再清理掉，这一部分垃圾就称作"浮动垃圾"，因此CMS收集器不能像其它收集器那样等到老年代几乎完全被填满了再进行收集，而是需要预留一部分空间提高并发收集时的程序运作使用。

3. CMS是基于"标记--清除"算法实现的，所以在收集结束的时候会有大量的`空间碎片`产生。空间碎片太多的时候，将会给大对象的分配带来很大的麻烦，往往会出现老年代还有很大的空间剩余，但是无法找到足够大的连续空间来分配当前对象的，只能提前触发 full gc。

为了解决这个问题，CMS提供了一个开关参数（`-XX: UseCMSCompactAtFullCollection`），用于在CMS顶不住要进行full gc的时候开启内存碎片的合并整理过程，内存整理的过程是无法并发的，空间碎片没有了，但是停顿的时间变长了。另外一个参数(`-XX: CMSFullGCsBeforeCompaction`)用于设置执行多少次不压缩的full gc后，跟着来一次带压缩的（默认值为0，表示每次进入full gc时都进行碎片整理）

## G1(Garbage First)

* 并行与并发

G1能充分利用多CPU,多核环境下的硬件优势，使用多个CPU来缩短`Stop-The-World`停顿的时间，部分其它收集器原本需要停顿Java线程执行GC动作，G1收集器仍然可以通过并发的方式让Java程序继续执行

* 分代收集

虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了分代的概念。它能够采用不同的方式去处理新创建的对象和已经存活了一段时间，熬过多次GC的旧对象以获取更好的收集效果。

* 空间整合

与CMS的"标记--清理"算法不同，G1从整体来看是基于"标记整理"算法实现的收集器；从局部上来看是基于"复制"算法实现的。但无论如何，这两种算法都意味着G1运作期间不会产生内存空间碎片，收集后能够提供规整的可用内存

* 可预测的停顿

这是G1相对于CMS的另一个大优势，降低停顿时间是G1和CMS共同的关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，

运作大致如下步骤：

1. 初始标记（Initial Marking）
2. 并发标记（Concurrent Markding）
3. 最终标记（Final Marking）
4. 筛选回收（Live Data Counting and Evacuation）

## 内存分配与回收策略

### 对象优先在Eden分配

大多数情况下，对象在新生代Eden区中分配。当Eden区没有足够空间进行分配是，虚拟机将发起一次`Minor GC`。（`-XX:+PrintGCDetails`）

#### minor gc

新生代GC(Minor GC): 指发生在新生代的垃圾收集动作，因为Java对象大多数都具备朝生夕灭的特效，所以`Minor GC`非常频繁，一般回收速度也比较快

新创建的对象都是用新生代分配内存，Eden空间不足时，触发Minor GC，这时会把存活的对象转移进Survivor区。

新生代通常存活时间较短基于Copying算法进行回收，所谓Copying算法就是扫描出存活的对象，并复制到一块新的完全未使用的空间中，对应于新生代，就是在Eden和From Space或To Space之间copy。新生代采用空闲指针的方式来控制GC触发，指针保持最后一个分配的对象在新生代区间的位置，当有新的对象要分配内存时，用于检查空间是否足够，不够就触发GC。当连续分配对象时，对象会逐渐从Eden到Survivor，最后到老年代。

#### major gc/ full gc

老年代GC(Major GC/Full GC): 指发生在老年代的GC,出现了`Major GC`, 经常会伴随至少一次的`Minor GC`(但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行`Major GC`的策略选择过程)。Major GC的速度一般会比Minor GC慢10倍以上

老年代用于存放经过多次Minor GC之后依然存活的对象。

老年代与新生代不同，老年代对象存活的时间比较长、比较稳定，因此采用标记(Mark)算法来进行回收，所谓标记就是扫描出存活的对象，然后再进行回收未被标记的对象，回收后对用空出的空间要么进行合并、要么标记出来便于下次进行分配，总之目的就是要减少内存碎片带来的效率损耗。

* -Xms 默认情况下堆内存的64分之一
* -Xmx 默认情况下对内存的4分之一
* -Xmn 默认情况下堆内存的64分之一， 新生代大小，该配置优先于-XX:NewRatio，即如果配置了-Xmn，-XX:NewRatio会失效。
* -XXNewRatio 默认为2
* -XX:SurvivorRatio 默认为8，表示Suvivor:eden=2:8,即一个Survivor占年轻代的1/10

#### Java GC测试程序 和 初始堆情况

* jdk1.7.0_80

```java
/*
 * -verbose:gc -Xms20M -Xmx20M -Xmn10M  -XX:+PrintGCDetails -XX:SurvivorRatio=8
 */
public class MainTest {
    public static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws Exception{
        byte[] alloc1, alloc2, alloc3, alloc4;
        alloc1 = new byte[2 * _1MB];
        alloc2 = new byte[2 * _1MB];
        alloc3 = new byte[2 * _1MB];
        // 出现 GC
        alloc4 = new byte[3 * _1MB];
    }

}
```

```js
-verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8
```

eden | from survivor | to survivor | old
-|-|-|-
8192K | 1024K | 1024K | 10240K

#### GC日志

```s
[GC [PSYoungGen: 7534K->416K(9216K)] 7534K->6560K(19456K), 0.0049590 secs] [Times: user=0.01 sys=0.00, real=0.01 secs]
[Full GC [PSYoungGen: 416K->0K(9216K)] [ParOldGen: 6144K->6466K(10240K)] 6560K->6466K(19456K) [PSPermGen: 3121K->3120K(21504K)], 0.0103590 secs] [Times: user=0.02 sys=0.00, real=0.01 secs]
Heap
 PSYoungGen      total 9216K, used 3403K [0x00000007ff600000, 0x0000000800000000, 0x0000000800000000)
  eden space 8192K, 41% used [0x00000007ff600000,0x00000007ff952de0,0x00000007ffe00000)
  from space 1024K, 0% used [0x00000007ffe00000,0x00000007ffe00000,0x00000007fff00000)
  to   space 1024K, 0% used [0x00000007fff00000,0x00000007fff00000,0x0000000800000000)
 ParOldGen       total 10240K, used 6466K [0x00000007fec00000, 0x00000007ff600000, 0x00000007ff600000)
  object space 10240K, 63% used [0x00000007fec00000,0x00000007ff250b48,0x00000007ff600000)
 PSPermGen       total 21504K, used 3142K [0x00000007f9a00000, 0x00000007faf00000, 0x00000007fec00000)
  object space 21504K, 14% used [0x00000007f9a00000,0x00000007f9d118a0,0x00000007faf00000)
```

* gc过程

状态 | eden | from survivor | to survivor | old
-|-|-|-|-
初始状态总大小 |8192K | 1024K | 1024K | 10240K
分配了alloc1,2,3 | 使用了6144K
接着需要分配alloc4，需要3M,不够用，GC |
内存重新分配 | 3M存alloc4 | - | - | old使用了6M用来存alloc1,2,3
最后使用占比 | 37%多 | 0% | 0% | 60%多

#### 参考：

JVM GC log文件的查看
https://www.cnblogs.com/xuezhiyizu1120/p/6237510.html

* [PSYoungGen: 7698K->448K(9216K)]
    1. PSYoungGen 新生代/
    2. GC前该内存区域已使用容量 -> GC后该内存区域已使用容量(该内存区域的总容量)。
* 7698K->6592K(19456K)
    1. GC前Java堆已使用容量 -> GC后Java堆已使用容量（Java堆总容量）。

### 大对象直接进入老年代

这里所谓的大对象是指，需要大量连续内存空间的Java对象，，最典型的大对象就是那种很长的字符串以及数组。大对象对虚拟机的内存分配来说就是一个坏消息。（更坏的：一群"朝生夕灭"的"短命大对象"），经常出现大对象容易导致内存还有不少空间时就提前触发垃圾收集以获取足够的连续空间来"安置"它们

`-XX:PretenureSizeThreshold`参数，另大于这个设置值的对象直接在老年代分配。这样做的目的是避免在Eden区以及两个Survivor区之间发生大量的内存复制(复习一下：新生代采用复制算法收集内存)

### 长期存活的对象将进入老年代

内存回收时，必须识别哪些对象应该在新生代，哪些对象应放到老年代。

虚拟机给每个对象定义了一个对象年龄(`Age`)计数器。如果对象在`Eden`出生并经历过第一次`Minor GC`后仍然存活，并且能被`Survivor`容纳的话，将被移动到`Survivor`空间,并且对象的年龄为1.对象在`Survivor`区中每"熬过"一次`Minor GC`，年龄就增加1岁，当他的年龄增加到一定程度（默认15岁），就将会被晋升到老年代中。对象晋升老年代的年龄阈值，可以通过参数`-XX:MaxTenuringThreshold`设置

### 动态对象的年龄判定

为了更好地适应不同程序的内存状况，虚拟机并不是永远地要求对象的年龄必须达到了`MaxTenuringThreshold`才能晋升老年代，如果在`Survivor`空间中相同年龄所有对象大小的总和大于`Survivor`空间的一半，年龄大于或等于该年龄对象就可以直接进入老年代，无须等到`MaxTenuringThreshold`要求的年龄。

### 空间分配担保

老年代最大可用的连续空间 > 新生代所有对象总空间, 这样能确保`Minor GC`是安全的。

如果不成立，虚拟机会查看`HandlerPromotionFailure`设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小

如果大于，将尝试着进行一次`Minor GC`,尽管这次的`Minor GC`是有风险的

如果小雨，或者`HandlerPromotionFailure`设置值不允许担保失败，这时改为进行一次`Full GC`

