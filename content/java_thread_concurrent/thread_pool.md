---
title: "Thread 线程池"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# 线程池

* 为什么需要线程池 ？ 线程池的应用范围 ？

1. 线程的创建销毁是个消耗系统的操作
2. 线程资源的复用

* 线程池应该具备哪些功能 ？

* 线程池的实现需要注意哪些细节 ？

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java8/imgs/thread_pool_flow.png)

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

* `corePoolSize`: 指定了线程池中的线程数量，它的数量决定了添加的任务是开辟新的线程去执行，还是放到workQueue任务队列中去；

* `maximumPoolSize`: 指定了线程池中的最大线程数量，这个参数会根据你使用的workQueue任务队列的类型，决定线程池会开辟的最大线程数量；

* `keepAliveTime`: 当线程池中空闲线程数量超过corePoolSize时，多余的线程会在多长时间内被销毁；

* `unit`: keepAliveTime的单位

* `workQueue`: 阻塞任务队列，被添加到线程池中，但尚未被执行的任务；它一般分为:1.直接提交队列、2.有界任务队列、3.无界任务队列、4.优先任务队列(特殊的无界队列)几种；

* `threadFactory`: 线程工厂，用于创建线程，一般用默认即可；

* `handler`: 拒绝策略；当任务太多来不及处理时，如何拒绝任务；

* `workerCount`: 当前活跃的线程数(也即线程池中的线程数量)

## 拒绝策略

线程池的拒绝策略，是指当任务添加到线程池中被拒绝，而采取的处理措施。

当任务添加到线程池中之所以被拒绝，可能是由于：第一，线程池异常关闭。第二，任务数量超过线程池的最大限制,并设置有界的`workeQueue`

1. `ThreadPoolExecutor.AbortPolicy`:丢弃任务并抛出RejectedExecutionException异常。（默认）
2. `ThreadPoolExecutor.DiscardPolicy`：丢弃任务，但是不抛出异常。
3. `ThreadPoolExecutor.DiscardOldestPolicy`：丢弃队列最前面的任务，然后重新提交被拒绝的任务
4. `ThreadPoolExecutor.CallerRunsPolicy`：由调用线程（提交任务的线程）处理该任务

## ThreadPoolExecutor 的使用

1. 当线程池**小于corePoolSize**时，新提交任务将创建一个新线程执行任务，即使此时线程池中存在空闲线程
2. 当线程池**达到corePoolSize**时，新提交任务将被放入workQueue中，等待线程池中任务调度执行
3. 当**workQueue已满**，如果workerCount >= corePoolSize && workerCount < maximumPoolSize，则创建并启动一个线程来执行新提交的任务；
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

### 线程池使用后关闭问题？

是否需要关闭？如何关闭？

A pool that is no longer referenced in a program and has no remaining threads will be shutdown automatically.

如果程序中不再持有线程池的引用，并且线程池中没有线程时，线程池将会自动关闭。

注：<small>线程池中没有线程是指线程池中的所有线程都已运行完自动消亡。然而我们常用的FixedThreadPool的核心线程没有超时策略，所以并不会自动关闭。</small>

---

https://www.jianshu.com/p/bdf06e2c1541

#### 固定线程池

```java
static void testFixPool(){
    while(true) {
        ExecutorService executorService = Executors.newFixedThreadPool(8);
        executorService.execute(() -> System.out.println("running"));
        executorService = null;
    }
}

public static void main(String[] args){
    testFixPool();
    System.out.println("main end");
}
```

* 因为固定线程池不会自己销毁，最终会耗尽内存（需要在合适的时候`shutdown`）

```java
running
running
running
running
running
Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread
	at java.lang.Thread.start0(Native Method)
	at java.lang.Thread.start(Thread.java:717)
	at java.util.concurrent.ThreadPoolExecutor.addWorker(ThreadPoolExecutor.java:957)
	at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1367)
	at com.threadpool.ThreadPoolUse.testFixPool(ThreadPoolUse.java:71)
	at com.threadpool.ThreadPoolUse.main(ThreadPoolUse.java:77)
```

