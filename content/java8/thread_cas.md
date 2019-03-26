---
title: "CAS(Conmpare And Swap) & AtomicInteger"
layout: page
date: 2019-03-24 00:00
---

[TOC]

# CAS

用于实现`多线程同步`的`原子指令`，非阻塞算法，是由CPU硬件实现

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java8/imgs/cas.png)

## CAS 缺点

https://blog.csdn.net/hxpjava1/article/details/79408692

# AtomicInteger 使用 CAS

## volatile 非原子性

```java
package com.mb;


import java.util.Random;
import java.util.concurrent.TimeUnit;

public class Main {

    public volatile static int num = 0;

    public static void add() {
        num++;
    }

    public synchronized static void addSync() {
        num++;
    }

    private final static int N = 30;

    public static void main(String[] args) throws Exception {
        Thread[] threads = new Thread[N];
        for(int i=0;i<N;i++){
            threads[i] = new Thread(()->{
                try{
                    TimeUnit.MILLISECONDS.sleep(new Random().nextInt(10));
                    int addCnt = 100;
                    for(int j=0;j<addCnt;j++){
                       add();
                    }
                }catch (Exception e){
                    e.printStackTrace();
                }
            });
            threads[i].start();
        }
        for(int i=0;i<N;i++) {
            threads[i].join();
        }
        System.out.println("num:" + num);
    }

}
```

## AtomicInteger


```java
package com.mb;


import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Random;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class Main {

    static SimpleDateFormat ft = new SimpleDateFormat ("yyyy.MM.dd HH:mm:ss SSS");

    public  static int num = 0;
    public  static AtomicInteger atomicInteger = new AtomicInteger(0);
    public static void atomicAdd() {
        atomicInteger.incrementAndGet();
    }

    public static void add() {
        num++;
    }

    public synchronized static void addSync() {
        num++;
    }

    private final static int N = 30;

    public static void main(String[] args) throws Exception {
        Thread[] threads = new Thread[N];

        System.out.println(ft.format(new Date()));
        for(int i=0;i<N;i++){
            threads[i] = new Thread(()->{
                try{
                    TimeUnit.MILLISECONDS.sleep(new Random().nextInt(10));
                    int addCnt = 100;
                    for(int j=0;j<addCnt;j++){
//                       addSync();
                        atomicAdd();
                    }
                }catch (Exception e){
                    e.printStackTrace();
                }
            });
            threads[i].start();
        }
        for(int i=0;i<N;i++) {
            threads[i].join();
        }
        System.out.println(ft.format(new Date()));
//        System.out.println("num:" + num);
        System.out.println("num:" + atomicInteger);
    }

}
```

## `incrementAndGet()`

```java
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```
