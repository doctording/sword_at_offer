---
title: "常见的线程题目"
layout: page
date: 2019-03-12 00:00
---

[TOC]

# 三个线程：怎么能实现依次打印ABC的功能

## 方法1: ReentrantLock & Condition（条件锁）

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

## 方法2: synchronized & wait() notify() notifyAll()（状态同步 + wait/notify）

```java

public class Main{

    static int N = 2;

    static int flag = 3;
    static Object object = new Object();

    public static void main(String[] args) throws Exception {

        Thread A = new Thread(()-> {
            synchronized (object) {
                for(int i=0;i<N;i++){
                    try {
                        //等待前一个的打印处理
                        while (flag != 3) {
                            object.wait();
                        }
                        System.out.print(Thread.currentThread().getName() + " ");
                        flag = 1;
                        // 唤醒其它线程
                        object.notifyAll();
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }, "A");

        Thread B = new Thread(()-> {
            synchronized (object) {
                for(int i=0;i<N;i++){
                    try {
                        //等待前一个的打印处理
                        while (flag != 1) {
                            object.wait();
                        }
                        System.out.print(Thread.currentThread().getName() + " ");
                        flag = 2;
                        // 唤醒其它线程
                        object.notifyAll();
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }, "B");

        Thread C = new Thread(()-> {
            synchronized (object) {
                for(int i=0;i<N;i++){
                    try {
                        //等待前一个的打印处理
                        while (flag != 2) {
                            object.wait();
                        }
                        System.out.print(Thread.currentThread().getName() + " ");
                        flag = 3;
                        // 唤醒其它线程
                        object.notifyAll();
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }, "C");

        A.start();
        B.start();
        C.start();

        A.join();
        B.join();
        C.join();
    }

}
```

## 方法3: 信号量

```java
class FooBar {
    private int n;
    Semaphore fooSemaphore = new Semaphore(1);
    Semaphore barSemaphore = new Semaphore(0);

    public FooBar(int n) {
        this.n = n;
    }

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            fooSemaphore.acquire();
            // printFoo.run() outputs "foo". Do not change or remove this line.
            printFoo.run();
            barSemaphore.release();
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            barSemaphore.acquire(); // 初始为0, barSemaphore.acquire 会阻塞等待
            // printBar.run() outputs "bar". Do not change or remove this line.
            printBar.run();
            fooSemaphore.release();
        }
    }
}
```
