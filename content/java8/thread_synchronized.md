---
title: "Thread Synchronized & Monitor"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# Synchronized & monitor

## synchronized的几个性质

* `synchronized` 提供了一种锁对机制，能确保共享变量的互斥访问，从而防止数据不一致问题的出现

* `synchronized` 包括了`monitor enter`和`monitor exit`两个JVM指令，他能确保在任何时候，任何线程执行到`monitor enter`成功之前都必须从主内存中获取数据，而不是从缓存中，在`monitor exit`运行成功之后，共享变量被更新后的值必须刷入主内存内

* `synchronized` 严格准守`Java happends-before`规则，一个`monitor exit`指令之前必定要有一个`monitor enter`

## 用法

`synchronized` 可用于代码块或方法进行修饰，而不能对class以及变量进行修饰, eg:

```java
public synchronized void sync(){}

public synchronized static void sync(){}
```

```java
private final Object Mutex = new Object()

public void sync(){
    // Mutex 一定不能为null
    synchronized(Mutex){

    }
}
```

## 实际例子

```java
package com.thread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;


public class Main {

    private final static Object Mutex = new Object();
    public int num;
    static int n = 16;

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

    void printNum(){
        System.out.println("num:" + this.num);
    }

    public static void main(String[] args) throws Exception{
        Main main = new Main();
        main.num = 1;
        List<Thread> threadList = new ArrayList<>(n);
        for(int i=0;i<n;i++){
           Thread t = new Thread( ()-> main.add(), "myname" + i);
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

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java8/imgs/jconsole.png)

`synchronized`存在排它性，所有的线程必须串行的经过`synchronized`保护的共享区域；`synchronized`作用域越大，则代表着其效率越低，甚至还会丧失并发的优势

注意到 Mutex 是如下定义的，`static`的, 如果非static则根本不是一个共享区域

```java
private final static Object Mutex = new Object();
```

## 多个锁的交叉导致的死锁

```java
package com.thread;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;


public class Main {

    private final static Object ReadMutex = new Object();
    private final static Object WriteMutex = new Object();
    public int num;

    int read(){
        synchronized (ReadMutex){
            synchronized (WriteMutex){
                return this.num;
            }
        }
    }

    void write(){
        synchronized (WriteMutex){
            synchronized (ReadMutex){
                this.num ++;
            }
        }
    }

    void printNum(){
        System.out.println("num:" + this.num);
    }

    public static void main(String[] args) throws Exception{
        Main main = new Main();
        main.num = 0;
        int n = 15;
        List<Thread> threadList = new ArrayList<>(n);
        int j = 0;
        int k = 0;
        for(int i=0;i<n;i++){
            if(i % 2 == 0){
                Thread t = new Thread( ()-> main.read(), "read-" + j++);
                threadList.add(t);
            }else {
                Thread t = new Thread( ()-> main.write(), "write-" + k++);
                threadList.add(t);
            }
        }
        threadList.forEach(t->t.start());
        threadList.forEach(t->{
            try{
                System.out.println(t.getName());
                t.join();
            }catch (Exception e){
                e.printStackTrace();
            }
        });
        main.printNum();
    }
}
```

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java8/imgs/death_1.png)

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java8/imgs/death_2.png)

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java8/imgs/death_3.png)

## this monotor

```java
package com.thread;

import java.util.concurrent.TimeUnit;

public class Main {

    public synchronized void method1(){
        System.out.println(Thread.currentThread().getName() + " enter to method 1  ");
        while (true) {

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public synchronized void method2(){
        System.out.println(Thread.currentThread().getName() + " enter to method 2  ");
        while (true) {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws Exception{
        Main main = new Main();
        new Thread(main::method1, "T1").start();
        new Thread(main::method2, "T2").start();
    }
}
```

* output如下，然后程序暂停，jstack查看如下

```js
T1 enter to method 1  

```

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java8/imgs/
monitor_jstack.png)

两个`synchronized`方法挣抢的是同一个`monitor`的`lock`,而与之关联的引用则是`ThisMonitor`的实例引用

方法2等同如下：

```java
public void method2(){
    synchronized (this){
        System.out.println(Thread.currentThread().getName() + " enter to method 2  ");
        while (true) {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## class monitor

```java
package com.thread;

import java.util.concurrent.TimeUnit;

public class Main {

    public synchronized static void method1(){
        System.out.println(Thread.currentThread().getName() + " enter to method 1  ");
        while (true) {

            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public synchronized static void method2(){
        System.out.println(Thread.currentThread().getName() + " enter to method 2  ");
        while (true) {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws Exception{
        new Thread(Main::method1, "T1").start();
        new Thread(Main::method2, "T2").start();
    }
}
```

* output如下，然后程序暂停

```js
T1 enter to method 1  

```

## synchronized 的缺陷

* 无法控制阻塞时长
* 阻塞不可被中断

```java
public class Main {

    public synchronized void syncMethod() {
        try {
            TimeUnit.HOURS.sleep(1);
        } catch (InterruptedException e) {
            e. printStackTrace();
        }
    }

    public static void main(String[] args) throws Exception{
        Main defect=new Main();

        Thread thread1 = new Thread(defect:: syncMethod,"T1");
        // thread1 将先于 thread2 执行
        thread1. start();
        TimeUnit.MILLISECONDS.sleep(2);

        Thread thread2 = new Thread(defect::syncMethod,"T2");
        thread2.start();
    }
}
```

1. thread1 先执行进入同步方法，然后sleep
2. 接着 thread2 进入同步方法，会阻塞，其获得执行权取决于 thread1 何时释放 monitor (如果thread2计划最多1分钟获得执行权，否则就放弃，使用 synchronized 无法做到)
3. thread2 竞争 monitor 而陷入阻塞状态，那么 thread2 会无法中断（因为 synchronized 无法被打断）

```java
public class Main {

    public synchronized void syncMethod() {
        try {
            TimeUnit.HOURS.sleep(1);
        } catch (InterruptedException e) {
            e. printStackTrace();
        }
    }

    public static void main(String[] args) throws Exception{
        Main defect=new Main();

        Thread thread1 = new Thread(defect:: syncMethod,"T1");
        // thread1 将先于 thread2 执行
        thread1. start();
        TimeUnit.MILLISECONDS.sleep(2);

        Thread thread2 = new Thread(defect::syncMethod,"T2");
        thread2.start();

        TimeUnit.MILLISECONDS.sleep(2);
        thread2.interrupt();
        // true
        System.out.println(thread2.isInterrupted());
        // BLOCKED
        System.out.println(thread2.getState());
        // TIMED_WAITING
        System.out.println(thread1.getState());

    }
}
```

synchronized是基于JVM层面实现的，如果一个代码块被synchronized修饰了，当一个线程获取了对应的锁，并执行该代码块时，其他线程便只能一直等待，等待获取锁的线程释放锁，而这里获取锁的线程释放锁会有三种情况：

1. 获取锁的线程执行完了该代码块，然后线程释放对锁的占有；
2. 线程执行发生异常，此时JVM会让线程自动释放锁。
3. wait()方法释放锁，方便其它的线程使用锁。而且被唤醒时，就在此处唤醒
