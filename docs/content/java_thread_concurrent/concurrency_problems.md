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

* 具体是可重入锁 + 条件变量 wait/signal 机制: condition.await(), signal()
* 代码中 count 必须是 static + volatile
* for循环可以放在lock之前或之后都可以

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;


public class Hello extends Thread{

    static final int N = 5; // 循环次数，打印多少次
    // 必须有static，因为线程lambda表达式无法访问全局非静态变量
    volatile static int count = 0;

    public static void main(String[] args) throws Exception {
        // 利用可重入锁，lock就是 state + 1， unlock 就是 state - 1
        ReentrantLock lock = new ReentrantLock();
        Condition con1 = lock.newCondition();
        Condition con2 = lock.newCondition();
        Condition con3 = lock.newCondition();

        Thread t1 = new Thread(()->{
            System.out.println("start t1");
            try {
                lock.lock();
                for(int i=0;i<N;i++) {
                    while (count % 3 != 0) {
                        con1.await();
                    }
                    System.out.print("A");
                    count ++;
                    con2.signal();
                }
            } catch (Exception e) {

            } finally {
                lock.unlock();
            }
        });
        Thread t2 = new Thread(()->{
            System.out.println("start t2");
            try {
                lock.lock();
                for(int i=0;i<N;i++) {
                    while (count % 3 != 1) {
                        con2.await();
                    }
                    System.out.print("B");
                    count ++;
                    con3.signal();
                }
            } catch (Exception e) {

            } finally {
                lock.unlock();
            }
        });
        Thread t3 = new Thread(()->{
            System.out.println("start t3");
            try {
                lock.lock();
                for(int i=0;i<N;i++) {
                    while (count % 3 != 2) {
                        con3.await();
                    }
                    System.out.print("C");
                    count ++;
                    con1.signal();
                }
            } catch (Exception e) {

            } finally {
                lock.unlock();
            }
        });
        // 模拟线程的不同启动顺序，不过不影响本题结果
        t2.start();
        TimeUnit.SECONDS.sleep(1);
        t3.start();
        TimeUnit.SECONDS.sleep(1);
        t1.start();

        t1.join();
        t2.join();
        t3.join();
    }

}
```

## 方法2: synchronized & wait() notify() notifyAll()（状态同步 + wait/notify）

```java
import java.util.concurrent.TimeUnit;

public class Hello extends Thread{

    static final int N = 5; // 循环次数，打印多少次
    // 必须有static，因为线程lambda表达式无法访问全局非静态变量
    static int count = 0;
    // synchronized 也是可重入锁
    static Object object = new Object();

    public static void main(String[] args) throws Exception {

        System.out.println("main Start");

        Thread t1 = new Thread(()-> {
            System.out.println("start t1");
            synchronized (object) { // 竞争得到锁
                try {
                    for (int i = 0; i < N; i++) {
                        while (count % 3 != 0) {
                            object.wait(); // 在wait()所在的代码行处暂停执行，进入wait队列，并释放锁，直到接到通知或中断。
                        }
                        System.out.print("A");
                        count++;
                        object.notifyAll(); // 使所有正在等待队列中线程退出等待队列，进入就绪状态。执行notify方法后，当前线程并不会立即释放锁，要等到程序执行完，即退出synchronized同步区域后。
                    }
                }catch (Exception e){

                }
            }
        });
        Thread t2 = new Thread(()->{
            System.out.println("start t2");
            synchronized (object) {
                try {
                    for (int i = 0; i < N; i++) {
                        while (count % 3 != 1) {
                            object.wait();
                        }
                        System.out.print("B");
                        count++;
                        object.notifyAll();
                    }
                }catch (Exception e){

                }
            }
        });
        Thread t3 = new Thread(()->{
            System.out.println("start t3");
            synchronized (object) {
                try {
                    for (int i = 0; i < N; i++) {
                        while (count % 3 != 2) {
                            object.wait();
                        }
                        System.out.print("C");
                        count++;
                        object.notifyAll();
                    }
                }catch (Exception e){

                }
            }
        });
        // 模拟线程的不同启动顺序,不过不影响本题结果
        t2.start();
        TimeUnit.SECONDS.sleep(1);
        t3.start();
        TimeUnit.SECONDS.sleep(1);
        t1.start();

        tA.join();
        tB.join();
        tC.join();
        System.out.println();

        System.out.println("main End");
    }

}
```

## 方法3: 信号量`Semaphore`

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class Hello extends Thread{

    static final int N = 5; // 循环次数，打印多少次
    // 必须有static，因为线程lambda表达式无法访问全局非静态变量
    static int count = 0;

//    static Semaphore semaphoreAB = new Semaphore(1);
//    static Semaphore semaphoreBC = new Semaphore(0);
//    static Semaphore semaphoreCA = new Semaphore(0);

    public static void main(String[] args) throws Exception {

        // 初始化 semaphoreAB 的permits为1,其它permits都是0,这样只有线程 t1 能够 acquire 成功
        Semaphore semaphoreAB = new Semaphore(1);
        Semaphore semaphoreBC = new Semaphore(0);
        Semaphore semaphoreCA = new Semaphore(0);

        Thread t1 = new Thread(()-> {
            System.out.println("start t1");
            for (int i = 0; i < N; i++) {
                try {
                    semaphoreAB.acquire();
                    System.out.print("A");
                    count++;
                    semaphoreBC.release();
                }catch (Exception e){

                }
            }
        });
        Thread t2 = new Thread(()->{
            System.out.println("start t2");
            for (int i = 0; i < N; i++) {
                try {
                    semaphoreBC.acquire();
                    System.out.print("B");
                    count++;
                    semaphoreCA.release();
                }catch (Exception e){

                }
            }
        });
        Thread t3 = new Thread(()->{
            System.out.println("start t3");
            for (int i = 0; i < N; i++) {
                try {
                    semaphoreCA.acquire();
                    System.out.print("C");
                    count++;
                    semaphoreAB.release();
                }catch (Exception e){

                }
            }
        });
        // 模拟线程的不同启动顺序,不过不影响本题结果
        t2.start();
        TimeUnit.SECONDS.sleep(1);
        t3.start();
        TimeUnit.SECONDS.sleep(1);
        t1.start();
    }

}
```

