---
title: "CountDownLatch & CyclicBarrier"
layout: page
date: 2020-03-19 00:00
---

[TOC]

# CountDownLatch (做减法，六国逐一灭，秦统一)

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
        // 等待，直到数字达到0，放行。
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

# CyclicBarriers（做加法，召唤神龙，要收集到七龙珠）

A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point. CyclicBarriers are useful in programs involving a fixed sized party of threads that must occasionally wait for each other. The barrier is called <em>cyclic</em> because it can be re-used after the waiting threads are released.

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