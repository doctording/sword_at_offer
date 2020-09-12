---
title: "常见的多线程题目"
layout: page
date: 2019-03-12 00:00
---

[TOC]

# 高并发、任务执行时间短的业务怎样使用线程池？并发不高、任务执行时间长的业务怎样使用线程池？并发高、业务执行时间长的业务怎样使用线程池？

1. `高并发`、`任务执行时间短`的业务：任务执行时间短，即线程很快就执行完成了，可能会有频繁的线程切换，所以线程池线程数可以设置为CPU核数+1，以减少线程上下文切换

2. `并发不高`、`任务执行时间长`的业务
    * 假如业务时间长是集中在IO操作上，也就是`IO密集型`的任务，因为IO操作并不占用CPU，所以不要让所有的CPU闲下来，可以加大线程池中的线程数目，让CPU处理更多的业务
    * 假如是集中在计算操作上，也就是`计算密集型`任务，和（1）类似，线程池中的线程数设置得少一些，减少线程上下文切换

3. `并发高`、且`业务执行时间长`的业务：解决这种类型任务的关键不在于线程池而在于整体架构的设计，看看这些业务里面某些数据是否能`缓存`；增加服务器；需要分析业务执行时间长的问题，看看能不能使用中间件对任务进行拆分和解耦

# 三个线程：怎么能实现依次打印ABC的功能

## 方法1: ReentrantLock & Condition（条件锁）

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

    final static int N = 5;
    static int count = 0;
    static int flag = 1;

    public static void main(String[] args) throws Exception{

        ReentrantLock lock = new ReentrantLock();
        Condition conditionA = lock.newCondition();
        Condition conditionB = lock.newCondition();
        Condition conditionC = lock.newCondition();

        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                while(count < N) {
                    try {
                        lock.lock();
                        // 标志为1，A才能执行，否则阻塞自己
                        while (flag != 1){
                            conditionA.await();
                        }
                        System.out.print("A");
                        // 因为A打印完应该打印B， 这里线程B唤醒，同时设置标记
                        flag = 2;
                        conditionB.signal();
                    } catch (Exception e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            }
        });

        Thread threadB = new Thread(()-> {
            while(count < N) {
                try {
                    lock.lock();
                    while (flag != 2){
                        conditionB.await();
                    }
                    System.out.print("B");
                    flag = 3;
                    conditionC.signal();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        });

        Thread threadC = new Thread(()-> {
            while (count <= N) {
                try {
                    lock.lock();
                    while (flag != 3){
                        conditionC.await();
                    }
                    System.out.println("C" + count);
                    flag = 1;
                    count++;
                    conditionA.signal();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    lock.unlock();
                }
            }
        });

        threadA.start();
        threadB.start();
        threadC.start();

        threadA.join();
        threadB.join();
        threadC.join();
        System.out.println("main end");
    }
}
```

## 方法2: synchronized & wait() notify() notifyAll()（状态同步 + wait/notify）

```java

public class Main{

    static int N = 2;

    static int flag = 3;
    static Object object = new Object();

