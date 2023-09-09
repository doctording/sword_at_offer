---
title: "AQS"
layout: page
date: 2020-20-15 00:00
---

[TOC]

# AbstractQueuedSynchronizer

* 锁的实现框架

参考：<a href="https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html" target="_blank">美团技术文章：从ReentrantLock的实现看AQS的原理及应用</a>

参考：<a href="https://www.cnblogs.com/dennyzhangdd/p/7218510.html" target="_blank">《The java.util.concurrent Synchronizer Framework》 JUC同步器框架（AQS框架）原文翻译</a>

<a href="http://gee.cs.oswego.edu/dl/papers/aqs.pdf">论文地址</a>

<a href="https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/locks/AbstractQueuedSynchronizer.html">docs api</a>

Provides a framework for implementing blocking locks and related synchronizers (semaphores, events, etc) that rely on first-in-first-out (FIFO) wait queues. This class is designed to be a useful basis for most kinds of synchronizers that rely on a single atomic int value to represent state. Subclasses must define the protected methods that change this state, and which define what that state means in terms of this object being acquired or released. Given these, the other methods in this class carry out all queuing and blocking mechanics. Subclasses can maintain other state fields, but only the atomically updated int value manipulated using methods getState(), setState(int) and compareAndSetState(int, int) is tracked with respect to synchronization.

## Aqs核心思想归纳

AQS核心思想是：<font color='red'>如果被请求的共享资源空闲，那么就将当前请求资源的线程设置为有效的工作线程，将共享资源设置为锁定状态；如果共享资源被占用，就需要一定的阻塞等待唤醒机制来保证锁分配。这个机制主要用的是CLH队列的变体实现的，将暂时获取不到锁的线程加入到队列中</font>

CLH：Craig、Landin and Hagersten队列，链表结构，AQS中的队列是CLH变体的虚拟双向队列（FIFO），AQS是通过将每条请求共享资源的线程封装成一个节点来实现锁的分配。

![](../../content/java_thread_concurrent/imgs/aqs-22.png)

![](../../content/java_thread_concurrent/imgs/aqs-2.png)

AQS使用一个`volatile`的int类型的成员变量state来表示同步状态，通过内置的FIFO队列来完成资源获取的排队工作，通过CAS完成对state值的修改。

## 基于aqs实现锁的例子代码

```java

class LeeLock  {

    private static class Sync extends AbstractQueuedSynchronizer {
        @Override
        protected boolean tryAcquire (int arg) {
            return compareAndSetState(0, 1);
        }

        @Override
        protected boolean tryRelease (int arg) {
            setState(0);
            return true;
        }

        @Override
        protected boolean isHeldExclusively () {
            return getState() == 1;
        }
    }

    private Sync sync = new Sync();

    public void lock () {
        sync.acquire(1);
    }

    public void unlock () {
        sync.release(1);
    }
}

public class MainTest {

    static int count = 0;
    static LeeLock leeLock = new LeeLock();

    public static void main (String[] args) throws InterruptedException {

        Runnable runnable = new Runnable() {
            @Override
            public void run () {
                try {
                    leeLock.lock();
                    for (int i = 0; i < 10000; i++) {
                        count++;
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    leeLock.unlock();
                }

            }
        };
        Thread thread1 = new Thread(runnable);
        Thread thread2 = new Thread(runnable);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(count); // 20000
    }
}
```

## 同步器

两个操作

1. `acquire`操作：阻塞调用的线程，直到或除非同步状态允许其继续执行。
2. `release`操作：则是通过某种方式改变同步状态，使得一或多个被acquire阻塞的线程继续执行。

同步器需要支持如下：

* 阻塞和非阻塞（例如tryLock）的同步
* 可选的超时设置，让调用者可以放弃等待
* 通过中断实现的任务取消，通常是分为两个版本，一个acquire可取消，而另一个不可以

同步器的实现根据其状态是否**独占**而有所不同。独占状态的同步器，在同一时间只有一个线程可以通过阻塞点，而**共享**状态的同步器可以同时有多个线程在执行。一般锁的实现类往往只维护独占状态，但是，例如计数信号量在数量许可的情况下，允许多个线程同时执行。为了使框架能得到广泛应用，这两种模式都要支持。

