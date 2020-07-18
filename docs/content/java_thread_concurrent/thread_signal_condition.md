---
title: "Thread Signal & Condition"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# Signal & Condition

与`Object.wait()\Object.notify()`功能很类似。

以AQS非静态内部类的方式实现，因此Condition初始化的前提是先要有Lock实例，并且要先获取到锁

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
