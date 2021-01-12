---
title: "Java SPI机制"
layout: page
date: 2020-08-30 00:00
---

[TOC]

# SPI

![](../../content/java_utils/imgs/spi.png)

## 回顾双亲委派模型

```java
Bootstrap class loader
    Extension class loader
        System class loader
            ClassLoader A
```

双亲委派中下层的类加载器可以使用上层父加载器加载的对象，但是上层父类的加载器不可以使用子类的加载器加载的对象）

比如上方`System class loader`想要使用`ClassLoader A`则是做不到的

```java
public class MainClass {
    public static void main(String[] args) {
        System.out.println(Integer.class.getClassLoader());
        System.out.println(Logging.class.getClassLoader());
        System.out.println(MainClass.class.getClassLoader());
    }
}
/*
null # Bootstrap类加载器
sun.misc.Launcher$ExtClassLoader@5e2de80c # Extension类加载器
sun.misc.Launcher$AppClassLoader@18b4aac2 # System类加载器
*/
```

## 线程上下文加载器

* 通过java.lang.Thread类的setContextClassLoader()设置当前线程的上下文类加载器（如果没有设置，默认会从父线程中继承，如果程序没有设置过，则默认是System类加载器）

* 有了线程上下文类加载器，应用程序就可以通过java.lang.Thread.setContextClassLoader()将应用程序使用的类加载器传递给使用更顶层类加载器的代码。

<font color='red'>可以打破双亲委派机制，实现逆向调用类加载器来加载当前线程中类加载器加载不到的类</font>

## SPI实现原理

1. 读取`META-INF/services/`下的配置文件，获得所有能被实例化的类的名称(ServiceLoader可以跨越jar包获取META-INF下的配置文件，具体加载配置的实现代码如下)

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}

private ServiceLoader(Class<S> svc, ClassLoader cl) {
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}
```

* java.util.ServiceLoader.LazyIterator

```java
try {
    String fullName = PREFIX + service.getName();
    if (loader == null)
        configs = ClassLoader.getSystemResources(fullName);
    else
        configs = loader.getResources(fullName);
} catch (IOException x) {
    fail(service, "Error locating configuration files", x);
}
```

2. 通过反射方法`Class.forName()`加载类对象，并将类实例化

```java
class SystemClassLoaderAction
    implements PrivilegedExceptionAction<ClassLoader> {
    private ClassLoader parent;

    SystemClassLoaderAction(ClassLoader parent) {
        this.parent = parent;
    }

    public ClassLoader run() throws Exception {
        String cls = System.getProperty("java.system.class.loader");
        if (cls == null) {
            return parent;
        }

        Constructor<?> ctor = Class.forName(cls, true, parent)
            .getDeclaredConstructor(new Class<?>[] { ClassLoader.class });
        ClassLoader sys = (ClassLoader) ctor.newInstance(
            new Object[] { parent });
        Thread.currentThread().setContextClassLoader(sys);
        return sys;
    }
}
```

3. 把实例化后的类缓存到providers对象中，(LinkedHashMap<String,S>类型）然后返回实例对象。

## Java SPI缺点

1. 不能按需加载。虽然 ServiceLoader 做了延迟载入，但是基本只能通过遍历全部获取，也就是接口的实现类得全部载入并实例化一遍。如果你并不想用某些实现类，或者某些类实例化很耗时，它也被载入并实例化了，这就造成了浪费。
2. 获取某个实现类的方式不够灵活，只能通过 Iterator 形式获取，不能根据某个参数来获取对应的实现类。
3. 多个并发多线程使用 ServiceLoader 类的实例是不安全的。
4. 加载不到实现类时抛出并不是真正原因的异常，错误很难定位。