j.u.c包里还定义了Condition接口，用于支持监控形式的await/signal操作，这些操作与独占模式的Lock类有关，且Condition的实现天生就和与其关联的Lock类紧密相关

## AbstractQueuedSynchronizer 相关概念

AbstractQueuedSynchronizer 抽象类的注释说明

1. AbstractQueuedSynchronizer 提供了一个框架，用来实现blocking locks 和 一些同步器，且是基于一个FIFO队列的
2. AbstractQueuedSynchronizer 被设计为使用一个`single atomic {@code int} value`来表示状态
3. AbstractQueuedSynchronizer的子类必须去定义状态，并提供protected方法去操作状态：getState、setState以及compareAndSet
4. 基于AQS的具体实现类必须根据暴露出的状态相关的方法定义tryAcquire和tryRelease方法，以控制acquire和release操作。当同步状态满足时，tryAcquire方法必须返回true，而当新的同步状态允许后续acquire时，tryRelease方法也必须返回true。这些方法都接受一个int类型的参数用于传递想要的状态

### 阻塞

j.u.c包有一个LockSuport类，这个类中包含了解决这个问题的方法。方法`LockSupport.park`阻塞当前线程除非/直到有个`LockSupport.unpark`方法被调用（unpark方法被提前调用也是可以的）。unpark的调用是没有被计数的，因此在一个park调用前多次调用unpark方法只会解除一个park操作。另外，它们作用于每个线程而不是每个同步器。一个线程在一个新的同步器上调用park操作可能会立即返回，因为在此之前可能有“剩余的”unpark操作。但是，在缺少一个unpark操作时，下一次调用park就会阻塞。虽然可以显式地消除这个状态但并不值得这样做。在需要的时候多次调用park会更高效。

**park**: n. 公园; 专用区; 园区; (英国) 庄园，庭院; v. 停(车); 泊(车); 坐下(或站着); 把…搁置，推迟(在以后的会议上讨论或处理);

### 同步操作

* 独占模式

![](../../content/java_thread_concurrent/imgs/aqs.png)

* 共享模式

![](../../content/java_thread_concurrent/imgs/aqs-3.png)

* AbstractQueuedSynchronizer 的变量：有CLH队列的头部，尾部，以及同步器状态的int变量

```java
//用于标识共享锁
static final Node SHARED = new Node();

//用于标识独占锁
static final Node EXCLUSIVE = null;

/**
* 因为超时或者中断，节点会被设置为取消状态，被取消的节点时不会参与到竞争中的，他会一直保持取消状态不会转变为其他状态；
*/
static final int CANCELLED =  1;

/**
* 当前节点释放锁的时候，需要唤醒下一个节点
*/
static final int SIGNAL    = -1;

/**
* 节点在等待队列中，节点线程等待Condition唤醒
*/
static final int CONDITION = -2;

/**
* 表示下一次共享式同步状态获取将会无条件地传播下去
*/
static final int PROPAGATE = -3;

/** 等待状态 */
volatile int waitStatus;

/** 前驱节点 */
volatile Node prev;

/** 后继节点 */
volatile Node next;

/** 节点线程 */
volatile Thread thread;

//
Node nextWaiter;
```

### FIFO队列(CLH 队列，双向链表)

整个框架的关键就是如何管理被阻塞的线程的队列，该队列是严格的FIFO队列，因此，框架不支持基于优先级的同步。

自旋判断前驱节点是否释放了锁：如果前驱没有释放锁，那么就一直自旋；否则就能获取到锁，结束自旋

#### `acquireQueued`方法中会使线程自旋阻塞，直到获取到锁

```java
/**
    * Acquires in exclusive uninterruptible mode for thread already in
    * queue. Used by condition wait methods as well as acquire.
    *
    * @param node the nodxe
    * @param arg the acquire argument
    * @return {@code true} if interrupted while waiting
    */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

#### `addWaiter`方法会将当前线程封装成Node节点，CAS操作追加在队尾，并返回该节点

```java
/**
    * Creates and enqueues node for current thread and given mode.
    *
    * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
    * @return the new node
    */
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // 将该线程节点加入到队列的尾部,头尾指针变化
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

在`addWaiter`方法处理失败的时候进一步会调用`enq`方法