#### CachedThreadPool

```java
static void testCachedThreadPool(){
    while(true) {
        // 默认keepAliveTime为 60s
        ExecutorService executorService = Executors.newCachedThreadPool();
        ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executorService;
        // 为了更好的模拟，动态修改为1纳秒
        threadPoolExecutor.setKeepAliveTime(1, TimeUnit.NANOSECONDS);
        threadPoolExecutor.execute(() -> System.out.println("running"));
        executorService = null;
    }
}

public static void main(String[] args){
    testCachedThreadPool();
    System.out.println("main end");
}
```

* CachedThreadPool 的线程 keepAliveTime 默认为 60s ，核心线程数量为 0 ，所以不会有核心线程存活阻止线程池自动关闭。 详见 线程池之ThreadPoolExecutor构造 ，为了更快的模拟，构造后将 keepAliveTime 修改为1纳秒，相当于线程执行完马上会消亡，所以线程池可以被回收。实际开发中，如果CachedThreadPool 确实忘记关闭，在一定时间后是可以被回收的。但仍然建议显示关闭。

#### 线程池`shutdown`，`shutdownNow`

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

    static void testFixPool(){
        while(true) {
            ExecutorService executorService = Executors.newFixedThreadPool(8);
            executorService.execute(() -> System.out.println("running"));
            executorService = null;
        }
    }

    static void testCachedThreadPool(){
        while(true) {
            // 默认keepAliveTime为 60s
            ExecutorService executorService = Executors.newCachedThreadPool();
            ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executorService;
            // 为了更好的模拟，动态修改为1纳秒
            threadPoolExecutor.setKeepAliveTime(1, TimeUnit.NANOSECONDS);
            threadPoolExecutor.execute(() -> System.out.println("running"));
            executorService = null;
        }
    }

    static class MyRejectPolicy implements RejectedExecutionHandler{
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            if (r instanceof MyTask) {
                MyTask r1 = (MyTask) r;
                //直接打印
                System.out.println("Reject Thread:" + r1.id);
            }
        }
    }

    static void test(){
        int corePoolSize = 2;
        int maximumPoolSize = 5;
        int keepAliveTime = 60 * 1;
        TimeUnit timeUnit = TimeUnit.SECONDS;
        BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(2);
//        RejectedExecutionHandler handler = new ThreadPoolExecutor.AbortPolicy();
        RejectedExecutionHandler handler = new MyRejectPolicy();
        ThreadFactory threadFactory = new MyThreadFactory();

        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                corePoolSize,
                maximumPoolSize,
                keepAliveTime,
                timeUnit,
                workQueue,
                threadFactory,
                handler);

        MyTask task = new MyTask(1);
        executor.execute(task);
        executor.shutdown();
        // 已经关闭的线程池，引用还在，再有新任务，会执行拒绝策略
        MyTask task2 = new MyTask(2);
        executor.execute(task2);
    }

    public static void main(String[] args){
        test();
        System.out.println("main end");
    }

}
```

## FutureTask & 线程池

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.FutureTask;

class Task implements Callable<Integer>{
    String name;

    public Task(String name) {
        this.name = name;
    }

    @Override
    public Integer call() throws Exception {
        Integer res = new Random().nextInt(100);
        Thread.sleep(1000);
        System.out.println("任务执行:获取到结果 :"+res);
        return  res;
    }

    public String getName() {
        return name;
    }
}

public class Solution {

    public void testFutureAndThreadPool(){
        // 线程池
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        //进行异步任务列表
        List<FutureTask<Integer>> futureTasks = new ArrayList<>();
        long start = System.currentTimeMillis();
        int n = 10;
        for(int i=0;i<n;i++){
            Task task = new Task("name_" + i);
            //创建一个异步任务
            FutureTask<Integer> futureTask = new FutureTask<>(task);
            futureTasks.add(futureTask);
            //提交异步任务到线程池，让线程池管理任务。
            //由于是异步并行任务，所以这里并不会阻塞
            executorService.submit(futureTask);
        }
        int count = 0;
        for (FutureTask<Integer> futureTask : futureTasks) {
            // get()
            // get(long timeout, TimeUnit unit) 第一个参数为最大等待时间，第二个为时间的单位
            try{
                count += futureTask.get();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        //清理线程池
        executorService.shutdown();

        long end = System.currentTimeMillis();
        System.out.println("线程池的任务全部完成:结果为:"+count+"，main线程关闭，进行线程的清理");
        System.out.println("使用时间："+(end-start)+"ms");
    }

    public void testLine(){
        long start = System.currentTimeMillis();
        int n = 10;
        int count = 0;
        for(int i=0;i<n;i++){
            Task task = new Task("name_" + i);
            try{
                count += task.call();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("线程池的任务全部完成:结果为:"+count+"，main线程关闭，进行线程的清理");
        System.out.println("使用时间："+(end-start)+"ms");
        //清理线程池
    }

    public static void main(String[] args) {
        Solution solution = new Solution();
        solution.testFutureAndThreadPool();
        solution.testLine();
        System.out.println("the end");
    }

}
```