# 按序打印：现在有T1、T2、T3三个线程，你怎样保证T2在T1执行完后执行，T3在T2执行完后执行？

同交替打印，如下信号量实现

```java
import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;


public class Hello extends Thread{

    public static void main(String[] args) throws Exception {

        // 初始化 semaphoreAB 的permits为1,其它permits都是0,这样只有线程 t1 能够 acquire 成功
        Semaphore semaphoreAB = new Semaphore(1);
        Semaphore semaphoreBC = new Semaphore(0);
        Semaphore semaphoreCA = new Semaphore(0);

        Thread t1 = new Thread(()-> {
            System.out.println("start t1");
            try {
                semaphoreAB.acquire();
                // a logic
                System.out.println("A");
                semaphoreBC.release();
            }catch (Exception e){

            }
        });
        Thread t2 = new Thread(()->{
            System.out.println("start t2");
            try {
                semaphoreBC.acquire();
                // b logic
                System.out.println("B");
                semaphoreCA.release();
            }catch (Exception e){

            }
        });
        Thread t3 = new Thread(()->{
            System.out.println("start t3");
            try {
                semaphoreCA.acquire();
                // c logic
                System.out.println("C");
                semaphoreAB.release();
            }catch (Exception e){

            }
        });
        // 模拟线程的不同启动顺序,不过不影响本题结果
        t2.start();
        TimeUnit.SECONDS.sleep(1);
        t3.start();
        TimeUnit.SECONDS.sleep(1);
        t1.start();
    }

}
```

# 多线程遍历数组打印（阿里社招）

给定一个数组`[1,2,3,4,5,6,7,8,9....,15]`，要求遍历数组，遇到可以同时被3和5整除的数字，打印C；遇到仅能被5整除的数字，打印B；遇到仅能被3整除的数字，打印A；其他打印数字本身；

要求四个线程，每一个线程执行一个打印方法。