#### `enq`方法会将将node加入队尾，不断的进行CAS操作

```java
/**
    * Inserts node into queue, initializing if necessary. See picture above.
    * @param node the node to insert
    * @return node's predecessor
    */
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

通过`自旋`(CAS操作)来保证该节点能顺利的加入到队列尾部，只有加入成功才会退出循环，否则会一直自旋直到成功。

## 回顾ReentrantLock非公平锁的加锁过程

```java
// 非公平锁
static final class NonfairSync extends Sync {
	...
	final void lock() {
		if (compareAndSetState(0, 1))
			setExclusiveOwnerThread(Thread.currentThread());
		else
			acquire(1);
		}
  ...
}
```

1. 若通过CAS设置变量State（同步状态）成功，也就是获取锁成功，则将当前线程设置为独占线程。
2. 若通过CAS设置变量State（同步状态）失败，也就是获取锁失败，则进入Acquire方法进行后续处理。（Acquire方法：存在某种排队等候机制，让此线程继续等待，仍然保留此线程获取锁的可能，其获取锁流程仍在继续。这也是多线程并发处理的要求）

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

1. 尝试获取锁成功什么都不做
2. 尝试获取锁失败，则线程本身要加到等待队列中，并进入acquireQueued方法，对排队中的线程进行`获锁`操作(即自旋判断来尝试获取锁)

java.util.concurrent.locks.ReentrantLock.NonfairSync#tryAcquire实现如下

```java
protected final boolean tryAcquire(int acquires) {
    return nonfairTryAcquire(acquires);
}

/**
    * Performs non-fair tryLock.  tryAcquire is implemented in
    * subclasses, but both need nonfair try for trylock method.
    */
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) { // 1. 没有线程持有锁，则如下3句还是加锁语句（注意：acquires=1）
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) { // 2. 否则是持有锁的，如果是自己持有锁则可重入加锁
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false; // 3.否则是别的线程正在持有锁，此次尝试获取锁是失败的
}
```

附：

1. State初始化的时候为0，表示没有任何线程持有锁。
2. 当有线程持有该锁时，值就会在原来的基础上+1，同一个线程多次获得锁是，就会多次+1，这里就是可重入的概念。
3. 解锁也是对这个字段-1，一直到0，此线程对锁释放。

---

java.util.concurrent.locks.AbstractQueuedSynchronizer#acquireQueued

acquireQueued是aqs中的方法，而其中tryAcquire则是具体实现类的方法，上文可见`java.util.concurrent.locks.ReentrantLock.NonfairSync#tryAcquire`

```java
/**
    * Acquires in exclusive uninterruptible mode for thread already in
    * queue. Used by condition wait methods as well as acquire.
    *
    * @param node the node
    * @param arg the acquire argument
    * @return {@code true} if interrupted while waiting
    */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) { // 自旋
            final Node p = node.predecessor(); // 当前线程的前驱节点p是头结点，说明当前线程节点在真实数据队列的首部，就尝试获取锁（别忘了头结点是虚节点），即FIFO机制
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 当前线程作为头节点但是没有获取到锁（可能是非公平锁被抢占了）或者本身不为头结点，这个时候就要判断当前node是否要被阻塞（被阻塞条件：前驱节点的waitStatus为-1），防止无限循环浪费资源。
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### 释放锁过程

java.util.concurrent.locks.ReentrantLock#unlock

```java
public void unlock() {
    sync.release(1);
}
```

调用的是aqs的release方法

```java
/**
    * Releases in exclusive mode.  Implemented by unblocking one or
    * more threads if {@link #tryRelease} returns true.
    * This method can be used to implement method {@link Lock#unlock}.
    *
    * @param arg the release argument.  This value is conveyed to
    *        {@link #tryRelease} but is otherwise uninterpreted and
    *        can represent anything you like.
    * @return the value returned from {@link #tryRelease}
    */
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

回到：java.util.concurrent.locks.ReentrantLock.Sync#tryRelease，可重入锁的Sync的释放逻辑如下：

```java
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c);
    return free;
}
```

1. 当前线程不是持有锁的，抛除异常
2. 否则State-1，释放一次锁。如果完全释放锁了，那么锁的自由的了，当前锁没有占用了，进行相应设置
