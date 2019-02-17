---
title: "Java8 Optional"
layout: page
date: 2019-02-17 13:00
---

[TOC]

# 用option取代null

## 经典的`java.lang.NullPointerException`问题

```java
class Insurance {
    private String name;
    public void setName(String name) { this.name = name; }
    public String getName() { return name; }
}

class Car {
    private Insurance insurance;
    public void setInsurance(Insurance insurance) { this.insurance = insurance; }
    public Insurance getInsurance() { return insurance; }

}
class Person {
    private Car car;
    public void setCar(Car car){ this.car = car; }
    public Car getCar() { return car; }
}


public class MainTest {

    public static String getCarInsuranceName(Person person) {
        return person.getCar().getInsurance().getName();
    }

    public static void main(String[] args) {
        Person person = new Person();
        String carInsuranceName = getCarInsuranceName(person);
        System.out.print(carInsuranceName);
    }
}
```

## 采用防御式检查减少`NullPointerException`

1. 深层质疑

```java
public static String getCarInsuranceName(Person person) {
    if(person != null) {
        if(person.getCar() != null) {
            if(person.getCar().getInsurance() != null) {
                return person.getCar().getInsurance().getName();
            }
        }
    }
    return "Unknown";
}
```

2. 过多的退出语句

```java
public static String getCarInsuranceName(Person person) {
    if(person != null) {
        return "Unknown";
    }
    if(person.getCar() != null) {
        return "Unknown";
    }
    if(person.getCar().getInsurance() != null) {
        return "Unknown";
    }
    return person.getCar().getInsurance().getName();
}
```

## `null` 带来的种种问题

* 它是错误之源

NullPointerException是目前Java程序开发中最典型的异常

* 它会使你的代码膨胀

它让你的代码充斥着深度嵌套的null检查，代码的可读性糟糕透顶

* 它自身是毫无意义的

null自身没有任何的语义，尤其是，它代表的是在静态类型语言中以一种错误的方式对缺少变量值的建模

* 它破坏了Java的哲学

Java一直试图避免让程序员意识到指针的存在，唯一的例外是:null指针。

* 它在Java的类型系统上开了个口子。

null并不属于任何类型，这意味着它可以被赋值给任意引用类型的变量。这会导致问题，原因是当这个变量被传递到系统中的另一个部分后，你将无法获知这个null变量最初的赋值到底是什么类型

## Optional

```java
public final class Optional<T> {
```

变量存在时，Optional类只是对类简单封装。变量不存在时，缺失的值会被建模成一个"空"的Optional对象，由方法`Optional.empty()`返回。`Optional.empty()`方法是一个静态工厂方法，它返回Optional类的特定单一实例。

使用Optional而不是null的一个非常重要而又实际的语义区别是，我们在声明变量时使用的是`Optional<Car>`类型，而不是`Car`类型，这句声明非常清楚地表明了这里发生变量缺失是允许的。与此相反，使用Car这样的类型，可能将变量赋值为null，这意味着你需要独立面对这些，你只能依赖你对业务模型的理解，判断一个null是否属于该变量的有效范畴。

### Optional 对象如何创建

* 几种创建方式

```java
// 依据一个非空值创建 Optional
Person person = new Person();
Optional<Person> optPerson = Optional.of(person);

// 声明一个空的 Optional
Optional<Person> optionalEmptyPerson = Optional.empty();

// 可接受 null 的 Optional
Person personNull = null;
Optional<Person> optionalNullPerson = Optional.ofNullable(personNull);
```

* 在域模型中使用Optional，以及为什么它们无法序列化

建议你像下面这个例子那样，提供一个能访问声明为Optional、变量值可能缺失的接口，

```java
public class Person {
    private Car car;
    public Optional<Car> getCarAsOptional() {
        return Optional.ofNullable(car);
    }
}
```

### Optional 使用实际例子

如果你对一个`空的Optional对象`调用`flatMap`，实际情况又会如何呢?结果不会发生任何改变，返回值也是个空的Optional对象(`Optional.empty`)。对null进行flatmap会抛出`NullPointerException`

```java
Optional<Person> personOptional = Optional.empty();
// 对 Optional.empty 进行 flatMap 会返回 Optional.empty
System.out.println( personOptional.flatMap(Person::getCar));
// 对 null 进行 flatMap 会抛出 java.lang.NullPointerException
personOptional = null;
personOptional.flatMap(Person::getCar);
```

```java

class Insurance {
    /**
     * 保险公司必须要有名字，此处非Optional
     */
    private String name;
    public void setName(String name) { this.name = name; }
    public String getName() { return name; }
}

class Car {
    /**
     * 车可能有保险，也可能没有保险，因此将这个字段声明为Optional
     */
    private Optional<Insurance> insurance;
    public void setInsurance(Optional<Insurance> insurance) { this.insurance = insurance; }
    public Optional<Insurance> getInsurance() { return insurance; }

}
class Person {
    /**
     * 人可能有车，也可能没有车，因此将这个字段声明为Optional
      */
    private Optional<Car> car;
    public void setCar(Optional<Car> car){ this.car = car; }
    public Optional<Car> getCar() { return car; }
}


public class MainTest {

    public static String getCarInsuranceName(Optional<Person> person) {
        /**
         * flatMap 链接 Optional 对象
         */
        return person.flatMap(Person::getCar)
                .flatMap(Car::getInsurance)
                .map(Insurance::getName)
                .orElse("Unknown");

    }

    public static void main(String[] args) {

        /**
         * 1.  Person 为空
         */
        Optional<Person> optionalEmptyPerson = Optional.empty();
        String carInsuranceName = getCarInsuranceName(optionalEmptyPerson);
        // Unknown
        System.out.println(carInsuranceName);

        /**
         * 2. Person非空，但 Car 为空
         */
        // 构造 Optional<Person>
        Person person = new Person();
        // person 非空，必须设置 Car
        person.setCar(Optional.empty());
        Optional<Person> optPerson = Optional.ofNullable(person);

        carInsuranceName = getCarInsuranceName(optPerson);
        // Unknown
        System.out.println(carInsuranceName);

        /**
         * 3. Person非空，Car 非空， 但 Insurance 为空
         */
        // 声明 Optional<Car>
        Car car = new Car();
        // car 非空，必须设置 Insurance
        car.setInsurance(Optional.empty());
        Optional<Car> optionalOfNullCar = Optional.ofNullable(car);
        // 构造 Optional<Person>
        person.setCar(optionalOfNullCar);
        optPerson = Optional.ofNullable(person);
        carInsuranceName = getCarInsuranceName(optPerson);
        // Unknown
        System.out.println(carInsuranceName);
    }
}
```
