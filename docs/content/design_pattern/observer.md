---
title: "观察者模式"
layout: page
date: 2019-03-20 00:00
---

[TOC]

# 观察者模式

观察者模式使用三个类: Subject(被观察的主体)、Observer(观察者) 和 Client

Subject的操作会通知所有注册的Observer,Subject可以添加删除Observer

* Subject

```java
package design;

import java.util.List;
import java.util.Vector;

/**
 * @Author mubi
 * @Date 2020/6/17 09:27
 */
public class Subject {

    private List<Observer> observers = new Vector<>();
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
        for (Observer observer : observers) {
            observer.update();
        }
    }
}
```

* Observer 抽象类

```java
package design;

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
package design;

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
package design;

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
package design;

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

Octal String: 15 // 此前删除了 BinaryObserver
```
