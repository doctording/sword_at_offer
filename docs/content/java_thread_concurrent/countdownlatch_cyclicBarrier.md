---
title: "CountDownLatch & CyclicBarrier"
layout: page
date: 2020-03-19 00:00
---

[TOC]

# CountDownLatch (做减法，例子：六国逐一灭，秦统一)

A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.

```java
public class Main {

    public static void main(String[] args) throws Exception {
        CountDownLatch countDownLatch = new CountDownLatch(6);

        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "国，被灭");
                countDownLatch.countDown(); // count 减一
            }, "" + i).start();
        }
        // 一直等待，直到数字达到0，放行。
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + "\t **************秦帝国，统一华夏");

        closeDoor();
    }

    private static void closeDoor() throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <= 6; i++) {
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t上完自习，离开教室");
                countDownLatch.countDown();
            }, "thread"+String.valueOf(i)).start();
        }
        // 等待，减少数字达到0，放行。
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName()+"\t **************班长最后关门走人");
    }
}
```

## CountDownLatch源码

自定义`Sync`类继承`AbstractQueuedSynchronizer`，利用AQS自身的状态变量`state`代表数量(初始化的时候指定),`countdown`操作让状态只减1(具体是CAS操作`compareAndSetState`方法让`state`减少1)

```java
public class CountDownLatch {
    /**
     * Synchronization control For CountDownLatch.
     * Uses AQS state to represent count.
     */
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }

    private final Sync sync;

    public void countDown() {
        sync.releaseShared(1);
    }
```

* `unsafe.compareAndSwapInt`方法

```java
/**
    * Atomically sets synchronization state to the given updated
    * value if the current state value equals the expected value.
    * This operation has memory semantics of a {@code volatile} read
    * and write.
    *
    * @param expect the expected value
    * @param update the new value
    * @return {@code true} if successful. False return indicates that the actual
    *         value was not equal to the expected value.
    */
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

* await方法

```java
/**

当前线程一直等待直到countDown到0，或者线程有interrupted异常抛出；

如果count本来就是0，那么该方法直接返回


* Causes the current thread to wait until the latch has counted down to
* zero, unless the thread is {@linkplain Thread#interrupt interrupted}.
*
* <p>If the current count is zero then this method returns immediately.
*
* <p>If the current count is greater than zero then the current
* thread becomes disabled for thread scheduling purposes and lies
* dormant until one of two things happen:
* <ul>
* <li>The count reaches zero due to invocations of the
* {@link #countDown} method; or
* <li>Some other thread {@linkplain Thread#interrupt interrupts}
* the current thread.
* </ul>
*
* <p>If the current thread:
* <ul>
* <li>has its interrupted status set on entry to this method; or
* <li>is {@linkplain Thread#interrupt interrupted} while waiting,
* </ul>
* then {@link InterruptedException} is thrown and the current thread's
* interrupted status is cleared.
*
* @throws InterruptedException if the current thread is interrupted
*         while waiting
*/
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}

public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}
```

* Aqs的`doAcquireSharedInterruptibly`方法

```java
/**
* Acquires in shared interruptible mode.
* @param arg the acquire argument
*/
private void doAcquireSharedInterruptibly(int arg)
    throws InterruptedException {
    // 添加共享模式节点，
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        for (;;) {
            // 拿到当前节点的前驱节点
            final Node p = node.predecessor();
            if (p == head) {
                // 尝试获取资源
                int r = tryAcquireShared(arg);
                // 大于等于0，说明有资源获取
                if (r >= 0) {
                    // 把当前节点设置成head节点，并传播唤醒后面的节点。
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
            }
            // 这里和独占模式一样，如果没资源申请，封装节点，并park等待
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

# CyclicBarrier（做加法，例子：召唤神龙，要收集到七龙珠）

A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point. CyclicBarriers are useful in programs involving a fixed sized party of threads that must occasionally wait for each other. The barrier is called <em>cyclic</em> because it can be re-used after the waiting threads are released.

* cyclic: adj. 循环的; 周期的;
* barrier: n. 屏障; 障碍物; 障碍; 阻力; 关卡; 分界线; 隔阂;

```java
public class Main {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println("****召唤神龙****");
        });
        for (int i = 1; i <= 7; i++) {
            final int tempInt = i;
            new Thread(()->{
                System.out.println(Thread.currentThread().getName()+"\t收集到第七颗龙珠："+tempInt+"龙珠");
                try {
                    cyclicBarrier.await(); // 找到了，就继续等其它龙珠收集
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, String.valueOf(i)).start();
        }
    }

}
```

## Semaphore

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class Main {

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3); // 模拟3个停车位

        for(int i=1;i<=6;i++){
            new Thread(()->{
                try{
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + " 抢到车位");
                    TimeUnit.SECONDS.sleep(3);
                    System.out.println(Thread.currentThread().getName() + " 停车3s后离开");
                }catch (Exception e){
                }finally {
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```
