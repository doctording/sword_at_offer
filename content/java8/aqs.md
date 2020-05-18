---
title: "AQS"
layout: page
date: 2020-20-15 00:00
---

[TOC]

# AbstractQueuedSynchronizer

参考：<a href="https://www.cnblogs.com/dennyzhangdd/p/7218510.html">《The java.util.concurrent Synchronizer Framework》 JUC同步器框架（AQS框架）原文翻译</a>

<a href="http://gee.cs.oswego.edu/dl/papers/aqs.pdf">论文地址</a>

## 同步器

两个操作

1. `acquire`操作阻塞调用的线程，直到或除非同步状态允许其继续执行。
2. `release`操作则是通过某种方式改变同步状态，使得一或多个被acquire阻塞的线程继续执行。

同步器需要支持如下：

* 阻塞和非阻塞（例如tryLock）的同步
* 可选的超时设置，让调用者可以放弃等待
* 通过中断实现的任务取消，通常是分为两个版本，一个acquire可取消，而另一个不可以

同步器的实现根据其状态是否**独占**而有所不同。独占状态的同步器，在同一时间只有一个线程可以通过阻塞点，而共享状态的同步器可以同时有多个线程在执行。一般锁的实现类往往只维护独占状态，但是，例如计数信号量在数量许可的情况下，允许多个线程同时执行。为了使框架能得到广泛应用，这两种模式都要支持。

j.u.c包里还定义了Condition接口，用于支持监控形式的await/signal操作，这些操作与独占模式的Lock类有关，且Condition的实现天生就和与其关联的Lock类紧密相关

## AbstractQueuedSynchronizer 相关概念

AbstractQueuedSynchronizer 抽象类的注释说明

1. AbstractQueuedSynchronizer 提供了一个框架，用来实现blocking locks 和 一些同步器，且是基于一个FIFO队列的
2. AbstractQueuedSynchronizer 被设计为使用一个`single atomic {@code int} value`来表示状态
3. AbstractQueuedSynchronizer的子类必须去定义状态，并提供protected方法去操作状态：getState、setState以及compareAndSet
4. 基于AQS的具体实现类必须根据暴露出的状态相关的方法定义tryAcquire和tryRelease方法，以控制acquire和release操作。当同步状态满足时，tryAcquire方法必须返回true，而当新的同步状态允许后续acquire时，tryRelease方法也必须返回true。这些方法都接受一个int类型的参数用于传递想要的状态

### 阻塞

j.u.c包有一个LockSuport类，这个类中包含了解决这个问题的方法。方法LockSupport.park阻塞当前线程除非/直到有个LockSupport.unpark方法被调用（unpark方法被提前调用也是可以的）。unpark的调用是没有被计数的，因此在一个park调用前多次调用unpark方法只会解除一个park操作。另外，它们作用于每个线程而不是每个同步器。一个线程在一个新的同步器上调用park操作可能会立即返回，因为在此之前可能有“剩余的”unpark操作。但是，在缺少一个unpark操作时，下一次调用park就会阻塞。虽然可以显式地消除这个状态但并不值得这样做。在需要的时候多次调用park会更高效。

**park**: n. 公园; 专用区; 园区; (英国) 庄园，庭院; v. 停(车); 泊(车); 坐下(或站着); 把…搁置，推迟(在以后的会议上讨论或处理);

### FIFO队列(CLH 队列)

整个框架的关键就是如何管理被阻塞的线程的队列，该队列是严格的FIFO队列，因此，框架不支持基于优先级的同步。
