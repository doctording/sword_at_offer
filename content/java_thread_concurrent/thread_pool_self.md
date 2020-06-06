---
title: "自定义线程池"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# 自定义线程池步骤

1. 自定义一个阻塞队列

2. 线程池：工作线程核心数，任务的接收执行（没有最大线程数量）

3. 自定义队列满时新增任务的拒绝策略

4. 测试验证

```java
class SelfBlockingQueue<T> {
    // 任务队列
    Deque<T> queue = new ArrayDeque<>();

    Lock lock = new ReentrantLock();

    Condition notFull = lock.newCondition();

    Condition notEmpty = lock.newCondition();

    // 容量
    private int capcity;

    SelfBlockingQueue(int cap){
        capcity = cap;
    }

    public void put(T val){
        lock.lock();
        try {
            while (getSize() == capcity) {
                System.out.println("queue is Full now. notFull await ...");
                try {
                    System.out.println("等待加入任务队列..." + val);
                    notFull.await();
                } catch (Exception e) {

                }
            }
            System.out.println("加入任务队列..." + val);
            queue.addLast(val);
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public boolean offer(T task, long timeout, TimeUnit timeUnit){
        long time = timeUnit.toNanos(timeout);
        lock.lock();
        try {
            while (getSize() == capcity) {
                System.out.println("queue is Full now. notFull await ...");
                try {
                    System.out.println("等待加入任务队列..." + task);
                    if(time <= 0){
                        System.out.println("等待加入任务队列...超时..." + task);
                        return false;
                    }
                    time = notFull.awaitNanos(time);
                } catch (Exception e) {

                }
            }
            System.out.println("加入任务队列..." + task);
            queue.addLast(task);
            notEmpty.signal();
            return true;
        } finally {
            lock.unlock();
        }
    }

    public T take(){
        lock.lock();
        try {
            while (queue.isEmpty()){
                System.out.println("queue is Empty now. notEmpty await ...");
                try {
                    notEmpty.await();
                } catch (Exception e) {

                }
            }
            T e = queue.removeFirst();
            notFull.signal();
            return e;
        } finally {
            lock.unlock();
        }
    }

    public T poll(long timeout, TimeUnit unit){
        lock.lock();
        try {
            long time = unit.toNanos(timeout);
            while (queue.isEmpty()){
                System.out.println("queue is Empty now. notEmpty await ...");
                try{
                    if(time <= 0){
                        return null;
                    }
                    // 返回剩余等待时间
                    time = notEmpty.awaitNanos(time);
                }catch (Exception e){

                }
            }
            T e = queue.removeFirst();
            notFull.signal();
            return e;
        } finally {
            lock.unlock();
        }
    }


    public int getSize(){
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }

}

@FunctionalInterface
interface RejectPolicy<T> {
    void reject(SelfBlockingQueue<T> queue, T task);
}

class SelfThreadPool{
    // 任务队列
    private SelfBlockingQueue<Runnable> taskQueue;
    // 任务对列大小
    int capcity;

    // 工作线程集合
    private HashSet workers = new HashSet();

    // 核心线程数
    int coreSize;

    // 超时时间，销毁线程
    long timeout;
    TimeUnit timeUnit;

    RejectPolicy<Runnable> rejectPolicy;

    public SelfThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int capcity, RejectPolicy rejectPolicy) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.taskQueue = new SelfBlockingQueue<>(capcity);
        this.rejectPolicy = rejectPolicy;
    }

    // 执行某个任务
    public void execute(Runnable task){
        synchronized (workers) {
            // 小于 coreSize 则直接Worker执行，否则加入任务队列
            if (workers.size() < coreSize) {
                System.out.println("新增worker..." + task);
                Worker worker = new Worker(task);
                worker.start();
                workers.add(worker);
            } else {
                System.out.println("加入队列等待..." + task);
                // 1. 死等
//                taskQueue.put(task);
                // 2. 等待超时
                // 3. 放弃任务执行
                // 4. 抛出异常
                // 5. 调用者自己去执行
                tryReject(task);
            }
        }
    }

    void tryReject(Runnable task){
        rejectPolicy.reject(taskQueue, task);
    }

    // 工作
    class Worker extends Thread{

        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run(){
            // 执行任务,当前任务；任务队列中
            // 1. 阻塞等待新任务
//            while (task != null || (task = taskQueue.take()) != null){
            // 2. 超时等待新任务，超时则结束核心线程
            while (task != null || (task = taskQueue.poll(timeout, TimeUnit.SECONDS)) != null){
                try {
                    System.out.println("正在运行..." + task);
                    task.run();
                } catch (Exception e) {

                } finally {
                    task = null;
                }
            }
            synchronized (workers){
                System.out.println("任务执行完，worker被移除..." + this);
                workers.remove(this);
            }
        }
    }
}

public class Main {

//    static void testPool(){
//        SelfThreadPool selfThreadPool = new SelfThreadPool(2, 1, TimeUnit.SECONDS, 3);
//        for(int i=0;i<5;i++) {
//            int[] num = new int[]{i};
//            Runnable r = () -> {
//                try {
//                    TimeUnit.MILLISECONDS.sleep(500);
//                    System.out.println("running over..." + num[0]);
//                } catch (Exception e) {
//
//                }
//            };
//            selfThreadPool.execute(r);
//        }
//    }

//    static void testPool2(){
//        SelfThreadPool selfThreadPool = new SelfThreadPool(2, 1, TimeUnit.SECONDS, 3);
//        for(int i=0;i<10;i++) {
//            int[] num = new int[]{i};
//            Runnable r = () -> {
//                try {
//                    TimeUnit.MILLISECONDS.sleep(2000);
//                    System.out.println("running over..." + num[0]);
//                } catch (Exception e) {
//
//                }
//            };
//            selfThreadPool.execute(r);
//        }
//    }

    static void testPool3(){
        RejectPolicy rejectPolicy1 = new RejectPolicy() {
            @Override
            public void reject(SelfBlockingQueue queue, Object task) {
                queue.put(task);
            }
        };

        RejectPolicy rejectPolicy2 = new RejectPolicy() {
            @Override
            public void reject(SelfBlockingQueue queue, Object task) {
                queue.offer(task, 500, TimeUnit.MILLISECONDS);
            }
        };

        RejectPolicy rejectPolicy3 = new RejectPolicy() {
            @Override
            public void reject(SelfBlockingQueue queue, Object task) {
                throw new RuntimeException("任务被拒绝执行..." + task);
            }
        };

        RejectPolicy<Runnable> rejectPolicy4 = new RejectPolicy<Runnable>() {
            @Override
            public void reject(SelfBlockingQueue<Runnable> queue, Runnable task) {
                task.run();
            }
        };

        SelfThreadPool selfThreadPool = new SelfThreadPool(2, 1,
                TimeUnit.SECONDS, 3, rejectPolicy4);
        for(int i=0;i<10;i++) {
            int[] num = new int[]{i};
            Runnable r = () -> {
                try {
                    TimeUnit.MILLISECONDS.sleep(1000);
                    System.out.println("running over..." + num[0]);
                } catch (Exception e) {

                }
            };
            selfThreadPool.execute(r);
        }
    }

    public static void main(String[] args) throws Exception{
        testPool3();
    }

}
```
