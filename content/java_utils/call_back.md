---
title: "回调 & 事件机制"
layout: page
date: 2019-04-13 00:00
---

[TOC]

# 同步 / 非同步调用

* 服务端

```java
package com.thirdpart;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Random;
import java.util.concurrent.TimeUnit;

/**
 * @Author mubi
 * @Date 2019/4/11 7:25 PM
 */
public class ServerBlock {

    private static SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss,SSS");

    String result = null;

    public int createJob(){
        int jobId = new Random().nextInt(100);
        System.out.println("Server do job start " + df.format(new Date()));
        try{
            TimeUnit.SECONDS.sleep(2);
        }catch (Exception e){
            e.printStackTrace();
        }
        System.out.println("Server do job end " + df.format(new Date()));
        result = "job done success";
        return jobId;
    }

    public String getJobResultByJobId(int jobId){
        return result;
    }
}
```

## 客户端同步调用

```java
static void sync(){
    ServerBlock server = new ServerBlock();
    try {
        // 阻塞在这里，直到返回JobId，期间什么都做不了
        int jobId = server.createJob();
        System.out.println("Job Id: " + jobId);
        System.out.println("Job Result:" + server.getJobResultByJobId(jobId));
    } catch (Exception e){
        e.printStackTrace();
    }
    System.out.println("==== end");
}
```

## 客户端异步调用

开启新线程去执行调用，自己做别的事去(不需要阻塞的等待调用结果)，之后再去获取调用的返回结果

```java
 static void async(){

    ServerBlock server = new ServerBlock();

    //创建Executor- Service，通 过它你可以 向线程池提 交任务
    ExecutorService executor = Executors.newCachedThreadPool();

    //向Executor- Service提交一个 Callable对象
    final Future<Integer> future = executor.submit(new Callable<Integer>() {
        @Override
        public Integer call() {
            //以异步方式在新的线程中执行耗时的操作
            return server.createJob();
        }
    });
    //异步操作进行的同时你可以做其他的事情
    System.out.println("=====do something else");
    try {
        // 获取异步操作的结果，如果最终被阻塞，无法得到结果，那么在最多等待 timeout 秒钟之后退出
        int jobId = future.get(3, TimeUnit.SECONDS);
        System.out.println("Job Id: " + jobId );
        System.out.println("Job Result:" + server.getJobResultByJobId(jobId));
    } catch (ExecutionException e) {
        // 计算抛出一个异常
        e.printStackTrace();
    } catch (InterruptedException ie) { // 当前线程在等待过程中被中断
        ie.printStackTrace();
    } catch (TimeoutException te) { // 在Future对象完成之前超过已过期
        te.printStackTrace();
    }
    executor.shutdown();
    System.out.println("==== end");
}
```

注意：

1. 调用Future的get()方法就能直接得到任务的返回值，该方法会一直阻塞直到任务的结果出来为止，我们可以调用Future的isDone()方法来判断该任务的结果是否准备就绪。

2. Future 是一种异步阻塞式的API，即没有通知回调。

## 异步回调

补充google guava的一种增强的 Future 为 ListenableFuture

```java
 static void asyncNotBlock(){
    ServerBlock server = new ServerBlock();

    ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));
    ListenableFuture future = service.submit(new Callable() {
        @Override
        public Integer call() {
            return server.createJob();
        }
    });

    Futures.addCallback(future, new FutureCallback<Integer>() {
        @Override
        public void onSuccess(Integer result) {
            // 回调执行
            int jobId = result;
            System.out.println("Job Id: " + jobId);
            System.out.println("Job Result:" + server.getJobResultByJobId(jobId));
        }

        @Override
        public void onFailure(Throwable t) {

        }
    });

    service.shutdown();
    System.out.println("==== end");
}
```

# 回调

对象a的方法`methodA()`中调用对象b的`methodB()`方法，在对象b的`methodB()`方法中反过来调用对象a的`callBack()`方法，这个`callBack()`方法称为回调函数，这种调用方法称为回调。

回调的核心在于：回调方将本身对象传给调用方，调用方在本身代码逻辑执行完之后，调用回调方的回调方法。

```java
package com.callback;

class A {
    public void methodA() {
        B b = new B();
        b.methodB(new A());
        System.out.println("this is class A method : methodA");
    }
    public void callBack() {
        System.out.println("this is class A method : callBack");
    }

}

class B {
    public void methodB(A a) {
        System.out.println("this is class B method : methodB");
        a.callBack();
    }
}

/**
 * @Author mubi
 * @Date 2019/4/11 11:41 PM
 */
public class MainTest {

    public static void main(String[] args){
        A a = new A();
        a.methodA();
    }

}
```

