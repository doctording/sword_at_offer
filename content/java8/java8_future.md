---
title: "Java8 Future"
layout: page
date: 2019-11-02 13:00
---

[TOC]

# Futur在规定时间内获取结果

```java
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Scanner;
import java.util.concurrent.*;


public class Main {

    public static Scanner sc = new Scanner(System.in);

    public static SimpleDateFormat simpleFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
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

    public static Future<Double> getPrice(double df){
        ExecutorService executor = Executors.newSingleThreadExecutor();
        Future<Double> future = executor.submit(new Callable<Double>() {
            @Override
            public Double call() throws Exception {
                delay(5);
                return df * 10;
            }
        });
        executor.shutdown();
        return future;
    }

    public static void priceTest(){
        Future<Double> futurePrice = getPrice(10.0f);
        Double price = null;
        try{
            price = futurePrice.get(1,TimeUnit.SECONDS);
        }catch (Exception e){
            System.out.println("cannot get within 1 sec");
            futurePrice.cancel(false);
            throw new RuntimeException(e);
        }
        System.out.println(price);
    }

    public static void main(String[] args) {
        System.out.println(simpleFormatter.format(new Date()));
        priceTest();
        System.out.println(simpleFormatter.format(new Date()));
    }

}
```

## 超时获取输出

```java
2019-11-02 15:14:47
cannot get within 1 sec
Exception in thread "main" java.lang.RuntimeException: java.util.concurrent.TimeoutException
	at Main.priceTest(Main.java:48)
	at Main.main(Main.java:55)
Caused by: java.util.concurrent.TimeoutException
	at java.util.concurrent.FutureTask.get(FutureTask.java:205)
	at Main.priceTest(Main.java:44)
	... 1 more

Process finished with exit code 1
```
