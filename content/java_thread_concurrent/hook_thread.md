---
title: "Hook线程"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# Hook线程以及捕获线程执行异常

## 获取线程运行时异常

当线程在运行过程中出现异常时，JVM会调用`dispatchUncaughtException`方法，该方法会将对应的线程实例以及异常信息传递给会调接口

### 例子代码

```java
import java.util.concurrent.TimeUnit;

/**
 * @Author mubi
 * @Date 2020/6/25 11:07
 */
public class CaptureThreadException {

    public static void main(String[] args) {
        Thread.setDefaultUncaughtExceptionHandler((t,e)->{
            System.out.println("thread:" + t.getName() + " occur exception");
            e.printStackTrace();
        });

        final Thread thread = new Thread(()->{
            try{
                TimeUnit.SECONDS.sleep(2);
            }catch (InterruptedException e){

            }
            // 有异常抛出
            System.out.println(1 / 0);
        }, "test-e");
        thread.start();
    }
}
```

* 程序输出（捕获到线程异常）

```java
thread:test-e occur exception
java.lang.ArithmeticException: / by zero
	at com.thread.CaptureThreadException.lambda$main$1(CaptureThreadException.java:24)
	at java.lang.Thread.run(Thread.java:748)
```

* 异常不断的往上抛：【线程异常】->【MainGroup】->【SystemGroup】->【System.err】

```java
public class CaptureThreadException {

    public static void main(String[] args) {
        // 获取当前线程的线程组
        ThreadGroup mainGroup = Thread.currentThread().getThreadGroup();
        System.out.println(mainGroup.getName());
        System.out.println(mainGroup.getParent());
        System.out.println(mainGroup.getParent().getParent());


//        Thread.setDefaultUncaughtExceptionHandler((t,e)->{
//            System.out.println("thread:" + t.getName() + " occur exception");
//            e.printStackTrace();
//        });

        final Thread thread = new Thread(()->{
            try{
                TimeUnit.SECONDS.sleep(2);
            }catch (InterruptedException e){

            }
            // 有异常抛出
            System.out.println(1 / 0);
        }, "test-e");
        thread.start();
    }
}
```

## 注入钩子线程

JVM进程的退出是由于JVM进程中没有活跃的非守护线程，或者收到了系统终端信号，向JVM程序注入一个Hook线程，在JVM进程退出的时候，Hook线程会启动执行，通过`Runtime`可以为JVM注入多个Hook线程

```java
public static void main(String[] args) {

    Runtime.getRuntime().addShutdownHook(new Thread(()->{
        try {
            System.out.println("the hook thread 1 is running.");
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("the hook thread 1 will exit.");
    }));
    // 钩子线程可以注册多个
    Runtime.getRuntime().addShutdownHook(new Thread(()->{
        try {
            System.out.println("the hook thread 2 is running.");
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("the hook thread 2 will exit.");
    }));

    System.out.println("the program will is stopping.");
}
```

### Hook线程的应用

防止重复启动，启动的时候创建lock文件，进程中断或退出的时候删除lock文件

```java
public class PreventDupStart {
    private final static String LOCK_PATH = ".";
    private final static String LOCK_FILE = ".lock";
    private final static String PERMISSION = "rw-------";

    public static void main(String[] args) throws Exception{

        Runtime.getRuntime().addShutdownHook(new Thread(()->{
            try {
                System.out.println("the program received kill SIGNAL.");
                getLockFile().toFile().delete();
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println("the hook thread 1 will exit.");
        }));

        // 检查是否存在.lock文件
        checkRunning();

        // 模拟程序运行
        for (;;){
            try{
                TimeUnit.MILLISECONDS.sleep(100);
                System.out.println("program is running");
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }

    private static void checkRunning() throws IOException {
        Path path = getLockFile();
        if(path.toFile().exists()){
            throw new RuntimeException("the program already start");
        }
        Set<PosixFilePermission> perms = PosixFilePermissions.fromString(PERMISSION);
        Files.createFile(path, PosixFilePermissions.asFileAttribute(perms));
    }

    private static Path getLockFile() {
        return Paths.get(LOCK_PATH, LOCK_FILE);
    }

}
```

### Hook线程应用场景以及注意事项

1. Hook线程只有在收到退出信号的时候会被执行，如果在kill的时候使用了参数9,那么Hook线程不会得到执行，进程将会立刻退出，因此`.lock`文件将得不到清理
2. Hook线程也可以执行一些资源释放的工作，比如关闭文件句炳，socket链接，数据库connection等
3. 尽量不要在Hook线程中执行一些耗时非常长的操作，因为其会导致程序迟迟不能退出
