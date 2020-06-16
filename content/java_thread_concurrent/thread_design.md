---
title: "多线程设计模式"
layout: page
date: 2020-06-09 00:00
---

[TOC]

# 多线程设计模式

## Immutable 模式

* eg: `SimpleDateFormat`线程不安全问题

```java
void test(){
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    for (int i = 0; i < 10; i++) {
        new Thread(()->{
            try {
                Date date = dateFormat.parse("1999-09-09");
                System.out.println(date);
            }catch (Exception e){
                e.printStackTrace();
            }
        }).start();
    }
}
```

* 某次运行输出如下，会报错:`NumberFormatException`

```java
Thu Sep 09 00:00:00 CST 1999
java.lang.NumberFormatException: For input string: ".909E.9092E"
Thu Sep 09 00:00:00 CST 1999
	at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:2043)
	at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
	at java.lang.Double.parseDouble(Double.java:538)
	at java.text.DigitList.getDouble(DigitList.java:169)
	at java.text.DecimalFormat.parse(DecimalFormat.java:2056)
	at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1869)
	at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
	at java.text.DateFormat.parse(DateFormat.java:364)
	at Main.lambda$test$0(Main.java:351)
	at java.lang.Thread.run(Thread.java:748)
java.lang.NumberFormatException: For input string: ".909E.9092E2"
Thu Sep 09 00:00:00 CST 1999
	at sun.misc.FloatingDecimal.readJavaFormatString(FloatingDecimal.java:2043)
Thu Sep 09 00:00:00 CST 1999
	at sun.misc.FloatingDecimal.parseDouble(FloatingDecimal.java:110)
	at java.lang.Double.parseDouble(Double.java:538)
	at java.text.DigitList.getDouble(DigitList.java:169)
	at java.text.DecimalFormat.parse(DecimalFormat.java:2056)
	at java.text.SimpleDateFormat.subParse(SimpleDateFormat.java:1869)
	at java.text.SimpleDateFormat.parse(SimpleDateFormat.java:1514)
	at java.text.DateFormat.parse(DateFormat.java:364)
	at Main.lambda$test$0(Main.java:351)
	at java.lang.Thread.run(Thread.java:748)
Thu Sep 09 00:00:00 CST 1999
Thu Sep 09 00:00:00 CST 1999
Thu Sep 09 00:00:00 CST 1999
Thu Sep 09 00:00:00 CST 1999
```

### 可以`synchronized`解决(互斥，效率低)

```java
void test(){
    SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
    for (int i = 0; i < 10; i++) {
        new Thread(()->{
            synchronized (dateFormat) {
                try {
                    Date date = dateFormat.parse("1999-09-09");
                    System.out.println(date);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

### DateTimeFormatter对象(immutable and thread-safe)

```java
void test(){
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
    for (int i = 0; i < 10; i++) {
        new Thread(()->{
            try {
                TemporalAccessor date = dateTimeFormatter.parse("1999-09-09");
                System.out.println(date);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

### 不可变设计模式

* eg: String类是不可变的（final类），`char[] value`也是final的

* final 保证 引用不能修改，值是可更改的； String保护性的拷贝(`深拷贝`)

矛盾： 对象会创建的比较多

## Flyweight pattern(享元模式)

### Integer.valueOf 例子

Integer.valueOf 会有一个缓存，数字在一定范围内不会创建新的对象

```java
/**
    * Returns an {@code Integer} instance representing the specified
    * {@code int} value.  If a new {@code Integer} instance is not
    * required, this method should generally be used in preference to
    * the constructor {@link #Integer(int)}, as this method is likely
    * to yield significantly better space and time performance by
    * caching frequently requested values.
    *
    * This method will always cache values in the range -128 to 127,
    * inclusive, and may cache other values outside of this range.
    *
    * @param  i an {@code int} value.
    * @return an {@code Integer} instance representing {@code i}.
    * @since  1.5
    */
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

Interger缓存

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

## Single Thread Execution

单线程模式：保证在同一时刻只能有一个线程访问共享资源

## Future设计模式

```java
public static void delay(int sec) {
    try {
        TimeUnit.SECONDS.sleep(sec);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
}

public static Future<Double> getPrice(double df){
    ExecutorService executor = Executors.newSingleThreadExecutor();
    Future<Double> future = executor.submit(new Callable<Double>() {
        @Override
        public Double call() throws Exception {
            delay(2);
            return df * 10;
        }
    });
    executor.shutdown();
    return future;
}

public static void priceTest(){
    Future<Double> futurePrice = getPrice(10.0f);
    Double price = null;
    try {
        int timeOut = 3;
        price = futurePrice.get(timeOut, TimeUnit.SECONDS);
    } catch (Exception e) {
        System.out.println("cannot get within 1 sec");
        futurePrice.cancel(false);
        throw new RuntimeException(e);
    }
    System.out.println("get price:" + price);
}
```

## work-stealing算法

一个大任务分割为若干个互不依赖的子任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应。

线程1 - 队列1（任务1，任务2，任务3，...）
线程2 - 队列2（任务1，任务2，任务3，...）

比如线程1早早的把队列中任务都处理完了有空闲，但是队列2执行任务较慢；这样队列2中任务可以让线程1帮忙执行（即`窃取`线程1）
