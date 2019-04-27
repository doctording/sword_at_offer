---
title: "Java中的各种锁概念"
layout: page
date: 2019-03-15 00:00
---

[TOC]

# 常见的各种锁

## CAS 原子操作

既然用锁或`synchronized`关键字可以实现原子操作，那么为什么还要用`CAS`呢，因为加锁或使用`synchronized`关键字带来的性能损耗较大，而用`CAS`可以实现乐观锁，它实际上是直接利用了`CPU`层面的指令(由现代CPU在硬件级实现的原子指令，允许进行无阻塞，多线程的数据操作同时兼顾了安全性以及效率)，所以性能稍高。

`synchronized`关键字会让没有得到锁资源的线程进入`BLOCKED`状态，而后在争夺到锁资源后恢复为`RUNNABLE`状态，这个过程中涉及到操作系统用户模式和内核模式的转换，代价比较高。

* `synchronized`属于**悲观锁**，悲观地认为程序中的并发情况严重，所以严防死守
* `CAS`属于乐观锁，乐观地认为程序中的并发情况不那么严重，所以让线程不断去尝试更新

### CAS 优缺点

#### 性能 和 适用场景

CAS 适合简单对象的操作，比如布尔值、整型值等；
CAS 适合冲突较少的情况，如果太多线程在同时自旋，那么长时间循环会导致 CPU 开销很大；

1. CPU开销较大
在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复，会给CPU带来很大的压力。

2. 不能保证代码块的原子性

CAS机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。比如需要保证3个变量共同进行原子性的更新，就不得不使用Synchronized了。

作者：Flagle
链接：https://www.jianshu.com/p/ae25eb3cfb5d
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

#### ABA 问题

ABA问题的优化

ABA问题导致的原因，是CAS过程中只简单进行了“值”的校验，再有些情况下，“值”相同不会引入错误的业务逻辑（例如库存），有些情况下，“值”虽然相同，却已经不是原来的数据了。

CAS不能只比对**值**，还必须确保的是原来的数据，才能修改成功。增加`版本号`的比对，一个数据一个版本，版本变化，即使值相同，也不应该修改成功。

#### Java8 LongAdder

高并发中的分段处理机制，例如：`ConcurrengHashMap`

## 自旋锁

自旋锁是采用让当前线程不停地的在循环体内执行实现的，当循环的条件被其他线程改变时 才能进入临界区

```java

/**
 * 使用了CAS原子操作，lock函数将owner设置为当前线程，并且预测原来的值为空。unlock函数将owner设置为null，并且预测值为当前线程
 *
 * 该例子为非公平锁，获得锁的先后顺序，不会按照进入lock的先后顺序进行
 *
 */
class SpinLock {
    private AtomicReference<Thread> sign =new AtomicReference<>();
    public void lock(){
        Thread current = Thread.currentThread();
        while(!sign .compareAndSet(null, current)){
        }
    }

    public void unlock (){
        Thread current = Thread.currentThread();
        sign .compareAndSet(current, null);
    }

}

public class Solution {
    static int cnt = 0;
    static SpinLock spinLock = new SpinLock();

    static void add(){
        for(int i=0;i<1000;i++) {
            cnt++;
        }
    }

    static void addSync(){
        spinLock.lock();
        for(int i=0;i<1000;i++) {
            cnt++;
        }
        spinLock.unlock();
    }


    public static void main(String[] args){
        int n = 20;
        List<Thread> threadList = new ArrayList<>(n);
        for(int i=0;i<n;i++){
            Thread t = new Thread( ()-> Solution.add(), "add" + i);
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
        System.out.println(Solution.cnt);
    }

}
```

由于自旋锁只是将当前线程不停地执行循环体，不进行线程状态的改变，所以响应速度更快。但当线程数不停增加时，性能下降明显，因为每个线程都需要执行，占用CPU时间。如果线程竞争不激烈，并且保持锁的时间段，适合使用自旋锁。

## ReentrantLock (可重入锁)

* 什么是可重入？

同一个线程可以反复获取锁多次，然后需要释放多次

### 公平锁

表示线程获取锁的顺序是按照线程加锁的顺序来分配的，即先来先得的FIFO先进先出顺序。

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

* 公平锁每次获取到锁为同步队列中的第一个节点，保证请求资源时间上的绝对顺序，而非公平锁有可能刚释放锁的线程下次继续获取该锁，则有可能导致其他线程永远无法获取到锁，造成`饥饿`现象。

* 公平锁为了保证时间上的绝对顺序，需要频繁的上下文切换；而非公平锁会降低了一定的上下文切换，降低性能开销。

### 非公平锁

