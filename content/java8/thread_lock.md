---
title: "Thread lock"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# lock

## lock基本用法例子

* lock, unlock

```java
package com.thread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    private final static Object Mutex = new Object();
    public int num;
    static int n = 16;

    public static ReentrantLock lock = new ReentrantLock();

    void add(){
        try {
            TimeUnit.SECONDS.sleep(1);
        }catch (Exception e){
            e.printStackTrace();
        }
        this.num++;
    }

    void addSync(){
        synchronized (Mutex){
            this.num++;
        }
    }

    void addLock(){
        //获取锁
        lock.lock();
        try{
            this.num++;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

    void printNum(){
        System.out.println("num:" + this.num);
    }

    public static void main(String[] args) throws Exception{
        Main main = new Main();
        main.num = 0;
        List<Thread> threadList = new ArrayList<>(n);
        for(int i=0;i<n;i++){
//            Thread t = new Thread( ()-> main.add(), "myname" + i);
            Thread t = new Thread( ()-> main.addLock(), "myname" + i);
//            Thread t = new Thread(()-> main.addSync());
            threadList.add(t);
        }
        threadList.forEach(t->t.start());
        threadList.forEach(t->{
            try{
                t.join();
            }catch (Exception e){
                e.printStackTrace();
            }
        });
        main.printNum();
    }
}
```

* tryLock(long time, TimeUnit unit)

```java
package com.thread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    private final static Object Mutex = new Object();
    public int num;
    static int n = 16;

    public static ReentrantLock lock = new ReentrantLock();

    void addLock(){
        try{
            lock.lock();
            System.out.println(Thread.currentThread().getName()+"获得锁");
            TimeUnit.SECONDS.sleep(2);
            this.num ++;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
            System.out.println(Thread.currentThread().getName() + "释放锁");
        }
    }

    void subLock(){
        try{
            if(lock.tryLock(1, TimeUnit.SECONDS)){
                System.out.println(Thread.currentThread().getName()+"获得锁");
                this.num --;
            }else{
                System.out.println(Thread.currentThread().getName()+"未获得锁");
            }
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if(lock.isHeldByCurrentThread()){
                lock.unlock();
            }
        }
    }

    void printNum(){
        System.out.println("num:" + this.num);
    }

    public static void main(String[] args) throws Exception{
        Main main = new Main();
        main.num = 0;
        List<Thread> threadList = new ArrayList<>(n);
        for(int i=0;i<n;i++){
            if(i % 2 == 0) {
                Thread t = new Thread(() -> main.addLock(), "myname" + i);
                threadList.add(t);
            }else {
                Thread t = new Thread(() -> main.subLock(), "myname" + i);
                threadList.add(t);
            }
        }
        threadList.forEach(t->t.start());
        threadList.forEach(t->{
            try{
                t.join();
            }catch (Exception e){
                e.printStackTrace();
            }
        });
        main.printNum();
    }
}
```

## 等待通知机制

在java中，对于任意一个java对象，它都拥有一组定义在java.lang.Object上监视器方法，包括wait()，wait(long timeout)，notify()，notifyAll()，这些方法配合synchronized关键字一起使用可以实现等待/通知模式。

同样，Condition接口也提供了类似Object监视器的方法，通过与Lock配合来实现等待/通知模式。

## ReentrantLock (可重入锁)
