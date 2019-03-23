---
title: "常见的线程题目"
layout: page
date: 2019-03-12 00:00
---

[TOC]

# 三个线程：怎么能实现依次打印ABC的功能

## 方法1: ReentrantLock & Condition

```java
package com.thread;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author mubi
 * @Date 2019/3/12 10:34 AM
 * lock & condition 对比 synchronized wait notify
 */
public class Main {

    final static int N = 5;
    static int count = 0;
    static int flag = 1;

    public static void main(String[] args) throws Exception{

        ReentrantLock lock = new ReentrantLock();
        Condition conditionA = lock.newCondition();
        Condition conditionB = lock.newCondition();
        Condition conditionC = lock.newCondition();

        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                while(count < N) {
                    try {
                        lock.lock();
                        // 标志为1，A才能执行，否则阻塞自己
                        while (flag != 1){
                            conditionA.await();
                        }
                        System.out.print("A");
                        // 因为A打印完应该打印B， 这里线程B唤醒，同时设置标记
                        flag = 2;
                        conditionB.signal();
                    } catch (Exception e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            }
        });

        Thread threadB = new Thread(()-> {
            while(count < N) {
                try {
                    lock.lock();
                    while (flag != 2){
                        conditionB.await();
                    }
                    System.out.print("B");
                    flag = 3;
                    conditionC.signal();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        });

        Thread threadC = new Thread(()-> {
            while (count <= N) {
                try {
                    lock.lock();
                    while (flag != 3){
                        conditionC.await();
                    }
                    System.out.println("C" + count);
                    flag = 1;
                    count++;
                    conditionA.signal();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        });

        threadA.start();
        threadB.start();
        threadC.start();

        threadA.join();
        threadB.join();
        threadC.join();
        System.out.println("main end");
    }
}
```

## 方法2: synchronized & wait() notify() notifyAll()

```java
package com.thread;

/**
 * @Author mubi
 * @Date 2019/3/12 10:34 AM
 * lock & condition 对比 synchronized wait notify
 */
public class Main {

    final static int N = 5;
    static int count = 0;
    static int flag = 1;

    void methodA() {
        synchronized (this) {
            while (count < N) {
                try {
                    while (flag != 1) {
                        this.wait();
                    }
                    System.out.print("A");
                    flag = 2;
                    this.notify();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    void methodB() {
        synchronized (this) {
            while (count < N) {
                try {
                    while (flag != 2) {
                        this.wait();
                    }
                    System.out.print("B");
                    flag = 3;
                    this.notifyAll();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    void methodC() {
        synchronized (this) {
            while (count <= N) {
                try {
                    while (flag != 3) {
                        this.wait();
                    }
                    System.out.println("C" + count);
                    flag = 1;
                    count ++;
                    this.notifyAll();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws Exception{
        Main main = new Main();
        Thread threadA = new Thread(main::methodA);
        Thread threadB = new Thread(main::methodB);
        Thread threadC = new Thread(main::methodC);

        threadA.start();
        threadB.start();
        threadC.start();

        threadA.join();
        threadB.join();
        threadC.join();
        System.out.println("main end");
    }
}
```