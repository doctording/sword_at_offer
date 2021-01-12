---
title: "ShutdownHook"
layout: page
date: 2019-04-13 00:00
---

[TOC]

# ShutdownHook

在Java程序中可以通过添加关闭钩子，实现在程序退出时关闭资源、平滑退出的功能。

使用`Runtime.addShutdownHook(Thread hook)`方法，可以注册一个JVM关闭的钩子，这个钩子可以在以下几种场景被调用：

* 程序正常退出
* 使用System.exit()
* 终端使用Ctrl+C触发的中断
* 系统关闭
* 使用Kill pid命令干掉进程(kill -9 不会)
* OutOfMemory宕机等

## demo1

```java
// 创建HookTest，我们通过main方法来模拟应用程序
public class HookTest {
 
    public static void main(String[] args) {
 
        // 添加hook thread，重写其run方法
        Runtime.getRuntime().addShutdownHook(new Thread(){
            @Override
            public void run() {
                System.out.println("this is hook demo...");
                // TODO
            }
        });
 
        int i = 0;
        // 这里会报错，我们验证写是否会执行hook thread
        int j = 10/i;
        System.out.println("j" + j);
    }
}
/*
Exception in thread "main" java.lang.ArithmeticException: / by zero
	at hook.HookTest.main(HookTest.java:23)
this is hook demo...
 
Process finished with exit code 1
*/
```

## demo2

```java
import java.util.concurrent.TimeUnit;

/**
 * @Author mubi
 * @Date 2020/12/26 09:58
 * -Xmx20m
 */
public class HookTestOom {

    public void start()
    {
        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
            @Override
            public void run()
            {
                System.out.println("Execute Hook.....");
            }
        }));
    }

    public static void main(String[] args)
    {
        new HookTestOom().start();
        System.out.println("The Application is doing something");
        byte[] b = new byte[500*1024*1024];
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("The Application is done:" + b);
    }


}
```

程序正常退出或者发生OOM时，都能执行到钩子函数
