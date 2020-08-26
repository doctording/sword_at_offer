---
title: "Java异常&错误"
layout: page
date: 2019-06-07 00:00
---

[TOC]

# Java Throwable

```bash

            Throwable
          /          \
         /            \
    Error            Exception
                   /            \
                  /              \
                checked           unchkecked
                  |                    |
                  |                    |
                IOException      RuntimeException
```

## checked exception

编译器要求必须处置的异常，正确的程序在运行中，很容易出现的、情理可容的异常状况。可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常状况，就必须采取某种方式进行处理。

除了`RuntimeException`及其子类以外，其它的Exception类及其子类都属于`checked`异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则**编译**不会通过。如SQLException , IOException,ClassNotFoundException 等。

## unchecked exceptions

javac在编译时，不会提示和发现这样的异常，不要求在程序处理这些异常。所以如果愿意，我们可以编写代码处理（使用try…catch…finally）这样的异常，也可以不处理。对于这些异常，我们应该修正代码，而不是去通过异常处理器处理 。这样的异常发生的原因多半是代码写的有问题。如除0错误ArithmeticException，错误的强制类型转换错误ClassCastException，数组索引越界ArrayIndexOutOfBoundsException，使用了空对象NullPointerException等等。

常见的RuntimeException

1. java.lang.NullPointerException;
2. java.lang.ArithmaticException;
3. java.lang.ArrayIndexoutofBoundsException
4. StringIndexOutOfBoundsException
5. ClassCastException
6. IllegalArgumentException

## finally块和return

### finally 一定会执行, try中也return了

```java
class Main {

    public static void main(String[] args) {
        int re = bar();
        System.out.println(re);
    }

    private static int bar() {
        try {
            return 5;
        } finally {
            System.out.println("finally");
        }
    }
}
/*输出：
finally
5
*/
```

### finally中的return会抑制（消灭）前面try或者catch块中的异常或return

```java
class Main {

    public static void main(String[] args) {
        int re = bar();
        System.out.println(re);
    }

    private static int bar() {
        try {
            return 5;
        } finally {
            System.out.println("finally");
            return 15;
        }
    }
}
/*输出：
finally
15
*/
```

### finally throw 异常

```java

class Main {

    public static void main(String[] args) {
        int result = -1;
        try{
            result = foo();
        } catch (Exception e){
            System.out.println(e.getMessage());    //输出：我是finaly中的Exception
        }finally {
            System.out.println(result); // -1
        }

        try{
            result  = bar();
        } catch (Exception e){
            System.out.println(e.getMessage());    //输出：我是finaly中的Exception
        }finally {
            System.out.println(result); // -1
        }
    }
    //catch中的异常被抑制
    @SuppressWarnings("finally")
    public static int foo() throws Exception {
        try {
            int a = 5/0;
            return 1;
        }catch(ArithmeticException amExp) {
            throw new Exception("foo 我将被忽略，因为下面的finally中抛出了新的异常");
        }finally {
            throw new Exception("foo 我是finaly中的Exception");
        }
    }

    //try中的异常被抑制
    @SuppressWarnings("finally")
    public static int bar() throws Exception {
        try {
            int a = 5/0;
            return 1;
        }finally {
            throw new Exception("bar 我是finaly中的Exception");
        }

    }
}
/*输出：
foo 我是finaly中的Exception
-1
bar 我是finaly中的Exception
-1
*/
```

### 建议

* 不要在fianlly中使用return。
* 不要在finally中抛出异常。
* 减轻finally的任务，不要在finally中做一些其它的事情，finally块仅仅用来释放资源是最合适的。
* 将尽量将所有的return写在函数的最后面，而不是`try … catch … finally`中

## 继承 Exception, IOException

```java
public
class IOException extends Exception {
    static final long serialVersionUID = 7818375828146090155L;

    /**
     * Constructs an {@code IOException} with {@code null}
     * as its error detail message.
     */
    public IOException() {
        super();
    }

    /**
     * Constructs an {@code IOException} with the specified detail message.
     *
     * @param message
     *        The detail message (which is saved for later retrieval
     *        by the {@link #getMessage()} method)
     */
    public IOException(String message) {
        super(message);
    }

    /**
     * Constructs an {@code IOException} with the specified detail message
     * and cause.
     *
     * <p> Note that the detail message associated with {@code cause} is
     * <i>not</i> automatically incorporated into this exception's detail
     * message.
     *
     * @param message
     *        The detail message (which is saved for later retrieval
     *        by the {@link #getMessage()} method)
     *
     * @param cause
     *        The cause (which is saved for later retrieval by the
     *        {@link #getCause()} method).  (A null value is permitted,
     *        and indicates that the cause is nonexistent or unknown.)
     *
     * @since 1.6
     */
    public IOException(String message, Throwable cause) {
        super(message, cause);
    }

    /**
     * Constructs an {@code IOException} with the specified cause and a
     * detail message of {@code (cause==null ? null : cause.toString())}
     * (which typically contains the class and detail message of {@code cause}).
     * This constructor is useful for IO exceptions that are little more
     * than wrappers for other throwables.
     *
     * @param cause
     *        The cause (which is saved for later retrieval by the
     *        {@link #getCause()} method).  (A null value is permitted,
     *        and indicates that the cause is nonexistent or unknown.)
     *
     * @since 1.6
     */
    public IOException(Throwable cause) {
        super(cause);
    }
}

```
