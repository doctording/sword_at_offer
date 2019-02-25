---
title: "Java Runtime"
layout: page
date: 2019-02-25 00:00
---

[TOC]

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
