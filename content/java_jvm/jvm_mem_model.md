---
title: "Java 内存模型与线程"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# Java 内存模型与线程

## 硬件的效率 与 一致性

每个处理器都有自己的高速缓存，而它们又共享同一内存（Main Memory）；当多个处理器的运算任务都涉及同一块主内存区域时，将可能导致各自的缓存数据不一致

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_jvm/imgs/mem_cache.png)

## Java 主内存与工作内存

Java内存模型(JMM), 主要目标是定义各个变量（注意是共享变量，如实例字段，静态字段，构成数组对象的元素等，不包括局部变量）的访问规则，即在虚拟机中将变量存储到内存和从内存中取出变量这样的底层细节

Java的每条线程有自己的工作线程（Working Memory, 可类比处理器的高速缓存）， 线程的工作内存中保存了该线程使用到的变量的主内存副本拷贝，**线程对变量的所有操作（读取，赋值等）都必须在工作内存中进行，而不能直接读写主内存中的变量**。不同的线程之间也无法直接访问对方工作内存中的变量，线程间变量值的传递需要通过主内存

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_jvm/imgs/java_working_mem.png)

主内存与工作内存之间的具体交互协议，虚拟机保证如下的每一种操作都是原子的，不可再分的（对于double,long类型的变量有例外，商用JVM基本优化了这个问题）

* lock: 作用于主内存的变量，它把一个变量标识为一条线程独占的状态

* unlock：作用于主内存的变量， 解锁

* load：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中

* use：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作

* assign: 作用于工作内存的变量，它把一个从执行引擎接收到的赋值给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作

* store: 作用于工作内存的变量，它把工作内存中的一个变量的值传送到主内存中，以便随后的write操作使用

* write：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中

### volatile

当一个变量定义为`volatile`之后，具备两个特性

1. 保证此变量对所有线程的可见性（指当一条线程修改了这个变量的值，新值对于其它线程来说是可以立即知晓的。普通变量需要通过工作线程，主线程传递来做到）

在各个线程的工作线程中, volatile变量也可以存在不一致的情况，但由于每次使用之前都要先刷新，执行引擎看不到不一致的情况，因为可以认为不存在一致性问题，但是Java里面的运算并非都是原子操作，这导致volatile变量的运算在并发下一样是不安全的

```java
static int count = 20_000;
volatile int num = 0;
int coreSize = Runtime.getRuntime().availableProcessors();
ThreadPoolExecutor exec = new ThreadPoolExecutor(coreSize * 2,
        coreSize * 3, 0, TimeUnit.SECONDS,
        new LinkedBlockingQueue<Runnable>(500), new ThreadPoolExecutor.CallerRunsPolicy());

public void test() {
    for (int i = 0; i < count; i++) {
        exec.execute(new Runnable() {
            @Override
            public void run() {
                num++;
            }
        });
    }
    System.out.println("count:" + count);
    System.out.println("num:" + num);
    System.out.println("abs(num-count):" + Math.abs(count - num));
}

public static void main(String[] args) throws Exception{
    MainTest mainTest = new MainTest();
    mainTest.test();
}
```

* output

```java
count:20000
num:19763
abs(num-count):237
```

2. 禁止指令重排序优化，即线程内表现为串行的语义

volatile使用原则：

volatile变量读操作的性能消耗与普通变量几乎没有什么差别，但是写操作可能慢一些，因为它需要在本地代码中插入许多内存屏障来保证处理器不发生乱序执行。不过即便如此，大多数场景下volatile的总开销仍然比锁低，我们在volatile与锁之中选择的唯一依据仅仅是volatile的语义能否满足使用场景的要求

### 原子性，可见性，有序性

* 原子性

lock, unlock 对应 Java指令的`monitorenter`和`monitorexit`,`synchronized`关键字隐含了这两个指令，所以`synchronized`块之间的操作也具备了原子性

* 可见性

指当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。

当一个变量被volatile修饰是，它会保证修改的值会立即被更新到主存，当有其它线程需要读取时，他回去内存中读取新值。

`synchronized` 和 `final` 也能保证可见性

* 有序性

线程内串行有序 / 指令重排 / 工作内存与主内存同步延迟

`synchronized` 和 `volotile`

## 并行（parallellism） 和 并发（concurrency）

并发当有多个线程在操作时,如果系统只有一个CPU,则它根本不可能真正同时进行一个以上的线程，它只能把CPU运行时间划分成若干个时间段,再将时间 段分配给各个线程执行，在一个时间段的线程代码运行时，其它线程处于挂起状。.这种方式我们称之为并发(Concurrent)。

并行：当系统有一个以上CPU时,则线程的操作有可能非并发。当一个CPU执行一个线程时，另一个CPU可以执行另一个线程，两个线程互不抢占CPU资源，可以同时进行，这种方式我们称之为并行(Parallel)。
