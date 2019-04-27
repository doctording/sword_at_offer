---
title: "Thread Lock"
layout: page
date: 2019-03-15 00:00
---

[TOC]

# Lock

## Lock基本用法例子

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

* 什么是可重入？

同一个线程可以反复获取锁多次，然后需要释放多次

### 对比`synchronized`, `ReentrantLock`的一些高级功能

* 等待可中断

持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于Synchronized来说可以避免出现死锁的情况。

* 公平锁

`ReentrantLock`默认的构造函数是创建的`非公平锁`，可以通过参数`true`设为`公平锁`，但公平锁表现的性能不是很好。

`synchronized`是非公平锁（因为`synchronized`是不公平竞争，后来的线程可能先得到锁，进而可能导致先到的线程持续饥饿，非公平竞争在很大程度上提升了`synchronized`吞吐率），可以重入

* 锁绑定多个条件

一个`ReentrantLock`对象可以同时绑定对个对象。

### 锁申请等待限时实际例子

申请锁，一定时间内获取不到，选择放弃

```java
package com.thread;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author mubi
 * @Date 2019/3/15 7:24 AM
 */
public class Main {

    public static void main(String[] args) throws Exception{
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    lock.lock();
                    System.out.println("A start");
                    TimeUnit.SECONDS.sleep(3);
                } catch (Exception e) {
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }
            }
        });

        Thread threadB = new Thread(()-> {
            try {
                if(lock.tryLock(1, TimeUnit.SECONDS)) {
                    System.out.println("B get lock start");
                }else{
                    System.err.println(Thread.currentThread().getName() + " tryLock error");
                }
            } catch (Exception e) {
                System.out.println("B Exception");
                e.printStackTrace();
            }finally {
                if ( ((ReentrantLock) lock).isHeldByCurrentThread()) {
                    lock.unlock();
                }
            }
        });

        threadA.start();
        // 让A 先于B 运行
        TimeUnit.SECONDS.sleep(1);
        threadB.start();

        threadA.join();
        threadB.join();
        System.out.println("main end");
    }
}
```

### 公平锁

```java
 /**
    * Sync object for fair locks
    */
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
        * Fair version of tryAcquire.  Don't grant access unless
        * recursive call or no waiters or is first.
        */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

### 公平锁

* 公平锁每次获取到锁为同步队列中的第一个节点，保证请求资源时间上的绝对顺序，而非公平锁有可能刚释放锁的线程下次继续获取该锁，则有可能导致其他线程永远无法获取到锁，造成`饥饿`现象。

* 公平锁为了保证时间上的绝对顺序，需要频繁的上下文切换，而非公平锁会降低了一定的上下文切换，降低性能开销。

## 读写锁

读写锁实际是一种特殊的自旋锁，它把对共享资源的访问者划分成读者和写者，读者只对共享资源进行读访问，写者则需要对共享资源进行写操作。

一次只有一个线程可以占有`写模式`的读写锁, 但是可以有多个线程同时占有`读模式`的读写锁.

### 伪代码

```cpp
count_mutex = mutex_init();
write_mutex = mutex_init();
read_count = 0;

void read_lock{
    lock(count_mutex);
    read_count++;
    if (read_count == 1) { // 第一个读者开始读时获得写锁
        lock(write_mutex);
    }
    unlock(count_mutex);
}

void read_unlock{
    lock(count_mutex);
    read_count--;
    if (read_count == 0) { //  最后一个读者离开时释放写锁
        unlock(write_mutex);
    }
    unlock(count_mutex);
}

void write_lock{
    lock(write_mutex);
}

void write_unlock{
    unlock(write_mutex);
}
```

参考：https://www.cnblogs.com/xiehongfeng100/p/4782135.html

### 实际例子

```java
package com.thread;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Random;
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * @Author mubi
 * @Date 2019/3/17 7:56 PM
 */
public class Main {

    private static ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
    Lock readLock = rwLock.readLock();
    Lock writeLock = rwLock.writeLock();

    private double data = 0;

    public void read(){
        readLock.lock();
        try {
            TimeUnit.SECONDS.sleep(1);
            System.out.println(Thread.currentThread().getName() + "读数据：" + data
                    + " " +  new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            readLock.unlock();
        }
    }

    public void write(){
        writeLock.lock();
        try {
            TimeUnit.SECONDS.sleep((long) (Math.random() * 10));
            this.data = new Random().nextDouble();
            System.out.println(Thread.currentThread().getName() + " ........写入数据: " + data
                    + " " +  new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            writeLock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Main main = new Main();
        ThreadPoolExecutor pool = new ThreadPoolExecutor(5,
                10,
                200, TimeUnit.MILLISECONDS,
                new ArrayBlockingQueue<Runnable>(5),
                new ThreadPoolExecutor.DiscardOldestPolicy());
        for(int i=0;i<10;i++){
            pool.submit(()-> main.read());
            pool.submit(()-> main.write());
        }

        pool.shutdown();
    }

}
```

* output

**某个时刻可以有多个读，但只能有一个写，写的时候不能读，读的时候不能写**

```java
pool-1-thread-1读数据：0.0 2019-03-17 19:53:43
pool-1-thread-3读数据：0.0 2019-03-17 19:53:43
pool-1-thread-1 ........写入数据: 0.14383955312719565 2019-03-17 19:53:48
pool-1-thread-2 ........写入数据: 0.36604507657651075 2019-03-17 19:53:57
pool-1-thread-4 ........写入数据: 0.24110141917904238 2019-03-17 19:54:03
pool-1-thread-4 ........写入数据: 0.6412197875946629 2019-03-17 19:54:08
pool-1-thread-6读数据：0.6412197875946629 2019-03-17 19:54:09
pool-1-thread-5读数据：0.6412197875946629 2019-03-17 19:54:09
pool-1-thread-7 ........写入数据: 0.3115143131633106 2019-03-17 19:54:12
pool-1-thread-8读数据：0.3115143131633106 2019-03-17 19:54:13
pool-1-thread-9 ........写入数据: 0.08244939010952534 2019-03-17 19:54:21
pool-1-thread-3读数据：0.08244939010952534 2019-03-17 19:54:22
pool-1-thread-10读数据：0.08244939010952534 2019-03-17 19:54:22
pool-1-thread-1 ........写入数据: 0.7060105583988802 2019-03-17 19:54:31
pool-1-thread-2读数据：0.7060105583988802 2019-03-17 19:54:32
```

## 分布式锁

很多时候我们需要保证一个方法在同一时间内只能被同一个线程执行。在单机环境中，通过 Java 提供的并发 API 我们可以解决，分布式环境下

1. 分布式与单机情况下最大的不同在于其不是多线程而是多进程
2. `多线程`由于可以`共享堆内存`，因此可以简单的采取内存作为标记存储位置。而进程之间甚至可能都不在同一台物理机上，因此需要将`标记`存储在一个`所有进程`都能看到的地方

<a target='_blank' href='https://www.cnblogs.com/seesun2012/p/9214653.html'>参考</a>

### 什么是分布式锁？

在分布式的部署环境下，通过锁机制来让多客户端互斥的对共享资源进行访问

* 排他性：在同一时间只会有一个客户端能获取到锁，其它客户端无法同时获取

* 避免死锁：这把锁在一段有限的时间之后，一定会被释放（正常释放或异常释放）

* 高可用：获取或释放锁的机制必须高可用且性能佳

### 分布式锁的实现方式

1. 基于数据库实现

2. 基于缓存

3. 基于ZooKeeper实现
