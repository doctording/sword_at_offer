---
title: "Thread 线程池"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# 线程池

* 为什么需要线程池 ？ 线程池的应用范围 ？

* 线程池应该具备哪些功能 ？

* 线程池的实现需要注意哪些细节 ？

## 常见类

### interface Executor

```java
public interface Executor {

    /**
     * Executes the given command at some time in the future.  The command
     * may execute in a new thread, in a pooled thread, or in the calling
     * thread, at the discretion of the {@code Executor} implementation.
     *
     * @param command the runnable task
     * @throws RejectedExecutionException if this task cannot be
     * accepted for execution
     * @throws NullPointerException if command is null
     */
    void execute(Runnable command);
}
```

### interface Future<V>

```java
/**
 * A {@code Future} represents the result of an asynchronous
 * computation.  Methods are provided to check if the computation is
 * complete, to wait for its completion, and to retrieve the result of
 * the computation.  The result can only be retrieved using method
 * {@code get} when the computation has completed, blocking if
 * necessary until it is ready.  Cancellation is performed by the
 * {@code cancel} method.  Additional methods are provided to
 * determine if the task completed normally or was cancelled. Once a
 * computation has completed, the computation cannot be cancelled.
 * If you would like to use a {@code Future} for the sake
 * of cancellability but not provide a usable result, you can
 * declare types of the form {@code Future<?>} and
 * return {@code null} as a result of the underlying task.
 *
```

### interface ExecutorService extends Executor

* execute() 执行任务
* shutdown() 调用后不再接收新任务，如果里面有任务，就执行完
* shutdownNow() 调用后不再接受新任务，如果有等待任务，移出队列；有正在执行的，尝试停止之
* isShutdown() 判断线程池是否关闭
* isTerminated() 判断线程池中任务是否执行完成
* submit() 提交任务
* invokeAll() 执行一组任务

## abstract class AbstractExecutorService implements ExecutorService

## class ThreadPoolExecutor extends AbstractExecutorService

### ThreadFactory

* 工厂模式
* ThreadFactory

```java
/**
     * The default thread factory
     */
    static class DefaultThreadFactory implements ThreadFactory {
        private static final AtomicInteger poolNumber = new AtomicInteger(1);
        private final ThreadGroup group;
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        DefaultThreadFactory() {
            SecurityManager s = System.getSecurityManager();
            group = (s != null) ? s.getThreadGroup() :
                                  Thread.currentThread().getThreadGroup();
            namePrefix = "pool-" +
                          poolNumber.getAndIncrement() +
                         "-thread-";
        }

        public Thread newThread(Runnable r) {
            Thread t = new Thread(group, r,
                                  namePrefix + threadNumber.getAndIncrement(),
                                  0);
            if (t.isDaemon())
                t.setDaemon(false);
            if (t.getPriority() != Thread.NORM_PRIORITY)
                t.setPriority(Thread.NORM_PRIORITY);
            return t;
        }
    }
```

### ThreadPoolExecutor 构造函数和成员含义

```java
public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
            Executors.defaultThreadFactory(), defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
            threadFactory, defaultHandler);
}

public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            RejectedExecutionHandler handler) {
    this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
            Executors.defaultThreadFactory(), handler);
}

public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

* corePoolSize:指定了线程池中的线程数量，它的数量决定了添加的任务是开辟新的线程去执行，还是放到workQueue任务队列中去；

* maximumPoolSize:指定了线程池中的最大线程数量，这个参数会根据你使用的workQueue任务队列的类型，决定线程池会开辟的最大线程数量；

* keepAliveTime:当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁；

* unit:keepAliveTime的单位

* workQueue:阻塞任务队列，被添加到线程池中，但尚未被执行的任务；它一般分为直接提交队列、有界任务队列、无界任务队列、优先任务队列几种；

* threadFactory:线程工厂，用于创建线程，一般用默认即可；

* handler:拒绝策略；当任务太多来不及处理时，如何拒绝任务；

* workerCount 当前活跃的线程数(也即线程池中的线程数量)

## ThreadPoolExecutor 的使用

1. 当线程池**小于corePoolSize**时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程
2. 当线程池**达到corePoolSize**时，新提交任务将被放入workQueue中，等待线程池中任务调度执行
3. 当**workQueue已满**，且**maximumPoolSize>corePoolSize**时，新提交任务会创建新线程执行任务
4. 当**workQueue已满**，且workerCount**超过maximumPoolSize**时，新提交任务由RejectedExecutionHandler处理
5. 当线程池中超过corePoolSize线程，空闲时间达到keepAliveTime时，关闭空闲线程
6. 当设置allowCoreThreadTimeOut(true)时，线程池中corePoolSize线程空闲时间达到keepAliveTime也将关闭

### ThreadPoolExecutor 实例1

```java
package com.threadpool;

import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Author mubi
 * @Date 2019/4/8 10:52 PM
 */
public class ThreadPoolUse {

    static class MyThreadFactory implements ThreadFactory {

        private AtomicInteger count = new AtomicInteger(0);

        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            String threadName = "MyThread" + count.addAndGet(1);
            System.out.println(threadName);
            t.setName(threadName);
            return t;
        }
    }

    static class MyTask implements Runnable{
        int id;
        MyTask(int id){
            this.id = id;
        }

        @Override
        public void run() {
           try{
               System.out.println("myTask id:" + id);
               TimeUnit.SECONDS.sleep(10);
           }catch (Exception e){
               e.printStackTrace();
           }
        }
    }

    public static void main(String[] args){

        int corePoolSize = 2;
        int maximumPoolSize = 4;
        int keepAliveTime = 2;
        TimeUnit timeUnit = TimeUnit.SECONDS;
        BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(1);
        RejectedExecutionHandler handler = new ThreadPoolExecutor.AbortPolicy();
        ThreadFactory threadFactory = new MyThreadFactory();

        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                corePoolSize,
                maximumPoolSize,
                keepAliveTime,
                timeUnit,
                workQueue,
                threadFactory,
                handler);

        for(int i=0;i<6;i++) {
            MyTask task = new MyTask(i);
            executor.execute(task);
        }
        executor.shutdown();

    }

}
```