就是一种获取锁的抢占机制，是随机获得锁的，和公平锁不一样的就是先来的不一定先得到锁，这个方式可能造成某些线程一直拿不到锁，结果也就是不公平。

#### 对比`synchronized`, `ReentrantLock`的一些高级功能

* 等待可中断

持有锁的线程长期不释放的时候，正在等待的线程可以选择放弃等待，这相当于Synchronized来说可以避免出现死锁的情况。

* `ReentrantLock`创建公平锁

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

## 共享锁(读锁) 和 排它锁(写锁)

### Exclusive locks

Exclusive locks protect updates to file resources, both recoverable and non-recoverable. They can be owned by only one transaction at a time. Any transaction that requires an exclusive lock must wait if another task currently owns an exclusive lock or a shared lock against the requested resource.

独占锁保护文件资源(包括可恢复和不可恢复的文件资源)的更新。它们一次只能由一个事务拥有。如果另一个任务当前拥有对所请求资源的独占锁或共享锁，则任何需要对该资源申请独占锁的事务都必须等待。

### Shared locks

Shared locks support read integrity. They ensure that a record is not in the process of being updated during a read-only request. Shared locks can also be used to prevent updates of a record between the time that a record is read and the next syncpoint.

共享锁支持读完整性。它们确保在只读请求期间不会更新记录。共享锁还可用于防止在读取记录和下一个同步点之间进行更新操作。

A shared lock on a resource can be owned by several tasks at the same time. However, although several tasks can own shared locks, there are some circumstances in which tasks can be forced to wait for a lock:

资源上的共享锁可以同时由多个任务拥有。 但是，尽管有几个任务可以拥有共享锁，但在某些情况下可以强制任务等待锁：

* A request for a shared lock must wait if another task currently owns an exclusive lock on the resource.
* A request for an exclusive lock must wait if other tasks currently own shared locks on this resource.
* A new request for a shared lock must wait if another task is waiting for an exclusive lock on a resource that already has a shared lock.

* 如果另一个任务当前拥有资源上的独占锁，则对共享锁的请求必须等待。
* 如果其他任务当前拥有此资源上的共享锁，则必须等待对独占锁的请求。
* 如果另一个任务正在等待已经具有共享锁的资源的独占锁，则对共享锁的新请求必须等待。

## 偏向锁

锁的实现机制与Java对象头息息相关，锁的所有信息，都记录在Java的对象头中。用2字（32位JVM中1字=32bit=4baye）存储对象头，如果是数组类型使用3字存储（还需存储数组长度）。对象头中记录了hash值、GC年龄、锁的状态、线程拥有者、类元数据的指针。

大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，这个线程就是锁的偏向线程。

为了让线程获得锁的代价更低而引入偏向锁。那么只需要**在锁第一次被拥有的时候，记录下偏向线程ID。这样偏向线程就一直持有着锁，直到竞争发生才释放锁**。以后每次同步，检查锁的偏向线程ID与当前线程ID是否一致，如果一致直接进入同步/退出同步,也无需每次加锁解锁都去CAS更新对象头，如果不一致意味着发生了竞争，锁已经不是总是偏向于同一个线程了，这时候需要锁膨胀为轻量级锁，才能保证线程间公平竞争锁。

## 轻量级锁

轻量锁与偏向锁不同的是：

1. 轻量级锁每次退出同步块都需要释放锁，而偏向锁是在竞争发生时才释放锁
2. 每次进入退出同步块都需要CAS更新对象头
3. 争夺轻量级锁失败时，自旋尝试抢占锁

可以看到轻量锁适合在竞争情况下使用，其自旋锁可以保证响应速度快，但自旋操作会占用CPU，所以一些计算时间长的操作不适合使用轻量级锁。

当竞争线程尝试占用轻量级锁失败多次之后，轻量级锁就会膨胀为重量级锁，重量级线程指针指向竞争线程，竞争线程也会阻塞，等待轻量级线程释放锁后唤醒他。

## 重量级锁

当竞争线程尝试占用轻量级锁失败多次之后，轻量级锁就会膨胀为重量级锁，重量级线程指针指向竞争线程，竞争线程也会阻塞，等待轻量级线程释放锁后唤醒他。

重量级锁是依赖对象内部的monitor锁来实现的，而monitor又依赖操作系统的MutexLock(互斥锁)来实现的，所以重量级锁也被成为互斥锁。

### 为什么说重量级锁开销大呢

主要是，当系统检查到锁是重量级锁之后，会把等待想要获得锁的线程进行阻塞，被阻塞的线程不会消耗cup。但是阻塞或者唤醒一个线程时，都需要操作系统来帮忙，这就需要从用户态转换到内核态，而转换状态是需要消耗很多时间的，有可能比用户执行代码的时间还要长。这就是说为什么重量级线程开销很大的。
