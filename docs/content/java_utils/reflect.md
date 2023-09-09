---
title: "反射机制"
layout: page
date: 2020-06-14 00:00
---

[TOC]

# 反射

Java的反射（`reflection`）机制是指在程序的运行状态中，可以构造任意一个类的对象(可以实例化该对象出来)，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用任意一个对象的属性和方法。这种动态获取程序信息以及动态调用对象的功能称为Java语言的反射机制。

对于一个字节码文件`.class`，虽然表面上我们对该字节码文件一无所知，但该文件本身却记录了许多信息。Java在将`.class`字节码文件载入时，JVM将产生一个`java.lang.Class`对象代表该`.class`字节码文件，从该Class对象中可以获得类的许多基本信息，这就是反射机制。

## 反射机制的功能

* 在运行时判断任意一个对象所属的类。
* 在运行时实例化任意一个类的对象。
* 在运行时判断任意一个类所具有的成员变量和方法，包括私有变量，也可以设置访问属性。
* 在运行时调用任意一个对象的任意一个方法。

一个类中包含的信息有： 构造器，字段，方法

* Class ： 描述类
* Method ： 描述方法
* Constructor ：描述构造器
* Field ：描述字段

## 获得`Class对象`的方式

* Class.forName("类的全限定名")
* 实例对象.getClass()
* 类名.class（类的字面常量）

### 获得`Class对象`的例子代码

```java
static void test() throws Exception{
    Class clazz = Class.forName("reflect.PayService");
    Class clazz2 = PayService.class;

    PayService payService = new AliPayServiceImpl();
    Class clazz3 = payService.getClass();

    // interface reflect.PayService
    System.out.println(clazz);
    // interface reflect.PayService
    System.out.println(clazz2);
    // class reflect.AliPayServiceImpl
    System.out.println(clazz3);
    // class java.lang.Object
    Class clazz4 = clazz3.getSuperclass();
    System.out.println(clazz4);
    // true
    System.out.println(clazz == clazz2);
    // false
    System.out.println(clazz2 == clazz3);
}
```

* PayService

```java
package reflect;

/**
 * @Author mubi
 * @Date 2020/6/16 06:44
 */
public interface PayService {
    String payUse();
}
```

* AliPayServiceImpl

```java
package reflect;

/**
 * @Author mubi
 * @Date 2020/6/16 06:43
 */
public class AliPayServiceImpl implements PayService {

    @Override
    public String payUse() {
        System.out.println("in method payUse");
        return "AliPayServiceImpl";
    }
}
```

## Class 对象

```java
/**
 * Instances of the class {@code Class} represent classes and
 * interfaces in a running Java application.  An enum is a kind of
 * class and an annotation is a kind of interface.  Every array also
 * belongs to a class that is reflected as a {@code Class} object
 * that is shared by all arrays with the same element type and number
 * of dimensions.  The primitive Java types ({@code boolean},
 * {@code byte}, {@code char}, {@code short},
 * {@code int}, {@code long}, {@code float}, and
 * {@code double}), and the keyword {@code void} are also
 * represented as {@code Class} objects.
 *
 * <p> {@code Class} has no public constructor. Instead {@code Class}
 * objects are constructed automatically by the Java Virtual Machine as classes
 * are loaded and by calls to the {@code defineClass} method in the class
 * loader.
 *
 * <p> The following example uses a {@code Class} object to print the
 * class name of an object:
 *
 * <blockquote><pre>
 *     void printClassName(Object obj) {
 *         System.out.println("The class of " + obj +
 *                            " is " + obj.getClass().getName());
 *     }
 * </pre></blockquote>
 *
 * <p> It is also possible to get the {@code Class} object for a named
 * type (or for void) using a class literal.  See Section 15.8.2 of
 * <cite>The Java&trade; Language Specification</cite>.
 * For example:
 *
 * <blockquote>
 *     {@code System.out.println("The name of class Foo is: "+Foo.class.getName());}
 * </blockquote>
 *
 * @param <T> the type of the class modeled by this {@code Class}
 * object.  For example, the type of {@code String.class} is {@code
 * Class<String>}.  Use {@code Class<?>} if the class being modeled is
 * unknown.
 *
 * @author  unascribed
 * @see     java.lang.ClassLoader#defineClass(byte[], int, int)
 * @since   JDK1.0
 */
public final class Class<T> implements java.io.Serializable,
                              GenericDeclaration,
                              Type,
                              AnnotatedElement {
```

