---
title: "volatile 关键字"
layout: page
date: 2019-03-09 00:00
---

[TOC]

# volatile & synchronized

## 回顾并发的三大性质

### 原子性

一个操作或者多个操作，要么全部执行并 且执行的过程不会被任何因素打断，要么就都不执行。即使是在多个线程一起执行的时候，一个操作一旦开始，就不会被其它线程干扰.

* `volatile`：不能保证原子性，
* `synchronized`：在作用对象的作用范围内，依赖JVM实现操作的原子性。
* `Lock`：依赖特殊的CPU指令，代码实现，如`ReentrantLock`

### 可见性

当多个线程访问同一个变量的时候，一旦线程修改了这个变量的值，其它线程能够立即看到修改的值。

#### volatile 可见性例子

```java
public class Main {

    private static volatile Boolean flag = true;

    public static void main(String[] args) throws Exception{

       new Thread(new Runnable() {
            @Override
            public void run() {
                while (flag) {

                }
                System.out.println("A end");
            }
        }).start();

        try{
            TimeUnit.SECONDS.sleep(2);
        }catch (Exception e){

        }
        flag = false;

        System.out.println("main end");
    }
}
```

#### 导致共享变量在线程间不可见的原因

1. 线程交叉执行
2. 代码重排序结合线程交叉执行
3. 共享变量更新后的值没有在工作内存与主内存之间及时更新

`volatile` 通过**加入内存屏障**,**禁止指令重排优化**来实现可见性

即被volatile关键字修饰的变量，在每个写操作之后，都会加入一条store内存屏障命令，此命令强制工作内存将此变量的最新值保存至主内存；在每个读操作之前，都会加入一条load内存屏障命令，此命令强制工作内存从主内存中加载此变量的最新值至工作内存。

`synchronized` monitor enter exit 确保可见性

### 有序性

程序执行的顺序按照代码的先后顺序执行，**内存屏障**（JVM规范要求）

1. 每个volatile写操作的前面插入一个`StoreStore`屏障；
2. 在每个volatile写操作的后面插入一个`StoreLoad`屏障；

StoreStoreBarrier =》 写操作 =》 StoreStoreLoadBarrier

3. 在每个volatile读操作的后面插入一个`LoadLoad`屏障；
4. 在每个volatile读操作的后面插入一个`LoadStore`屏障。

LoadLoadBarrier =》 读操作 =》 LoadStoreBarrier

* Java内存模型中，允许**编译器**和**处理器**对指令进行重排序，但重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性(`CPU指令流水线`)

`volatile`、`syncronized`、`Lock`都可保证有序性。

<a target='_blank' href='https://blog.csdn.net/qq_30948019/article/details/80193392'>参考</a>

#### 指令乱序例子

```java
public class Main {

    static int x, y, a, b;

    public static void main(String[] args) throws Exception{
        int i = 0;
        while (true) {
            x = 0;
            y = 0;
            b = 0;
            a = 0;

            Thread A = new Thread(new Runnable() {
                @Override
                public void run() {
                    a = 1;
                    x = b;
                }
            });

            Thread B = new Thread(new Runnable() {
                @Override
                public void run() {
                    b = 1;
                    y = a;
                }
            });

            A.start();
            B.start();

            A.join();
            B.join();

            i++;
            if(x == 0 && y == 0){
                System.err.println(i + " " + x + " " + y);
                break;
            }
        }
        System.out.println("main end");
    }
}
```

#### double check 单例模式需要 volatile吗

正确答案：需要

`Object o = new Object();`的汇编指令

```java
0 new #2 <java/lang/Object>
3 dup
4 invokespecial #1 <java/lang/Object.<init>>
7 astore_1
8 return
```

隐含一个对象创建的过程：(记住3步)

1. 堆内存中申请了一块内存 （new指令）【半初始化状态，成员变量初始化为默认值】
2. 这块内存 构造方法执行（invokespecial指令）
3. 把栈中变脸，建立连接到这块内存（astore_1指令）

<font color='red'>问题</font>：由于指令重排和半初始化状态，导致多线程会使用半初始化的对象

附：<a href="https://doctording.github.io/sword_at_offer/design_pattern/singleton.html" target="_blank">单例模式</a>

## 回顾 Java 内存模型

CPU -> 缓存 -> 主存 -> 线程工作内存

<a target='_blank' href='https://doctording.github.io/sword_at_offer/java_jvm/jvm_mem_model.html'>参考</a>

## volatile 使用场景

volatile 无`原子性`，需要充分利用其的`可见性`和`顺序性`

### 利用可见性 进行开关控制

一个线程改变共享遍历，其它线程立刻能感知到，并根据其值执行各自的逻辑

### 利用顺序性

```java
线程A：
content = initContent();    //(1)
isInit = true;              //(2)
```

```java
线程B
while (isInit) {            //(3)
    content.operation();    //(4)
}
```

### Singleton 设计模式的 double-check 也是利用了顺序性的特点

<a target='_blank' href='https://doctording.github.io/sword_at_offer/java_utils/Singleton.html'>参考:单例模式</a>

```java
instance= new Singleton()

memory = allocate();    //1：分配对象的内存空间
ctorInstance(memory);  //2：初始化对象
instance = memory;     //3：设置instance指向刚分配的内存地址

可能指令重排

memory = allocate();    //1：分配对象的内存空间
instance = memory;     //3：instance指向刚分配的内存地址，此时对象还未初始化
ctorInstance(memory);  //2：初始化对象
```
