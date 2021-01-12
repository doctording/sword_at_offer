---
title: "linux spin lock & mutex"
layout: page
date: 2019-03-15 00:00
---

[TOC]

# linux spin lock & mutex

* mutex: linux提供的互斥锁(pthread_mutex_t)
* spin lock: linux提供的自旋锁(pthread_spinlock_t)

拿不到锁，sleep，sleep要系统调用，是重量锁

CAS：硬件级别的指令，不进入内核态

Java的线程本质是内核线程，所以对于java线程的操作基本上是基于JVM--操作系统内核函数

JVM 解析 synchronized 的 monitor_enter 指令
1. 偏向锁：基于CAS
2. 非偏向：膨胀为重量锁,使用mutex

锁
* 死循环CPU耗在那里
* CPU上下文切换一下，不占CPU，放到一个等待队列，到时候唤醒它，然后在上下文切换回来
