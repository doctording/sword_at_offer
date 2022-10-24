---
title: "观察者模式"
layout: page
date: 2019-03-20 00:00
---

[TOC]

# 观察者模式

也称为`订阅—发布模式`，在此模式中，一个目标对象管理所有相依于它的观察者对象，并且在它本身的状态改变时主动发出通知。这通常透过呼叫各观察者所提供的方法来实现。此种模式通常被用在事件处理系统。

观察者模式使用三个类: Subject(被观察的主体)、Observer(观察者) 和 测试类

Subject的操作会通知所有注册的Observer，Subject可以添加删除Observer

* Subject

```java
import java.util.List;
import java.util.Vector;

/**
 * 被观察主体
 * @Author mubi
 * @Date 2020/6/17 09:27
 */
public class Subject {
    // 观察者
    private List<Observer> observers = new Vector<>();
    // 观察主体的状态变化
    private int state;

    public int getState() {
        return state;
    }

    public void setState(int state) {
        this.state = state;
        notifyAllObservers();
    }

    public void addObserver(Observer observer){
        observers.add(observer);
    }

    public void delObserver(Observer observer){
        observers.remove(observer);
    }

    public void notifyAllObservers(){
        // 通知观察者，不同观察者执行不同的事情
        for (Observer observer : observers) {
            observer.update();
        }
    }
}
```

* Observer 抽象类

``` java
/**
 * @Author mubi
 * @Date 2020/6/17 09:27
 */
public abstract class Observer {
    protected Subject subject;

    public abstract void update();
}
```

* 具体的Observer类

```java
/**
 * @Author mubi
 * @Date 2020/6/17 09:27
 */
public class BinaryObserver extends Observer {

    public BinaryObserver(Subject subject) {
        this.subject = subject;
    }

    @Override
    public void update() {
        System.out.println("Binary String: " + Integer.toBinaryString(subject.getState()));
    }
}
```

```java
/**
 * @Author mubi
 * @Date 2020/6/17 09:28
 */
public class OctalObserver extends Observer {

    public OctalObserver(Subject subject) {
        this.subject = subject;
    }

    @Override
    public void update() {
        System.out.println("Octal String: " + Integer.toOctalString(subject.getState()));
    }
}
```

* 测试

```java
/**
 * @Author mubi
 * @Date 2020/6/17 09:32
 */
public class DesignTest {

    public static void main(String[] args) {
        Subject subject = new Subject();

        Observer observer1 = new BinaryObserver(subject);
        Observer observer2 = new OctalObserver(subject);

        subject.addObserver(observer1);
        subject.addObserver(observer2);

        subject.setState(10);
        System.out.println();

        subject.setState(12);
        System.out.println();

        subject.delObserver(observer1);

        subject.setState(13);
        System.out.println();
    }
}
```

输出：

```java
Binary String: 1010
Octal String: 12

Binary String: 1100
Octal String: 14

Octal String: 15 // 因为此前删除了 BinaryObserver
```
