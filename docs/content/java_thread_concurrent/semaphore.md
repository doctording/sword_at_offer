---
title: "Semaphore"
layout: page
date: 2019-06-27 00:00
---

[TOC]

# Semaphore

信号量(Semaphore)，有时被称为信号灯，是在多线程环境下使用的一种设施，是可以用来保证两个或多个关键代码段不被并发调用。在进入一个关键代码段之前，线程必须获取一个信号量；一旦该关键代码段完成了，那么该线程必须释放信号量。其它想进入该关键代码段的线程必须等待直到第一个线程释放信号量。为了完成这个过程，需要创建一个信号量VI，然后将Acquire Semaphore VI以及Release Semaphore VI分别放置在每个关键代码段的首末端。确认这些信号量VI引用的是初始创建的信号量。（百度百科）

## Java Semaphore

### 常用API

名称 | 用法
-|-
acquire() | 获取一个信号量（许可）
acquire(int permits) | 获取 permits 数量的信号量，在获取到 permits 数量的信号量之前会一直阻塞等待，并且数量必须小于等于 Semaphore 允许的总信号量，否则会出现死锁
acquireUninterruptibly() | 获取一个不可被中断的信号量
tryAcquire() | 尝试去回去一个信号量，获取到了，返回 true，否则 false
tryAcquire(long timeout, TimeUnit unit) | 在限定时间内尝试去回去一个信号量，获取到了，返回 true，否则 false
release() | 释放当前的信号量
release(int permits) | 同样，上面获取了多少个信号量，这里就需要释放多少个，否则容易出现死锁一直等待的情况

### "公平信号量"和"非公平信号量"

Semaphore类采用AQS的共享模式，里面的两个内部类`FairSync`和`NonfairSync`都继承自`AbstractQueuedSynchronizer`

"公平信号量"和"非公平信号量"的释放信号量的机制是一样的！不同的是它们**获取信号量**的机制：线程在尝试获取信号量许可时，对于公平信号量而言，如果当前线程不在CLH队列(CLH即Craig, Landin, and Hagersten (CLH)。AQS内部维护着一个FIFO的队列，即CLH队列。AQS的同步机制就是依靠CLH队列实现的。CLH队列是FIFO的双端双向队列，实现公平锁。线程通过AQS获取锁失败，就会将线程封装成一个Node节点，插入队列尾。当有线程释放锁时，后尝试把队头的next节点占用锁。)的头部，则排队等候；而对于非公平信号量而言，无论当前线程是不是在CLH队列的头部，它都会直接获取信号量。该差异具体的体现在，它们的`tryAcquireShared()`函数的实现不同。

* 非公平获取的源码

```java
final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

<font color='red'>无限循环，获取状态，然后CAS操作保证state确实减少acquires</font>

### 构造函数(默认非公平)

```java
/**
    * Creates a {@code Semaphore} with the given number of
    * permits and nonfair fairness setting.
    *
    * @param permits the initial number of permits available.
    *        This value may be negative, in which case releases
    *        must occur before any acquires will be granted.
    */
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

/**
    * Creates a {@code Semaphore} with the given number of
    * permits and the given fairness setting.
    *
    * @param permits the initial number of permits available.
    *        This value may be negative, in which case releases
    *        must occur before any acquires will be granted.
    * @param fair {@code true} if this semaphore will guarantee
    *        first-in first-out granting of permits under contention,
    *        else {@code false}
    */
public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

### 例子代码

```java
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.concurrent.*;

/**
 * @Author mubi
 * @Date 2019/6/28 11:38 AM
 *
 * Semaphore 测试
 */
public class ContextTest {

    static SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss,SSS");

    // 限定进入线程的并发数量
    private static final Semaphore semaphore = new Semaphore(3);

    static class TestThread1 implements Runnable {
        @Override
        public void run() {
            try {
                semaphore.acquire(1);
                Calendar cal = Calendar.getInstance();
                System.out.println( df.format(cal.getTime()) + " :" + Thread.currentThread().getName());
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release(1);
            }
        }
    }

    static class TestThread2 implements Runnable {
        @Override
        public void run() {
            try {
                semaphore.acquire(2);
                Calendar cal = Calendar.getInstance();
                System.out.println( df.format(cal.getTime()) + " :" + Thread.currentThread().getName());
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                semaphore.release(2);
            }
        }
    }

    void test1(){
        ExecutorService exec = new ThreadPoolExecutor(10,
                Integer.MAX_VALUE,
                60L,
                TimeUnit.SECONDS,
                new SynchronousQueue<>());
        int n = 10;
        for(int i=0;i<n;i++) {
            Thread t = new Thread(new TestThread1(), "Thread_" + i);
            exec.execute(t);
        }
        exec.shutdown();
    }

    void test2(){
        ExecutorService exec = new ThreadPoolExecutor(10,
                Integer.MAX_VALUE,
                60L,
                TimeUnit.SECONDS,
                new SynchronousQueue<>());
        int n = 10;
        for(int i=0;i<n;i++) {
            Thread t = new Thread(new TestThread2(), "Thread_" + i);
            exec.execute(t);
        }
        exec.shutdown();
    }

    public static void main(String[] args) throws InterruptedException {
        ContextTest contextTest = new ContextTest();
        contextTest.test1();
        contextTest.test2();
    }

}
/* test1 output, 能同时并发3个

2019-06-28 12:00:52,195 :pool-1-thread-1
2019-06-28 12:00:52,199 :pool-1-thread-2
2019-06-28 12:00:52,199 :pool-1-thread-3
2019-06-28 12:00:55,200 :pool-1-thread-4
2019-06-28 12:00:55,204 :pool-1-thread-6
2019-06-28 12:00:55,204 :pool-1-thread-5
2019-06-28 12:00:58,204 :pool-1-thread-7
2019-06-28 12:00:58,210 :pool-1-thread-9
2019-06-28 12:00:58,210 :pool-1-thread-8
2019-06-28 12:01:01,209 :pool-1-thread-10

 */
/* test2 output，只能并发1个

 2019-06-28 11:59:37,038 :pool-1-thread-1
2019-06-28 11:59:40,043 :pool-1-thread-2
2019-06-28 11:59:43,048 :pool-1-thread-3
2019-06-28 11:59:46,050 :pool-1-thread-4
2019-06-28 11:59:49,056 :pool-1-thread-5
2019-06-28 11:59:52,060 :pool-1-thread-6
2019-06-28 11:59:55,064 :pool-1-thread-7
2019-06-28 11:59:58,067 :pool-1-thread-8
2019-06-28 12:00:01,069 :pool-1-thread-9
2019-06-28 12:00:04,073 :pool-1-thread-10

 */
```