### 线程池新进来的任务被拒绝

1. 线程池shutdown了，新进来的任务会被拒绝
2. 线程池用满了，且新进来的任务超过了任务队列大小，任务被拒绝

```java
import java.text.SimpleDateFormat;
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Author mubi
 * @Date 2019/9/2 21:38
 */
public class ThreadPoolTest {
    static SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss,SSS");

    static RejectedExecutionHandler defaultHandler = new RejectedExecutionHandler() {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
            throw new RejectedExecutionException("MyTest Task " + r.toString() +
                    " rejected from " +
                    e.toString());
        }
    };

    static AtomicInteger atomicInteger = new AtomicInteger(0);

    ThreadFactory threadFactory = new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {
            Thread thread = new Thread(r);
            thread.setName("mythread-" + atomicInteger.get());
            atomicInteger.incrementAndGet();
            return thread;
        }
    };

    void test1(){
        ThreadPoolExecutor EXECUTOR =
                new ThreadPoolExecutor(5,
                        10,
                        3000L,
                        TimeUnit.MILLISECONDS,
                        new LinkedBlockingQueue<>(2),
                        threadFactory,
                        defaultHandler);
        int n = 2;
        for (int i = 0; i < n; i++) {
            // 使用前可以判断
//            if(!EXECUTOR.isShutdown()) {
            Thread t = threadFactory.newThread(new Runnable() {
                @Override
                public void run() {
                    System.out.println("Hello World");
                    try{
                        TimeUnit.SECONDS.sleep(5);
                    }catch (Exception e){

                    }
                    System.out.println("Hello World2");
                }
            });
            EXECUTOR.execute(t);
//            }else {
//                System.out.println("executor already shutdown");
//            }
            EXECUTOR.shutdown();
        }
    }

    void test2(){
        ThreadPoolExecutor EXECUTOR =
                new ThreadPoolExecutor(5,
                        10,
                        3000L,
                        TimeUnit.MILLISECONDS,
                        new LinkedBlockingQueue<>(2),
                        threadFactory,
                        defaultHandler
                );
        int n = 15;
        for (int i = 0; i < n; i++) {
            final int tmpint = i;
            EXECUTOR.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        System.out.println(tmpint+"Hello World");
                        TimeUnit.MILLISECONDS.sleep(1000L);
                    } catch (InterruptedException e) {
                    }
                }
            });
        }
        EXECUTOR.shutdown();
    }

    public static void main(final String[] args) throws Exception {
        ThreadPoolTest threadPoolTest = new ThreadPoolTest();
        threadPoolTest.test2();
        System.out.println("end main()");
    }

}
```