# 事件机制

1. event object（事件类，事件触发/发布）：事件状态对象，用于listener的相应的方法之中，作为参数，一般存在于listerner的方法之中

2. event source（事件源类）：具体的事件源，比如说，你点击一个button，那么button就是event source，要想使button对某些事件进行响应，你就需要注册特定的listener。

3. event listener（监听类）：对每个明确的事件的发生，都相应地定义一个明确的Java方法。这些方法都集中定义在事件监听者（EventListener）接口中，这个接口要继承 java.util.EventListener。 实现了事件监听者接口中一些或全部方法的类就是事件监听者。

```java

import java.text.SimpleDateFormat;
import java.util.*;

/**
 * 状态枚举类
 */
enum JobStatus{
    JOB_NONE("none"),
    JOB_RUNNING("running"),
    JOB_FINISHED("finished");
    JobStatus(String msg){
        this.msg = msg;
    }
    String msg;

    public String getMsg() {
        return msg;
    }
}

/**
 * 事件对象
 */
class JobEvent extends EventObject {

    private static final long serialVersionUID = 6496098798146410884L;

    private JobStatus state;

    public JobEvent(Object source, JobStatus state) {
        super(source);
        this.state = state;
    }

    public JobStatus getState() {
        return state;
    }
}

/**
 * 事件监听器 interface
 */
interface JobListener extends EventListener {
    void jobEventDeal(JobEvent event);
}

/**
 * 具体监听器 实现1
 */
class JobListener1 implements JobListener{

    @Override
    public void jobEventDeal(JobEvent event) {
        String s = null;
        if(null != event.getState() && event.getState().equals(JobStatus.JOB_NONE)){
            s = String.format("%s 监听到: Job 不存在", this.getClass().getName());
        }
        if(null != event.getState() && event.getState().equals(JobStatus.JOB_RUNNING)){
            s = String.format("%s 监听到: 有 Job 运行中", this.getClass().getName());
        }
        if(null != event.getState() && event.getState().equals(JobStatus.JOB_FINISHED)){
            s = String.format("%s 监听到: Job 已结束", this.getClass().getName());
        }
        System.out.println(s);
    }
}

/**
 * 事件源
 */
class JobSource{
    private Collection listeners;

    public void addListener(JobListener listener){
        if(null == listeners){
            this.listeners = new HashSet<JobListener>(4);
        }
        if(null != listener){
            String s = String.format("添加了一个监听器：%s", listener.getClass().getName());
            System.out.println(s);
            listeners.add(listener);
        }
    }

    public void removeListener(JobListener listener) {
        if(null == listener) {
            return;
        }
        if(null != listeners) {
            String s = String.format("删除了一个监听器：%s", listener.getClass().getName());
            System.out.println(s);
            listeners.remove(listener);
        }
    }

    /**
     * 触发 Job Running 事件
     */
    protected void fireJobRunning() {
        if (null == listeners) {
            return;
        }
        String s = String.format("产生新事件：%s", JobStatus.JOB_RUNNING.getMsg());
        System.out.println(s);
        JobEvent event = new JobEvent(this, JobStatus.JOB_RUNNING);
        notifyListeners(event);
    }

    /**
     * 通知所有的 Listener
     */
    private void notifyListeners(JobEvent event) {
        Iterator iterator = listeners.iterator();
        while (iterator.hasNext()) {
            JobListener listener = (JobListener) iterator.next();
            listener.jobEventDeal(event);
        }
    }
}

/**
 * @Author mubi
 * @Date 2019/4/11 11:18 PM
 */
public class ClientMain {

    private static SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss,SSS");

    public static void main(String[] args){
        // 事件源
        JobSource jobSource = new JobSource();
        // 给 时间源 注册 事件监听器
        JobListener jobListener = new JobListener1();
        jobSource.addListener(jobListener);
        // 产生事件
        jobSource.fireJobRunning();
        // 移除监听器
        jobSource.removeListener(jobListener);
    }
}

```

* output

```java
添加了一个监听器：com.client.JobListener1
产生新事件：running
com.client.JobListener1 监听到: 有 Job 运行中
删除了一个监听器：com.client.JobListener1
```
