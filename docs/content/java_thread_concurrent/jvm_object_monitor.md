---
title: "ObjectMonitor"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# ObjectMonitor

## Java线程回顾

open jdk源码地址：https://github.com/openjdk/jdk

Java线程（Java Thread）是由Java虚拟机（JVM）创建和管理的，它是Java程序中最基本的执行单元。Java线程和操作系统线程（OS Thread）是不同的概念。在HotSpot中，Java线程实际上是由JavaThread类表示的。JavaThread类是Thread类的子类，它继承了Thread类的一些属性和方法，并添加了一些额外的属性和方法，用于实现Java线程的特性，如线程状态、调用栈、异常处理等。JavaThread类的实例代表了一个Java线程。

而OS Thread类则是一个抽象类，它封装了HotSpot对操作系统线程的抽象和管理，如线程ID、优先级、调度等。Java Thread类包含了一个OS Thread类的成员变量，用于表示Java线程所对应的操作系统线程。**Java线程和操作系统线程之间的关系是一对一的**。每个Java线程都会有一个对应的操作系统线程来执行它的任务。因此，Java线程的生命周期受操作系统线程的调度和管理。

在`/hotspot/src/share/vm/runtime/thread.cpp`中：Thread对象内部包含了该线程的状态、调度优先级、执行栈、栈帧、持有的锁等信息。

## ObjectMonitor对象

每个Java对象中都具有一个一个`ObjectMonitor`对象。在Java中，每个对象都可以用作锁来同步多个线程的访问。当线程获取某个对象的锁时，它实际上是获取该对象关联的ObjectMonitor对象的锁。因此，每个对象在Java中都有一个与之关联的`ObjectMonitor`对象来控制线程对该对象的访问。

### ObjectMonitor主要对象成员

* _object 指向被监视的对象，即 Java 层面的对象
* _owner 指向持有ObjectMonitor对象的线程地址。
* _WaitSet 一个 ObjectWaiter 对象的链表，用于存储被阻塞的线程由于 wait() 或 join() 等待 monitor 的状态
* _EntryList 一个 ObjectWaiter 对象的链表，用于存储被阻塞(block住)的线程等待 monitor 进入
* _recursions 锁的重入次数。
* _count 线程获取锁的次数。

### ObjectWaiter

ObjectWaiter是一个用于等待唤醒的数据结构。在Java中，Object.wait() 方法调用后，线程会被挂起，直到另一个线程调用Object.notify() 或 Object.notifyAll() 方法，或者线程等待时间到期，或者线程被中断，才会被唤醒。当一个线程调用Object.wait() 方法后，会创建一个ObjectWaiter对象，该对象会被加入到等待队列中。当另一个线程调用Object.notify() 或 Object.notifyAll() 方法时，会从等待队列中取出一个或多个ObjectWaiter对象，并将它们加入到可用队列中，以便在下一次竞争锁时唤醒这些线程。

### ObjectMonitor的基本工作机制

1. 当多个线程同时访问一段同步代码时，首先会进入 _EntryList 队列中。

2. 当某个线程获取到对象的Monitor后进入临界区域，并把Monitor中的 _owner 变量设置为当前线程，同时Monitor中的计数器 _count 加1。即获得对象锁。

3. 若持有Monitor的线程调用 wait() 方法，将释放当前持有的Monitor，_owner变量恢复为null，_count自减1，同时该线程进入 _WaitSet 集合中等待被唤醒。

4. 在_WaitSet 集合中的线程会被再次放到_EntryList 队列中，重新竞争获取锁。

5. 若当前线程执行完毕也将释放Monitor并复位变量的值，以便其它线程进入获取锁

### 获取锁 ObjectMonitor::enter

1. 首先如果没有线程使用这个锁则，直接获取锁，
2. 若有线程是会尝试通过原子操作来将当前线程设置成此对象的监视器锁的持有者。
    * 如果原来的持有者是 null，则当前线程成功获取到了锁。
    * 如果原来的持有者是当前线程，则说明当前线程已经持有该锁，并且将**计数器递增**；
    * 如果原来的持有者是其它线程，则说明存在多线程竞争，代码会将当前线程阻塞，并且进入一个等待队列中等待被唤醒。如果开启了自旋锁，则会尝试自旋一段时间，以避免多线程竞争导致的阻塞开销过大。
    * 如果自旋后仍未获得锁，则当前线程将进入一个等待队列中，并且设置自己为队列的尾部。等待队列中的线程按照LIFO（避免头部饥饿）的顺序进行排队。当持有者释放锁时，队列头的线程将被唤醒并尝试重新获取锁。

### 释放锁 ObjectMonitor::exit

用于释放当前线程占用的 monitor 并唤醒等待该 monitor 的其它线程

1. 检查当前线程是否持有该锁。如果没有持有该锁，会对其进行修复（假设线程实际上持有该锁，但是由于某些原因，owner字段没有正确更新）或抛出异常（如果线程没有正确地获取该锁，即不在_owner字段中）。
2. 如果当前线程是多次重入该锁，将计数器减1，并直接返回。这是因为线程实际上仍然持有该锁。
3. 检查是否有其它线程等待该锁。如果没有等待线程，直接将_owner字段设置为null并返回。如果有等待线程，则释放该锁，并使等待线程之一成为新的owner。
4. 如果等待线程中有线程使用了公平自旋（Ticket Spinlock算法），则使用该算法来释放该锁。否则，使用等待队列或Cache Exclusive Queue（CXQ）算法来释放该锁。这些算法可以更有效地处理多个线程对同一对象锁的竞争，从而提高性能。