## 反射例子(仿Spring)

* UserController

```java
package reflect;

/**
 * @Author mubi
 * @Date 2020/6/16 06:43
 */
public class UserController {

    @Autoware
    private UserService userService;

    public UserService getUserService() {
        return userService;
    }
}
```

* UserService

```java
package reflect;

/**
 * @Author mubi
 * @Date 2020/6/16 06:44
 */
public class UserService {

}
```

* Autoware

```java
package reflect;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD})
@Documented
public @interface Autoware {
}
```

* 测试程序

```java
package reflect;

import java.util.Arrays;

/**
 * @Author mubi
 * @Date 2020/6/16 06:44
 */
public class ClientTest {

    public static void main(String[] args) {
        UserController userController = new UserController();

        Arrays.stream(userController.getClass().getDeclaredFields()).forEach(field -> {
            // 是否有 @Autoware
            Autoware autoware = field.getAnnotation(Autoware.class);
            if (autoware != null) {
                field.setAccessible(true);
                // 获取 filed 的 类型
                Class clazzType = field.getType();
                try {
                    // new filed 并注入给 userController
                    Object o = clazzType.newInstance();
                    field.set(userController, o);
                } catch (Exception e){

                }
            }
        });

        // userService:reflect.UserService@2aafb23c
        System.out.println("userService:" + userController.getUserService());
    }
}
```

userController的`userService`属性不是自己new出来，而是通过反射注入

## `Class.forName()`和`ClassLoader.loadClass()`的区别

* 加载：通过类的全限定名获取二进制字节流，将二进制字节流转换成方法区中的运行时数据结构，在内存中生成`Java.lang.class`对象
* 链接：执行下面的校验、准备和解析步骤，其中解析步骤是可以选择的；
    1. 校验：检查导入类或接口的二进制数据的正确性；（文件格式验证，元数据验证，字节码验证，符号引用验证）
    2. 准备：给类的静态变量分配并初始化存储空间；
    3. 解析：将常量池中的符号引用转成直接引用；
* 初始化：激活类的静态变量的初始化代码和静态码块，并初始化成设置的变量值。

### Class.forName()

```java
/**
    * Returns the {@code Class} object associated with the class or
    * interface with the given string name.  Invoking this method is
    * equivalent to:
    *
    * <blockquote>
    *  {@code Class.forName(className, true, currentLoader)}
    * </blockquote>
    *
    * where {@code currentLoader} denotes the defining class loader of
    * the current class.
    *
    * <p> For example, the following code fragment returns the
    * runtime {@code Class} descriptor for the class named
    * {@code java.lang.Thread}:
    *
    * <blockquote>
    *   {@code Class t = Class.forName("java.lang.Thread")}
    * </blockquote>
    * <p>
    * A call to {@code forName("X")} causes the class named
    * {@code X} to be initialized.
    *
    * @param      className   the fully qualified name of the desired class.
    * @return     the {@code Class} object for the class with the
    *             specified name.
    * @exception LinkageError if the linkage fails
    * @exception ExceptionInInitializerError if the initialization provoked
    *            by this method fails
    * @exception ClassNotFoundException if the class cannot be located
    */
@CallerSensitive
public static Class<?> forName(String className)
            throws ClassNotFoundException {
    Class<?> caller = Reflection.getCallerClass();
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
}
```

将类的`.class`文件加载到jvm中之外，还会对类进行**解释**，即会执行类中的`static`块(静态成员初始化，静态代码块)

### ClassLoader.loadClass(仅加载生成class对象,不进行链接)

```java
/**
    * Loads the class with the specified <a href="#name">binary name</a>.
    * This method searches for classes in the same manner as the {@link
    * #loadClass(String, boolean)} method.  It is invoked by the Java virtual
    * machine to resolve class references.  Invoking this method is equivalent
    * to invoking {@link #loadClass(String, boolean) <tt>loadClass(name,
    * false)</tt>}.
    *
    * @param  name
    *         The <a href="#name">binary name</a> of the class
    *
    * @return  The resulting <tt>Class</tt> object
    *
    * @throws  ClassNotFoundException
    *          If the class was not found
    */
public Class<?> loadClass(String name) throws ClassNotFoundException {
    return loadClass(name, false);
}
```

