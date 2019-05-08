---
title: "Java Runtime"
layout: page
date: 2019-02-25 00:00
---

[TOC]

# Runtime类

## 内存相关的方法

```java
/**
 * Every Java application has a single instance of class
 * <code>Runtime</code> that allows the application to interface with
 * the environment in which the application is running. The current
 * runtime can be obtained from the <code>getRuntime</code> method.
 * <p>
 * An application cannot create its own instance of this class.
 *
 * @author  unascribed
 * @see     java.lang.Runtime#getRuntime()
 * @since   JDK1.0
 */

public class Runtime {
```

* totalMemory

```java
/**
    * Returns the total amount of memory in the Java virtual machine.
    * The value returned by this method may vary over time, depending on
    * the host environment.
    * <p>
    * Note that the amount of memory required to hold an object of any
    * given type may be implementation-dependent.
    *
    * @return  the total amount of memory currently available for current
    *          and future objects, measured in bytes.
    */
    public native long totalMemory();
```

* freeMemory

```java
/**
     * Returns the amount of free memory in the Java Virtual Machine.
     * Calling the
     * <code>gc</code> method may result in increasing the value returned
     * by <code>freeMemory.</code>
     *
     * @return  an approximation to the total amount of memory currently
     *          available for future allocated objects, measured in bytes.
     */
    public native long freeMemory();
```

* maxMemory

```java
 /**
     * Returns the maximum amount of memory that the Java virtual machine will
     * attempt to use.  If there is no inherent limit then the value {@link
     * java.lang.Long#MAX_VALUE} will be returned.
     *
     * @return  the maximum amount of memory that the virtual machine will
     *          attempt to use, measured in bytes
     * @since 1.4
     */
    public native long maxMemory();
```

* usedMemory

```java
totalMemory - freeMemory;
```

* 测试程序 和 jvm参数设置

```java
// -Xms20M -Xmx20M -Xss512K -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=2 -XX:ConcGCThreads=1 -XX:InitiatingHeapOccupancyPercent=70 -XX:-OmitStackTraceInFastThrow -XX:MaxMetaspaceSize=500m -XX:+PrintGCDetails
    public static final int _1MB = 1024 * 1024;

    public static void main(String[] args){
        Runtime r = Runtime.getRuntime();
        System.out.println("=======init");
        System.out.println("freeMemory:" + r.freeMemory());
        System.out.println("maxMemory:" + r.maxMemory());
        System.out.println("totalMemory:" + r.totalMemory());

        byte[] alloc1, alloc2, alloc3, alloc4;
        alloc1 = new byte[3 * _1MB];
        alloc2 = new byte[3 * _1MB];
        alloc3 = new byte[3 * _1MB];
//        alloc4 = new byte[3 * _1MB];
//        System.out.println(alloc4.length);

        System.out.println("=======used 9M");
        System.out.println("freeMemory:" + r.freeMemory());
        System.out.println("maxMemory:" + r.maxMemory());
        System.out.println("totalMemory:" + r.totalMemory());

        System.out.println("main end");
    }

```

## addShutdownHook, jvm shutdown的钩子函数

```java
public void addShutdownHook(Thread hook) {
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        sm.checkPermission(new RuntimePermission("shutdownHooks"));
    }
    ApplicationShutdownHooks.add(hook);
}
```

## availableProcessors，可用的cpu核心数

```java
/**
* Returns the number of processors available to the Java virtual machine.
*
* <p> This value may change during a particular invocation of the virtual
* machine.  Applications that are sensitive to the number of available
* processors should therefore occasionally poll this property and adjust
* their resource usage appropriately. </p>
*
* @return  the maximum number of processors available to the virtual
*          machine; never smaller than one
* @since 1.4
*/
public native int availableProcessors();
```