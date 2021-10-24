---
title: "工厂模式"
layout: page
date: 2020-05-10 00:00
---

[TOC]

# 工厂模式

<a href="https://refactoring.guru/design-patterns/factory-method">工厂模式</a>

Factory Method is a creational design pattern that provides an interface for creating objects in a superclass, but allows subclasses to alter the type of objects that will be created.（Factory方法是一种创造性的设计模式，它提供了在超类中创建对象的接口，但允许子类更改将要创建的对象的类型。）

* 意图：定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。
* 主要解决：主要解决接口选择的问题。
* 何时使用：我们明确地计划不同条件下创建不同实例时。
* 如何解决：让其子类实现工厂接口，返回的也是一个抽象的产品。

## 简单工厂模式

eg:

```java
                  car
                /  
出行 -> 工厂 -  train
                \
                  ship
```

问题：突然新增了一个交通方式(比如两地可以直飞了); 则必须新增一个plane类，同时还要修改工厂方法（不符合开闭原则了）

```java
                  car
                /  
出行 ->  工厂   -  train
                \
                  ship
                \
                  plane  
```

代码eg:

```java
interface BaseOutService{
    void drive();
}

class CarServiceImpl implements BaseOutService{

    @Override
    public void drive() {
        System.out.println("car drive");
    }
}

class PlaneServiceImpl implements BaseOutService{

    @Override
    public void drive() {
        System.out.println("plane drive");
    }
}

class Factory {
    public BaseOutService getBean(String beanName){
        BaseOutService baseService = null;
        if(beanName.equalsIgnoreCase("car")){
            baseService = new CarServiceImpl();
        }
        if(beanName.equalsIgnoreCase("plane")){
            baseService = new PlaneServiceImpl();
        }
        return baseService;
    }
}


public class Solution {


    public static void main(String[] args) throws InterruptedException {
        Factory factory = new Factory();
        BaseOutService carService = factory.getBean("car");
        carService.drive();
        BaseOutService planeService = factory.getBean("plane");
        planeService.drive();
    }

}
```

## 抽象工厂

* 解决`开闭`问题（解耦）；不过增加了不少类，代码变多了

```java
                    工厂       car
                             /  
出行  -> 抽象工厂 ->  工厂  -    train
                             \
                    工厂       ship
                             \
                    工厂       plane
```

这样:增加一种新的plane方式,同时再新增一个plane工厂即可；原有代码不需要修改

```java
package factory;

interface Phone{
    void call();
}

class XiaoMiPhone implements Phone{

    @Override
    public void call() {
        System.out.println("Xiaomi call");
    }
}

class HuaweiPhone implements Phone{

    @Override
    public void call() {
        System.out.println("Huawei call");
    }
}

/**
 * 抽象工厂
 */
interface PhoneFactory{
    Phone genPhone();
}

class XiaomiPhoneFactory implements PhoneFactory{

    @Override
    public Phone genPhone() {
        return new XiaoMiPhone();
    }
}

class HuaweiPhoneFactory implements PhoneFactory{

    @Override
    public Phone genPhone() {
        return new HuaweiPhone();
    }
}

public class FactoryTest {

    public static void main(String[] args) {
        PhoneFactory phoneXiaomiFactory = new XiaomiPhoneFactory();
        Phone xiaomiPhone = phoneXiaomiFactory.genPhone();
        xiaomiPhone.call();

        PhoneFactory phoneHuaweiFactory = new HuaweiPhoneFactory();
        Phone huaweiPhone = phoneHuaweiFactory.genPhone();
        huaweiPhone.call();
    }
}
```
