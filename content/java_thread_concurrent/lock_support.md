---
title: "LockSupport Park & Unpark"
layout: page
date: 2020-05-31 00:00
---

[TOC]

# LockSupport

底层是native方法

```java
public native void park(boolean isAbsolute, long time);

public native void unpark(Object thread);
```

* Park

```java
public static void park(Object blocker) {
    // 获取当前运行线程
    Thread t = Thread.currentThread();
    // 设置t中的parkBlockerOffset的值为blocker，即设置锁遍历
    setBlocker(t, blocker);
    // 阻塞线程
    UNSAFE.park(false, 0L);
    // 线程被释放，则将parkBlockerOffset设为null
    setBlocker(t, null);
}

private static void setBlocker(Thread t, Object arg) {
    // Even though volatile, hotspot doesn't need a write barrier here.
    UNSAFE.putObject(t, parkBlockerOffset, arg);
}
```

* Unpark

```java
public static void unpark(Thread thread) {
    if (thread != null)
        UNSAFE.unpark(thread);
}
```

<a href="http://hg.openjdk.java.net/jdk8u/jdk8u40/hotspot/file/68577993c7db/src/share/vm/runtime/park.hpp">Java8 park.hpp定义</a>

<a href="http://hg.openjdk.java.net/jdk8u/jdk8u40/hotspot/file/95e9083cf4a7/src/os/solaris/vm/os_solaris.cpp">Java8 park的hpp实现</a>

## 例子

* 先park，后unpack

```java
static void testLockSupport(){
    Thread t1 = new Thread(()-> {
        System.out.println("t1 start");
        try {
            TimeUnit.SECONDS.sleep(1);
        }catch (Exception e){

        }
        System.out.println("t1 park");
        LockSupport.park();
        System.out.println("t2 after park");
    },"t1");


    t1.start();

    try {
        TimeUnit.SECONDS.sleep(2);
    }catch (Exception e){

    }

    System.out.println("main unpark");
    LockSupport.unpark(t1);
}
```

线程执行park后，属于wait状态

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_thread_concurrent/imgs/park_unpark.png)

* unpark 在 park 之前

```java
static void testLockSupport(){
    Thread t1 = new Thread(()-> {
        System.out.println("t1 start");
        try {
            TimeUnit.SECONDS.sleep(3);
        }catch (Exception e){

        }
        System.out.println("t1 park");
        LockSupport.park();
        System.out.println("t2 after park");
    },"t1");


    t1.start();

    try {
        TimeUnit.SECONDS.sleep(1);
    }catch (Exception e){

    }

    System.out.println("main unpark");
    LockSupport.unpark(t1);
}
```

输出如下:（park自己恢复了）

```java
t1 start
main unpark
t1 park
t2 after park
```

简单特点

1. park unpark 不需要对象的monitor锁
2. 以线程为单位【阻塞】或者【唤醒】线程，notify是随意唤醒，notifyAll是唤醒所有等待线程
3. park之前就可以unpark; 而wait之前不能notify

### 例子2，park使得线程wait

```java
static void testLockSupport(){
    Thread t1 = new Thread(()-> {
        System.out.println("t1 start");
        try {
            TimeUnit.SECONDS.sleep(3);
        }catch (Exception e){

        }
        System.out.println("t1 park");
        LockSupport.park();
        System.out.println("t2 after park");
    },"t1");


    t1.start();

    try {
        TimeUnit.SECONDS.sleep(1);
    }catch (Exception e){

    }

    System.out.println("main unpark");
//        LockSupport.unpark(t1);
}
```

* jstack 分析, 线程`waiting on condition`

```java
"t1" #10 prio=5 os_prio=31 tid=0x00007fbf7e290800 nid=0x4103 waiting on condition [0x000070000c06f000]
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:304)
	at Main.lambda$testLockSupport$3(Main.java:193)
	at Main$$Lambda$1/1651191114.run(Unknown Source)
	at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
	- None
```

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_thread_concurrent/imgs/thread_park.png)

## 进一步原理

每个线程有一个Parker对象，三部分组成`_counter`,`_cond`(条件变量,等待队列),`_mutex`,见：<a href="http://hg.openjdk.java.net/jdk8u/jdk8u40/hotspot/file/95e9083cf4a7/src/os/solaris/vm/os_solaris.cpp" target="_blank">Java8 park的hpp实现</a>

Park decrements _counter if > 0, else does a condvar wait. Unpark sets count to 1 and signals condvar. Only one thread ever waits on the condvar. Contention seen when trying to park implies that someone is unparking you, so don't wait. And spurious returns are fine, so there is no need to track notifications.

* 如果`_counter > 0`，那么`park`操作会对_counter减1操作，否则条件等待（使得线程阻塞）
* `unpark`操作会对_counter加1操作，并且唤醒条件变量（唤醒阻塞线程）

---

### 先 park 再 unpark

* park

1. 当前线程调用`park`方法
2. 检查_counter为0，则，获取_mutex互斥锁
3. 线程进入_cond条件变量阻塞
4. _counter仍然是0

* unpark

1. 其它线程调用`unpark`方法
2. 设置_counter为1
3. 把_cond条件变量中阻塞队列中的线程唤醒
4. 唤醒的线程执行起来，_counter会减1

### 先 unpark 再 park

1. `unpark`设置_counter为1
2. `park`操作发现_counter>0,减少1，不阻塞，接着执行
3. 最后_counter变成0
