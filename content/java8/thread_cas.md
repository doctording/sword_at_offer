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

## AtomicInteger 使用 CAS

### volatile 非原子性

```java

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

### AtomicInteger使用例子

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

### `incrementAndGet()`

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

参考

https://baijiahao.baidu.com/s?id=1618478640686359591&wfr=spider&for=pc

## unsafe 的一些方法

### unsafe.objectFieldOffset()

```java
private static Unsafe unsafe = null;
private static long valueOffset;

static {
    try{
        Class<?> clazz = Unsafe.class;
        Field f;
        f = clazz.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        unsafe = (Unsafe) f.get(clazz);
        valueOffset = unsafe.objectFieldOffset(Main.class.getDeclaredField("value"));
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }catch (SecurityException e) {
        e.printStackTrace();
    } catch (NoSuchFieldException e) {
        e.printStackTrace();
    }
}
```

JVM的实现可以自由选择如何实现Java对象的“布局”，也就是在内存里Java对象的各个部分放在哪里，包括对象的实例字段和一些元数据之类。`sun.misc.Unsafe`里关于对象字段访问的方法把对象布局抽象出来，它提供了`objectFieldOffset()`方法用于获取某个字段相对Java对象的“起始地址”的偏移量，也提供了getInt、getLong、getObject之类的方法可以使用前面获取的偏移量来访问某个Java对象的某个字段

* 参考：

作者：世界屋顶
来源：CSDN
原文：https://blog.csdn.net/blogs_broadcast/article/details/80672515 
版权声明：本文为博主原创文章，转载请附上博文链接！

### compareAndSwapInt

```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```

即:

```java
boolean compareAndSwapInt(Object obj,long fieldoffset, int expect, int update);
内存值V、旧的预期值A、要修改的值B
当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做并返回false。
```

即 当要修改obj对象的`（fieldoffset）Int`属性值与`expect`相同时,则修改`（fieldoffset）Int`为update，并返回`true`,否则什么都不做，返回`false`

### getIntVolatile

```java
public native int getIntVolatile(Object var1, long var2);
```

getIntVolatile方法用于在对象指定偏移地址处volatile读取一个int。

### unsage线程安全操作例子程序

```java
public class Main {

    private static Unsafe unsafe = null;
    private static long valueOffset;

    static {
        try{
            Class<?> clazz = Unsafe.class;
            Field f;
            f = clazz.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            unsafe = (Unsafe) f.get(clazz);
            valueOffset = unsafe.objectFieldOffset(Main.class.getDeclaredField("value"));
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }catch (SecurityException e) {
            e.printStackTrace();
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
    }

    private volatile int value;

    public final int get() {
        return value;
    }

    public final void set(int newValue) {
        value = newValue;
    }

    public void addAndIncrease() {
        int var5;
        do {
            var5 = unsafe.getIntVolatile(this, valueOffset);
        } while(!unsafe.compareAndSwapInt(this, valueOffset, var5, var5 + 1));
    }

    public void add() {
        value ++;
    }

    public synchronized void addSync() {
       value ++;
    }

    private final static int N = 30;

    public static void main(String[] args) throws Exception {
        Main m = new Main();
        m.set(2);
        Thread[] threads = new Thread[N];
        for(int i=0;i<N;i++){
            threads[i] = new Thread(()->{
                try{
                    TimeUnit.MILLISECONDS.sleep(new Random().nextInt(10));
                    int addCnt = 100;
                    for(int j=0;j<addCnt;j++){
//                        m.add();
//                        m.addSync();
                        m.addAndIncrease();
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
        System.out.println("value:" + m.get());
    }

}
```