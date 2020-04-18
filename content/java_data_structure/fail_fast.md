---
title: "集合 fail-fast,ConcurrentModificationException"
layout: page
date: 2019-05-25 00:00
---

[TOC]

# fail-fast（快速失败）

fail-fast,即快速失败机制，它是java集合中的一种错误检测机制，当多个线程（当个线程也是可以滴）,在结构上对集合进行改变时，就有可能会产生fail-fast机制。

如：在使用迭代器对集合对象进行遍历的时候，如果 A 线程正在对集合进行遍历，此时 B 线程对集合进行修改（增加、删除、修改），或者 A 线程在遍历过程中对集合进行修改，都会导致 A 线程抛出 `ConcurrentModificationException` 异常。

* eg：

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;


public class Main {


    private static List<Integer> list = new ArrayList<>();

    // 添加
    private static class ThreadA extends Thread{

        ThreadA(){
            this.setName("A");
        }

        @Override
        public void run() {
            Iterator<Integer> iterator = list.iterator();
            while(iterator.hasNext()){
                int i = iterator.next();
                System.out.println("ThreadOne 遍历:" + i);
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    // 修改
    private static class ThreadB extends Thread{

        ThreadB(){
            this.setName("B");
        }

        @Override
        public void run(){
            int i = 0 ;
            while(i < 6){
                System.out.println("ThreadTwo run：" + i);
                if(i == 3){
                    list.remove(i);
                }
                i++;
            }
        }
    }

    public static void main(String[] args) {
        for(int i = 0 ; i < 10;i++){
            list.add(i);
        }
        new ThreadA().start();
        new ThreadB().start();
    }

}
```

* output

```java
ThreadOne 遍历:0
ThreadTwo run：0
ThreadTwo run：1
ThreadTwo run：2
ThreadTwo run：3
ThreadTwo run：4
ThreadTwo run：5
Exception in thread "A" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at Main$ThreadA.run(Main.java:24)
```

# fail-safe（安全失败）

采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。

由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，故不会抛 ConcurrentModificationException 异常

```java
import java.util.Iterator;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;


public class Main {


    private static List<Integer> list = new CopyOnWriteArrayList<>();

    // 添加
    private static class ThreadA extends Thread{

        ThreadA(){
            this.setName("A");
        }

        @Override
        public void run() {
            Iterator<Integer> iterator = list.iterator();
            while(iterator.hasNext()){
                int i = iterator.next();
                System.out.println("ThreadOne 遍历:" + i);
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    // 修改
    private static class ThreadB extends Thread{

        ThreadB(){
            this.setName("B");
        }

        @Override
        public void run(){
            int i = 0 ;
            while(i < 6){
                System.out.println("ThreadTwo run：" + i);
                if(i == 3){
                    list.remove(i);
                }
                i++;
            }
        }
    }

    public static void main(String[] args) {
        for(int i = 0 ; i < 10;i++){
            list.add(i);
        }
        try {
            ThreadA threadA = new ThreadA();
            ThreadB threadB = new ThreadB();
            threadA.start();
            threadB.start();

            threadA.join();
            threadB.join();
        }catch (Exception e){

        }
        for(Integer val: list){
           System.out.print(" " + val);
        }
        System.out.println();
    }
}
```

* output

```java
ThreadOne 遍历:0
ThreadTwo run：0
ThreadTwo run：1
ThreadTwo run：2
ThreadTwo run：3
ThreadTwo run：4
ThreadTwo run：5
ThreadOne 遍历:1
ThreadOne 遍历:2
ThreadOne 遍历:3
ThreadOne 遍历:4
ThreadOne 遍历:5
ThreadOne 遍历:6
ThreadOne 遍历:7
ThreadOne 遍历:8
ThreadOne 遍历:9
 0 1 2 4 5 6 7 8 9
```
