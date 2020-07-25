---
title: "Thread 死锁"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# 死锁

## 死锁的4个必要条件（操作系统）

1. 互斥：某种资源一次只允许一个进程访问，即该资源一旦分配给某个进程，其他进程就不能再访问，直到该进程访问结束。
2. 不可剥夺：进程所获得的资源在未使用完毕之前，不被其他进程强行剥夺，而只能由获得该资源的进程资源释放。
3. 请求和保持：进程每次申请它所需要的一部分资源，在申请新的资源的同时，继续占用已分配到的资源。
4. 循环等待：存在一个进程循环链，使得每个进程都占有下一个进程所需的至少一种资源。

## 程序死锁的可能原因

1. 交叉锁

2. 内存不足

例如线程T1获得了10M内存,线程T2获得了20M内存，每个线程都需要30M的运行内存，但是此时剩余可用的内存刚好为20M,那么两个线程有可能都在等待彼此能够释放的内存资源

3. 一问一答式的数据交换

服务端，客户端都在等待对方

4. 数据库锁

5. 文件锁

某个线程获得了文件锁意外退出，其它读取该文件的线程也将会进入死锁直到系统释放文件句柄资源

6. 死循环引起的死锁

系统假死

## 死锁代码例子

```java
class DeadLock {
    // 创建资源
    private static Object resourceA = new Object();
    private static Object resourceB = new Object();

    //测试
    public static void test() {
        //启动线程A,B
        new Thread(new A(), "threadA").start();
        new Thread(new B(), "threadB").start();
    }

    static class A implements Runnable{
        @Override
        public void run() {
            synchronized (resourceA) { // 持有资源A
                System.out.println(Thread.currentThread() + " get ResourceA");

                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    System.err.print(e);
                }

                System.out.println(Thread.currentThread() + " waiting get ResourceB");
                // 想获取资源B
                synchronized (resourceB) {
                    System.out.println(Thread.currentThread() + " get ResourceB");
                }
            }
        }
    }

    static class B implements Runnable{
        @Override
        public void run() {
            synchronized (resourceB) { // 持有资源B
                System.out.println(Thread.currentThread() + " get ResourceB");

                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    System.err.print(e);
                }

                System.out.println(Thread.currentThread() + " waiting get ResourceA");
                // 想获取资源A
                synchronized (resourceA) {
                    System.out.println(Thread.currentThread() + " get ResourceA");
                }
            }
        }
    }
}
```

### 信号量解决死锁

```java
class DeadLock {
    // 创建资源
    private static Object resourceA = new Object();
    private static Object resourceB = new Object();

    private static Semaphore semaphoreA = new Semaphore(1);
    private static Semaphore semaphoreB = new Semaphore(1);

    //测试
    public static void test() {
        //启动线程A,B
        new Thread(new A(), "threadA").start();
        new Thread(new B(), "threadB").start();
    }

    static class A implements Runnable {
        @Override
        public void run() {
            try {
                if(semaphoreA.tryAcquire(1, TimeUnit.SECONDS)) {
                    System.out.println(Thread.currentThread() + " get ResourceA");


                    TimeUnit.SECONDS.sleep(2);

                    System.out.println(Thread.currentThread() + " waiting get ResourceB");
                    // 想获取资源B
                    if(semaphoreB.tryAcquire(1, TimeUnit.SECONDS)) {
                        System.out.println(Thread.currentThread() + " get ResourceB");

                        System.out.println("A do sth");

                        semaphoreB.release();
                    }else {
                        System.out.println("A get ResourceB failed");
                    }

                    semaphoreA.release();
                }else{
                    System.out.println("A get ResourceA failed");
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }

    static class B implements Runnable{
        @Override
        public void run() {
            try {
                if(semaphoreB.tryAcquire(1, TimeUnit.SECONDS)) {
                    System.out.println(Thread.currentThread() + " get ResourceB");


                    TimeUnit.SECONDS.sleep(1);

                    System.out.println(Thread.currentThread() + " waiting get ResourceA");
                    // 想获取资源B
                    if(semaphoreA.tryAcquire(1, TimeUnit.SECONDS)) {
                        System.out.println(Thread.currentThread() + " get ResourceA");

                        System.out.println("B do sth");

                        semaphoreA.release();
                    }else{
                        System.out.println("B get ResourceA failed");
                    }
                    semaphoreB.release();
                }else{
                    System.out.println("B get ResourceB failed");
                }
            }catch (Exception e){
                e.printStackTrace();
            }
        }
    }
}
```
