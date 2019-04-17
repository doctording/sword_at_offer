---
title: "注解 annotation"
layout: page
date: 2019-04-14 00:00
---

[TOC]

# 注解

```java
@interface TestAnnotation{

}

@TestAnnotation
class Test{

}
```

理解为 Test 具有了 TestAnnotation 标签

* 参考1：
作者：frank909
来源：CSDN
原文：https://blog.csdn.net/briblue/article/details/73824058 
版权声明：本文为博主原创文章，转载请附上博文链接！

## 元注解

* @Retention

* RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
* RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
* RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。


* @Documented
* @Target

ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
ElementType.CONSTRUCTOR 可以给构造方法进行注解
ElementType.FIELD 可以给属性进行注解
ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
ElementType.METHOD 可以给方法进行注解
ElementType.PACKAGE 可以给一个包进行注解
ElementType.PARAMETER 可以给一个方法内的参数进行注解
ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举

* @Inherited
* @Repeatable


```java
@interface Persons {
    Person[] value();
}


@Repeatable(Persons.class)
@interface Person{
    String role() default "";
}

@Person(role="artist")
@Person(role="coder")
@Person(role="PM")
class SuperMan{

}
```

## 注解的用途

* 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息
* 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。
* 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取

当开发者使用了Annotation 修饰了类、方法、Field 等成员之后，这些 Annotation 不会自己生效，必须由开发者提供相应的代码来提取并处理 Annotation 信息。这些处理提取和处理 Annotation 的代码统称为 APT（Annotation Processing Tool)。