```java
public class Hello implements Runnable {

    private static final int[] array = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16};

    private static ReentrantLock lock = new ReentrantLock();
    private static Condition condition = lock.newCondition();
    private static volatile int currentCount = 0;

    private int selfFlag;

    public Hello(int flag) {
        this.selfFlag = flag;
    }

    private int checkFlag(int n) {
        if (n % 3 == 0 && n % 5 == 0) {
            return 0;
        } else if (n % 5 == 0) {
            return 1;
        } else if (n % 3 == 0) {
            return 2;
        } else {
            return 3;
        }
    }

    @Override
    public void run() {
        while (true) {
            lock.lock();
            try {
                // 这里需要判断数组越界情况
                while (currentCount < array.length && checkFlag(array[currentCount]) % 4 != selfFlag) {
                    condition.await();
                }
                if (currentCount < array.length) {
                    if(selfFlag == 0){
                        System.out.print("C ");
                    } else if(selfFlag == 1){
                        System.out.print("B ");
                    } else if(selfFlag == 2){
                        System.out.print("A ");
                    } else {
                        System.out.print(array[currentCount] + " ");
                    }
                    currentCount++;
                    // 注意这里是 signalAll，一个lock下的条件变量可以唤醒其它所有的
                    condition.signalAll();
                } else {
                    return;
                }
            } catch (InterruptedException e) {
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        new Thread(new Hello(0)).start();
        new Thread(new Hello(1)).start();
        new Thread(new Hello(2)).start();
        new Thread(new Hello(3)).start();

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

# 生产者/消费者

## synchonized + wait + notity + buffer

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Random;


public class Hello extends Thread {

    // buffer 的总大小
    private final int MAX_LEN = 10;
    // buffer 队列
    private Queue<Integer> buffer = new LinkedList<>();

    class Producer extends Thread {
        @Override
        public void run() {
            producer();
        }
        private void producer() {
            while(true) {
                synchronized (buffer) {
                    while (buffer.size() == MAX_LEN) {
                        System.out.println("当前队列满");
                        try {
                            buffer.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    buffer.add(1);
                    System.out.println("生产者生产一条任务，当前队列长度为" + buffer.size());
                    try {
                        Thread.sleep(new Random().nextInt(200));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    buffer.notify();
                }
            }
        }
    }
    class Consumer extends Thread {
        @Override
        public void run() {
            consumer();
        }
        private void consumer() {
            while (true) {
                synchronized (buffer) {
                    while (buffer.size() == 0) {
                        System.out.println("当前队列为空");
                        try {
                            buffer.wait();
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                    buffer.poll();
                    System.out.println("消费者消费一条任务，当前队列长度为" + buffer.size());
                    try {
                        Thread.sleep(new Random().nextInt(200));
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    buffer.notify();
                }
            }
        }
    }
    public static void main(String[] args) {
        Hello hello = new Hello();
        Producer producer = hello.new Producer();
        Consumer consumer = hello.new Consumer();
        producer.start();
        consumer.start();
    }

}
```

## 信号量伪代码

```java
Deposit(c)
{
    empty->P();//检查是否还有空缓冲区

    mutex->P();
    Add c;
    mutex->V();

    full->V();//生产者生成了数据，需要将满缓冲区个数加1
}

Remove(c)
{
    full->P();//检查缓冲区里面是否还有东西，若缓冲区为空，则阻塞自己

    mutex->P();
    Remove c;
    mutex->V();

    empty->V();//消费者读取一个数据，释放一个空缓冲区资源
}
```

### Java Semaphore实现

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.Random;
import java.util.concurrent.Semaphore;


public class Hello extends Thread {

    // buffer 的总大小
    private final int MAX_LEN = 10;
    // buffer 队列
    private Queue<Integer> buffer = new LinkedList<>();

    // 作为一把互斥锁；直接用 ReentrantLock 也可以
    Semaphore mutex = new Semaphore(1);
    // 初始化为0的信号量
    Semaphore semaphoreFull = new Semaphore(0);
    // 初始化为 MAX_LEN 的信号量
    Semaphore semaphoreEmpty = new Semaphore(MAX_LEN);

    class Producer extends Thread {
        @Override
        public void run() {
            producer();
        }

        private void producer() {
            while(true) {
                try {
                    semaphoreEmpty.acquire(); // 信号量P操作，减少1

                    try {
                        mutex.acquire();

                        buffer.add(1);
                        System.out.println("生产者生产一条任务，当前队列长度为" + buffer.size());
                        Thread.sleep(new Random().nextInt(200));

                    } catch (Exception e) {
                    } finally {
                        mutex.release();
                    }

                    semaphoreFull.release(); // 信号量V操作，增加1
                } catch (Exception e){

                }
            }
        }
    }

    class Consumer extends Thread {
        @Override
        public void run() {
            consumer();
        }
        private void consumer() {
            while(true) {
                try {
                    semaphoreFull.acquire();

                    try {
                        mutex.acquire();

                        buffer.poll();
                        System.out.println("消费者消费一条任务，当前队列长度为" + buffer.size());
                        Thread.sleep(new Random().nextInt(200));

                    } catch (Exception e) {
                    } finally {
                        mutex.release();
                    }

                    semaphoreEmpty.release();
                } catch (Exception e){

                }
            }
        }
    }

    public static void main(String[] args) {
        Hello hello = new Hello();
        Producer producer = hello.new Producer();
        Consumer consumer = hello.new Consumer();
        producer.start();
        consumer.start();
    }
}
```
