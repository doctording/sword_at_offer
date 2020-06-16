---
title: "反射机制"
layout: page
date: 2020-06-14 00:00
---

[TOC]

# 反射

Java的反射（`reflection`）机制是指在程序的运行状态中，可以构造任意一个类的对象，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用任意一个对象的属性和方法。这种动态获取程序信息以及动态调用对象的功能称为Java语言的反射机制。

对于一个字节码文件`.class`，虽然表面上我们对该字节码文件一无所知，但该文件本身却记录了许多信息。Java在将.class字节码文件载入时，JVM将产生一个`java.lang.Class`对象代表该.class字节码文件，从该Class对象中可以获得类的许多基本信息，这就是反射机制。

## 获得Class对象的方式

* Class.forName(“类的全限定名”)
* 实例对象.getClass()
* 类名.class （类字面常量）

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

## 例子(仿Spring)

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
* 初始化：激活类的静态变量的初始化Java代码和静态Java代码块，并初始化程序员设置的变量值。

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

将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块(静态成员初始化，静态代码块)

### ClassLoader.loadClass

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

第2个boolean参数，表示目标对象是否进行链接，false表示不进行链接，不进行链接意味着不进行包括初始化等一些列步骤，那么静态块和静态成员就不会得到执行
