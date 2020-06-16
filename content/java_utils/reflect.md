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
