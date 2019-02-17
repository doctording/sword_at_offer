---
title: "Java8 lambda表达式"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# lambda

## 需求不断变化的找苹果

函数式编程，方法和Lambda作为一等公民

```java
public class Apple {

    private String color;
    private long weight;

    public Apple() {
    }

    public Apple(String color, long weight) {
        this.color = color;
        this.weight = weight;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public long getWeight() {
        return weight;
    }

    public void setWeight(long weight) {
        this.weight = weight;
    }

    @Override
    public String toString() {
        return "Apple{" +
                "color='" + color + '\'' +
                ", weight=" + weight +
                '}';
    }
}
```

```java
@FunctionalInterface
public interface AppleFilter {
    /**
        * 过滤器接口
        * @param apple
        * @return
        */
    boolean filter(Apple apple);

}

public static List<Apple> findApple(List<Apple> apples, AppleFilter appleFilter) {
    List<Apple> list = new ArrayList<>();

    for (Apple apple : apples) {
        if (appleFilter.filter(apple)) {
            list.add(apple);
        }
    }
    return list;
}

```

```java
// lambda表达式，找到绿色的苹果
List<Apple> lambdaResult = findApple(list, apple -> apple.getColor().equals("green"));
```

### java8 与 java7 内存空间的区别 // TODO

```java
^Cmubi@mubideMacBook-Pro Home $ pwd
/Library/Java/JavaVirtualMachines/jdk1.7.0_80.jdk/Contents/Home
mubi@mubideMacBook-Pro Home $ bin/jstat -gcutil 62850 1000 10
Warning: Unresolved Symbol: sun.gc.generation.2.space.0.capacity substituted NaN
Warning: Unresolved Symbol: sun.gc.generation.2.space.0.used substituted NaN
Warning: Unresolved Symbol: sun.gc.generation.2.space.0.capacity substituted NaN
  S0     S1     E      O      P     YGC     YGCT    FGC    FGCT     GCT
  0.00  85.81  76.97  56.33      �    161    1.624    14    0.603    2.227
  0.00  85.81  77.08  56.33      �    161    1.624    14    0.603    2.227
  0.00  85.81  77.22  56.33      �    161    1.624    14    0.603    2.227
  0.00  85.81  77.33  56.33      �    161    1.624    14    0.603    2.227
^Cmubi@mubideMacBook-Pro Home $ jstat -gcutil 62850 1000 10
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00  85.81  78.86  56.33  93.46  89.80    161    1.624    14    0.603    2.227
  0.00  85.81  78.93  56.33  93.46  89.80    161    1.624    14    0.603    2.227
  0.00  85.81  79.04  56.33  93.46  89.80    161    1.624    14    0.603    2.227
  0.00  85.81  79.17  56.33  93.46  89.80    161    1.624    14    0.603    2.227
  0.00  85.81  79.29  56.33  93.46  89.80    161    1.624    14    0.603    2.227
^Cmubi@mubideMacBook-Pro Home $
```

java7:

* S0  — Heap上的 Survivor space 0 区已使用空间的百分比
* S1  — Heap上的 Survivor space 1 区已使用空间的百分比
* E   — Heap上的 Eden space 区已使用空间的百分比
* O   — Heap上的 Old space 区已使用空间的百分比
* P   — Perm space 区已使用空间的百分比
* YGC — 从应用程序启动到采样时发生 Young GC 的次数
* YGCT– 从应用程序启动到采样时 Young GC 所用的时间(单位秒)
* FGC — 从应用程序启动到采样时发生 Full GC 的次数
* FGCT– 从应用程序启动到采样时 Full GC 所用的时间(单位秒)
* GCT — 从应用程序启动到采样时用于垃圾回收的总时间(单位秒)

java8:

少了一个 Perm space

新增了

* M   - 元空间（Metaspace）： Klass Metaspace, NoKlass Metaspace
* CCS - 表示的是NoKlass Metaspace的使用率

### `@FunctionalInterface` 接口使用条件

只能定义了唯一的抽象方法的接口

如下的默认方法不是，不是抽象方法

![](./imgs/functionalinterface.png)

如下正确: 函数式接口里允许定义`默认方法`和`静态方法`,也可以包含`Object`里的`public`方法

```java
@FunctionalInterface
public interface AppleFilter2 {
    boolean filter(Apple apple);

    default boolean filter2(){
        return true;
    }

    static boolean filter3(){
        return true;
    }

    @Override
    boolean equals(Object obj);

    @Override
    String toString();

}
```

### lambda的使用

#### 基本语法

```java
(parameters) -> expression

(parameters) -> { statements; }
```

```bash
(1) () -> {}
(2) () -> "Raoul"
(3) () -> {return "Mario";}
(4) (Integer i) -> return "Alan" + i;
(5) (String s) -> {"IronMan";}

(4),(5) 无效
(4) 需要加上`{}`
(5) 需要去掉`{}`和`;`
```

#### java8源码使用lambda的场景

```java
@FunctionalInterface
public interface Comparator<T> {

@FunctionalInterface
public interface Runnable {

@FunctionalInterface
public interface Function<T, R> {
```

#### 复合式lambda表达式

* 比较器复合

1. 逆序
2. 比较器链

* 谓词复合

negate,and,or

* 函数复合

`Funtion`接口

#### 《java8 in action》 lambda总结

* `Lambda`表达式可以理解为一种匿名函数:它没有名称，但有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常的列表。
* `Lambda`表达式让你可以简洁地传递代码。
* 函数式接口就是仅仅声明了一个抽象方法的接口。
* 只有在接受函数式接口的地方才可以使用Lambda表达式。
* `Lambda`表达式允许你直接内联，为函数式接口的抽象方法提供实现，并且`将整个表达式作为函数式接口的一个实例`。
* Java 8自带一些常用的函数式接口，放在`java.util.function`包里，包括`Predicate<T>`、`Function<T,R>`、`Supplier<T>`、`Consumer<T>`和`BinaryOperator<T>`
* 为了避免装箱操作，对`Predicate<T>`和`Function<T, R>`等通用函数式接口的原始类型
特化:`IntPredicate`、`IntToLongFunction`等。
* 环绕执行模式(即在方法所必需的代码中间，你需要执行点儿什么操作，比如资源分配 和清理)可以配合Lambda提高灵活性和可重用性。
* `Lambda`表达式所需要代表的类型称为目标类型。
* 方法引用让你重复使用现有的方法实现并直接传递它们。
* `Comparator`、`Predicate`和`Function`等函数式接口都有几个可以用来结合Lambda表达式的默认方法。