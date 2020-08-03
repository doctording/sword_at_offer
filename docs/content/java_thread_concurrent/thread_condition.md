---
title: "Condition接口"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# Condition

## 实现原理

与`Object.wait()\Object.notify()`功能很类似。

以AQS非静态内部类的方式实现，因此`Condition`初始化的前提是先要有Lock实例，并且要先获取到锁

每个Condition对象都包含一个队列(等待队列)。等待队列是一个FIFO的队列，在队列中的每个节点都包含了一个线程引用，该线程就是在Condition对象上等待的线程

* <font color='red'>调用condition的await()方法后，会将当前线程加入到等待队列中，然后释放锁，然后循环判断节点是否在同步队列中，再获取锁，否则一直阻塞</font>

* <font color='red'>调用signal()方法后，先判断当前线程是否有锁，然后调用doSignal()方法，并唤醒线程，被唤醒的线程，再调用acquireQueude()方法，重新开始竞争锁，得到锁后返回，退出该方法</font>

### 为什么要有Condition？

`Condition`是在JDK5中出现的技术，使用它有更好的灵活性，比如可以实现**选择性通知功能**，也就是在<font color='red'>一个Lock对象里可以创建多个Condition实例，线程对象可以注册在指定的Condition中从而选择性的进行线程通知<font>，在调度线程上更加灵活。而<font color='red'>在使用notify()/notifuAll()方法进行通知时，被调度的线程却是由JVM随机选择的</font>。

## Condition接口方法

* await() ：造成当前线程在接到信号或被中断之前一直处于等待状态。

* await(long time, TimeUnit unit) ：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。

* awaitNanos(long nanosTimeout) ：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在nanosTimesout之前唤醒，那么返回值 = nanosTimeout - 消耗时间，如果返回值 <= 0 ,则可以认定它已经超时了。

* awaitUninterruptibly() ：造成当前线程在接到信号之前一直处于等待状态。【注意：该方法对中断不敏感】。

* awaitUntil(Date deadline) ：造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回返回false。

* signal() ：唤醒一个等待线程。该线程从等待方法返回前必须获得与Condition相关的锁。

* signal()All ：唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁。

## 仿写ArrayBlockingQueue，使用ReentrantLock & Condition

```java

import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

class BoundedArrayQueue<E> {
    Object[] items;

    /** items index for next take, poll, peek or remove */
    int takeIndex;

    /** items index for next put, offer, or add */
    int putIndex;

    /** Number of elements in the queue */
    int count;

    /** Main lock guarding all access */
    final ReentrantLock lock;

    /** Condition for waiting takes */
    private final Condition notEmpty; // 可理解为 读线程 锁

    /** Condition for waiting puts */
    private final Condition notFull;    // 可理解为 写线程 锁


    public BoundedArrayQueue(int size) {
        items = new Object[size];
        lock = new ReentrantLock();
        notEmpty = lock.newCondition();
        notFull =  lock.newCondition();
    }

    private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length) {
            putIndex = 0;
        }
        count++;
        notEmpty.signal();
    }

    private E dequeue() {
        // assert lock.getHoldCount() == 1;
        // assert items[takeIndex] != null;
        final Object[] items = this.items;
        E x = (E) items[takeIndex];
        items[takeIndex] = null;
        if (++takeIndex == items.length) {
            takeIndex = 0;
        }
        count--;
        notFull.signal();
        return x;
    }

    public void put(E e) throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
                System.out.println("now Array is Full. notFull await ...");
                notFull.await();
            }
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }

    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0) {
                System.out.println("now Array is Empty. notEmpty await ...");
                notEmpty.await();
            }
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

    public void print() {
        System.out.println(Arrays.toString(items));
    }
}

class Main {

    private static final Random random = new Random(System.currentTimeMillis());

    public static void main(String[] args) throws Exception {
        BoundedArrayQueue<String> boundedArrayQueue = new BoundedArrayQueue<>(3);

        //
        boundedArrayQueue.put("" + random.nextInt(10));
        System.out.print("producer: ");
        boundedArrayQueue.print();

        //
        boundedArrayQueue.take();
        System.out.print("consumer: ");
        boundedArrayQueue.print();

        //
        boundedArrayQueue.take();
        System.out.print("consumer: ");
        boundedArrayQueue.print();
    }
}
```
