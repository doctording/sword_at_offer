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

## 等待通知机制(Lock & Condition)

在java中，对于任意一个java对象，它都拥有一组定义在`java.lang.Object`上监视器方法，包括`wait()`，`wait(long timeout)`，`notify()`，`notifyAll()`，这些方法配合`synchronized`关键字一起使用可以实现等待/通知模式。

同样，`Condition`接口也提供了类似`Object`监视器的方法，通过与`Lock`配合来实现等待/通知模式。

* lock
* condition.signal()
* condition.await()

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

    public static void main(String[] args) throws Exception{
        ReentrantLock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    lock.lock();
                    System.out.println("A start");
                    TimeUnit.SECONDS.sleep(3);
                    System.out.println("A notify");
                    condition.signal();
                } catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
                System.out.println("A end");
            }
        });

        Thread threadB = new Thread(()-> {
            try {
                lock.lock();
                System.out.println("B start");
                // 使得当前执行的线程B等待
                // 即：当前线程B 进入到 threadA对象的等待集合中 并等待唤醒。
                // B 释放其cpu给其它线程，自己让出资源进入等待池(A的等待集合中)等待
                condition.await();
                System.out.println("B wait end, restart running");
            } catch (Exception e) {
                System.out.println("B Exception");
                e.printStackTrace();
            }finally {
                lock.unlock();
            }
            System.out.println("B end");
        });

        threadB.start();
        // 确保B 先于A 运行
        TimeUnit.SECONDS.sleep(1);
        threadA.start();

        threadA.join();
        threadB.join();
        System.out.println("main end");
    }
}
```

* output

```java
B start
A start
A notify
A end
B wait end, restart running
B end
main end
```

## ReentrantLock (可重入锁)

* 公平锁

表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的FIFO先进先出顺序。

* 非公平锁

就是一种获取锁的抢占机制，是随机获得锁的，和公平锁不一样的就是先来的不一定先得到锁，这个方式可能造成某些线程一直拿不到锁，结果也就是不公平。

// TODO