第2个boolean参数，表示目标对象是否进行链接，false表示<font color='red'>不进行链接</font>，不进行链接意味着就不会进行包括初始化等的一系列步骤，那么静态块和静态成员就不会得到执行

## 反射机制会不会有性能问题？

Oracle官方文档：<a href='https://docs.oracle.com/javase/tutorial/reflect/index.html'>Drawbacks of Reflection
</a>

Reflection is powerful, but should not be used indiscriminately. If it is possible to perform an operation without using reflection, then it is preferable to avoid using it. The following concerns should be kept in mind when accessing code via reflection.

尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心：

* Performance Overhead（性能开销）

Because reflection involves types that are dynamically resolved, certain Java virtual machine optimizations can not be performed. Consequently, reflective operations have slower performance than their non-reflective counterparts, and should be avoided in sections of code which are called frequently in performance-sensitive applications.

反射包括了一些要动态解析的类型，所以JVM无法对这些代码进行优化；因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。

* Security Restrictions（安全限制）

Reflection requires a runtime permission which may not be present when running under a security manager. This is in an important consideration for code which has to run in a restricted security context, such as in an Applet.

使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如Applet，那么这就是个问题了。

* Exposure of Internals（内部暴露）

Since reflection allows code to perform operations that would be illegal in non-reflective code, such as accessing private fields and methods, the use of reflection can result in unexpected side-effects, which may render code dysfunctional and may destroy portability. Reflective code breaks abstractions and therefore may change behavior with upgrades of the platform.

由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用：代码有功能上的错误，降低可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。

### 反射实例化和普通new的效率对比

```java
class A {
    public void doSomeThing(){

    }
}

public class Main {

    static int N = 1_000_000;

    public static void main(String[] args) throws Exception {
        doRegular(); // 打印8、6 几毫秒
        doReflection(); // 打印421、391，几百毫秒
    }

    public static void doRegular() {
        long start = System.currentTimeMillis();
        for (int i = 0; i < N; i++) {
            A a = new A();
            a.doSomeThing();
        }
        System.out.println(System.currentTimeMillis() - start);
    }

    public static void doReflection() throws Exception {
        long start = System.currentTimeMillis();
        for (int i = 0; i < N; i++) {
            A a = (A) Class.forName("A").newInstance();
            a.doSomeThing();
        }
        System.out.println(System.currentTimeMillis() - start);
    }

}
```

### 反射:私有属性设置

```java

import java.lang.reflect.Field;

class A{
    public int a;
    private String b;

    public A(int a, String b) {
        this.a = a;
        this.b = b;
    }

    @Override
    public String toString() {
        return "A{" +
                "a=" + a +
                ", b='" + b + '\'' +
                '}';
    }
}

public class ReflectTest {

    public static void main(String[] args) throws Exception{
        A a = new A(1, "aaa");
        System.out.println(a);
        a.a = 2;
        System.out.println(a);

        // 利用反射设置属性
        Field field = a.getClass().getDeclaredField("b");   // 获取私有属性
        field.setAccessible(true);  // 设置属性可访问
        field.set(a, "bbb");    // 设置属性的值

        System.out.println(a);
    }

}
/*
A{a=1, b='aaa'}
A{a=2, b='aaa'}
A{a=2, b='bbb'}
 */
```

### 反射：通过构造器实例化对象

```java
 public static void main(String[] args) throws Exception{
    Class c = Class.forName("com.A",false, ClassLoader.getSystemClassLoader());
    Constructor<?>[] constructors = c.getConstructors();
    Constructor constructor = constructors[0];
    A a = (A)constructor.newInstance(1, "aaa");
    System.out.println(a);
}
```

```java
public static void main(String[] args) throws Exception{
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    Class c = Class.forName("com.A",false, cl);
    Constructor<?>[] constructors = c.getConstructors();
    Constructor constructor = constructors[0];
    A a = (A)constructor.newInstance(1, "aaa");
    System.out.println(a);
}
```