    public static void main(String[] args) throws Exception {

        Thread A = new Thread(()-> {
            synchronized (object) {
                for(int i=0;i<N;i++){
                    try {
                        //等待前一个的打印处理
                        while (flag != 3) {
                            object.wait();
                        }
                        System.out.print(Thread.currentThread().getName() + " ");
                        flag = 1;
                        // 唤醒其它线程
                        object.notifyAll();
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }, "A");

        Thread B = new Thread(()-> {
            synchronized (object) {
                for(int i=0;i<N;i++){
                    try {
                        //等待前一个的打印处理
                        while (flag != 1) {
                            object.wait();
                        }
                        System.out.print(Thread.currentThread().getName() + " ");
                        flag = 2;
                        // 唤醒其它线程
                        object.notifyAll();
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }, "B");

        Thread C = new Thread(()-> {
            synchronized (object) {
                for(int i=0;i<N;i++){
                    try {
                        //等待前一个的打印处理
                        while (flag != 2) {
                            object.wait();
                        }
                        System.out.print(Thread.currentThread().getName() + " ");
                        flag = 3;
                        // 唤醒其它线程
                        object.notifyAll();
                    }catch (Exception e){
                        e.printStackTrace();
                    }
                }
            }
        }, "C");

        A.start();
        B.start();
        C.start();

        A.join();
        B.join();
        C.join();
    }

}
```

## 方法3: 信号量

```java
class FooBar {
    private int n;
    Semaphore fooSemaphore = new Semaphore(1);
    Semaphore barSemaphore = new Semaphore(0);

    public FooBar(int n) {
        this.n = n;
    }

    public void foo(Runnable printFoo) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            fooSemaphore.acquire();
            // printFoo.run() outputs "foo". Do not change or remove this line.
            printFoo.run();
            barSemaphore.release();
        }
    }

    public void bar(Runnable printBar) throws InterruptedException {
        for (int i = 0; i < n; i++) {
            barSemaphore.acquire(); // 初始为0, barSemaphore.acquire 会阻塞等待
            // printBar.run() outputs "bar". Do not change or remove this line.
            printBar.run();
            fooSemaphore.release();
        }
    }
}
```

# 现在有T1、T2、T3三个线程，你怎样保证T2在T1执行完后执行，T3在T2执行完后执行？

```java
public class Main {

    public static void main(String[] args) throws Exception{
        Thread t1 = new Thread(()->{
            System.out.println(Thread.currentThread().getName() + " running");
        },"t1");

        Thread t2 = new Thread(()->{

            try{
                t1.join();
            }catch (Exception e){
            }

            System.out.println(Thread.currentThread().getName() + " running");
        },"t2");

        Thread t3 = new Thread(()->{

            try{
                t2.join();
            }catch (Exception e){
            }

            System.out.println(Thread.currentThread().getName() + " running");
        },"t3");

        t1.start();
        t2.start();
        t3.start();

        TimeUnit.SECONDS.sleep(1);

        System.out.println("main end");
    }

}
```

# 用Java实现阻塞队列？

阻塞队列与普通队列的不同在于。当队列是空的时候，从队列中获取元素的操作将会被阻塞，或者当队列满时，往队列里面添加元素将会被阻塞。需要保证多个线程可以安全的访问队列

1. ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列
2. LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列
3. PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列
4. DelayQueue：一个使用优先级队列实现的无界阻塞队列
5. SynchronousQueue：一个不存储元素的阻塞队列
6. LinkedTransferQueue：一个由链表结构组成的无界阻塞队列
7. LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列

# Future & guava

## future + 观察者模式

```java
public static void listenFutureTest() throws Exception{
    ExecutorService executor = Executors.newFixedThreadPool(2);
    ListeningExecutorService service = MoreExecutors.listeningDecorator(executor);
    ListenableFuture<Double> future = service.submit(new Callable<Double>() {
        @Override
        public Double call() throws Exception {
            delay(5);
            System.out.println("cal ok");
            return 10.0;
        }
    });

    Futures.addCallback(future, new FutureCallback<Double>() {
        @Override
        public void onSuccess(@Nullable Double aDouble) {
            System.out.println("onSuccess:" + aDouble);
        }

        @Override
        public void onFailure(Throwable throwable) {
            throwable.printStackTrace();
        }
    }, service);

//        service.shutdown();
}
```

## CompletableFuture的应用

场景：一个任务T有N个子任务，每个子任务完成时间不一样；若其中一个子任务失败，则该任务T结束，要求:fail fast。

```java
public class Main {

    /**
     * 模拟sec秒延迟
     */
    public static void delay(int sec) {
        try {
            TimeUnit.SECONDS.sleep(sec);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) throws Exception{
        testTask2();
    }

    static List<MyTask> taskList = new ArrayList<>();

    public static void testTask2() throws Exception{
        MyTask myTask1 = new MyTask("task1", 3, Result.SUCCESS);
        MyTask myTask2 = new MyTask("task2", 4, Result.SUCCESS);
        MyTask myTask3 = new MyTask("task3", 1, Result.FAIL);

        taskList.add(myTask1);
        taskList.add(myTask2);
        taskList.add(myTask3);

        for(MyTask task : taskList) {
            CompletableFuture f =
                    CompletableFuture.supplyAsync(() ->
                            task.runTask()).thenAccept(
                                    (result) -> callback(result, task));
        }

        System.in.read();
    }

    private static void callback(Result result, MyTask task){
        if(result == Result.FAIL){
            // 一个失败，其它所有任务按需要处理，可能回滚，取消，忽略等
            for(MyTask _task : taskList) {
                if(task != _task){
                    _task.cancel();
                }
            }
        }
    }

    private enum Result{
        SUCCESS("cancel"),
        FAIL("fail"),
        CANCELED("canceled");

        Result(String value) {
            this.value = value;
        }

        String value;

        public String getValue() {
            return value;
        }
    }

    private static class MyTask{
        private String name;
        private int timeOut;
        private Result ret;
        volatile boolean cancelled = false;

        public MyTask(String name, int timeOut, Result ret) {
            this.name = name;
            this.timeOut = timeOut;
            this.ret = ret;
        }

        public Result runTask(){
            int interval = 100;
            int total = 0;

            try{
                for (;;){
                    // 模拟CPU密集型
                    TimeUnit.MILLISECONDS.sleep(interval);
                    total += interval;
                    if(total > timeOut * 1000){
                        break;
                    }
                    if(cancelled){
                        return Result.CANCELED;
                    }
                }
            }catch (Exception e){
                e.printStackTrace();
                return Result.FAIL;
            }
            System.out.println(name + ":end");
            return ret;
        }

        public void cancel(){
            if(!cancelled) {
                synchronized (this) {
                    if (cancelled) {
                        return;
                    }
                    System.out.println(name + ": cancelling");
                    try {
                        delay(1);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    System.out.println(name + ": cancelled");
                }
                cancelled = true;
            }
        }
    }

}
```
