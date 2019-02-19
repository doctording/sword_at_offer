---
title: "Java8 lambda表达式 与 JVM字节码"
layout: page
date: 2019-02-19 00:00
---

[TOC]

# 匿名内部类 jvm字节码

* InnerClass.java

```java
import java.util.function.Function;
public class InnerClass {
    Function<Object, String> f = new Function<Object, String>() {
        @Override
        public String apply(Object obj) {
            return obj.toString();
        } };
}
```

* `javac InnerClass.java`

生成

```bash
InnerClass$1.class
InnerClass.class
```

* `javap -c -v InnerClass` 查看字节码和常量池

```java
public com.other.InnerClass();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=4, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: new           #2                  // class com/other/InnerClass$1
         8: dup
         9: aload_0
        10: invokespecial #3                  // Method com/other/InnerClass$1."<init>":(Lcom/other/InnerClass;)V
        13: putfield      #4                  // Field f:Ljava/util/function/Function;
        16: return
      LineNumberTable:
        line 4: 0
        line 5: 4
}
```

* 通过字节码操作`new`，一个InnerClass$1类型的对象被实例化了。与此同时，一个指向新创建对象的引用会被压入栈。

* `dup`操作会复制栈上的引用。

* 接着，这个值会被`invokespecial`指令处理，该指令会初始化对象。

* 栈顶现在包含了指向对象的引用，该值通过`putfield`指令保存到了LambdaBytecode类的f1字段

# lambda 字节码

```java
import java.util.function.Function;

public class Lambda {
    Function<Object, String> f = obj -> obj.toString();
}
```

```java
 public com.other.Lambda();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: invokedynamic #2,  0              // InvokeDynamic #0:apply:()Ljava/util/function/Function;
        10: putfield      #3                  // Field f:Ljava/util/function/Function;
        13: return
      LineNumberTable:
        line 5: 0
        line 6: 4
}
```

创建额外的类现在被`invokedynamic`指令替代了

## invokedynamic 指令

字节码指令`invokedynamic`最初被JDK7引入，用于支持运行于JVM上的动态类型语言。执行方法调用时，`invokedynamic`添加了更高层的抽象，使得一部分逻辑可以依据动态语言的特征来决定调用目标。这一指令的典型使用场景如下:
`def add(a, b) { a + b }`
这里a和b的类型在编译时都未知，有可能随着运行时发生变化。由于这个原因，JVM首次执行`invokedynamic`调用时，它会查询一个`bootstrap`方法，该方法实现了依赖语言的逻辑，可以决定选择哪一个方法进行调用。bootstrap方法返回一个链接调用点(linked call site)。很多情况下，如果add方法使用两个int类型的变量，紧接下来的调用也会使用两个int类型的值。所以，每次调用也没有必要都重新选择调用的方法。调用点自身就包含了一定的逻辑，可以判断在什么情况下需要进行重新链接。

`invokedynamic`指令将实现Lambda表达式的这部分代码的字节码生成推迟到运行时

* Lambda表达式的代码块到字节码的转换由高层的策略变成了存粹的实现细节。它现在可以动态地改变，或者在未来版本中得到优化、修改，并且保持了字节码的后向兼容性

* 没有带来额外的开销，没有额外的字段，也不需要进行静态初始化，而这些如果不使用 Lambda，就不会实现。

* 对无状态非捕获型Lambda，我们可以创建一个Lambda对象的实例，对其进行缓存，之后 对同一对象的访问都返回同样的内容。这是一种常见的用例，也是人们在Java 8之前就惯用的方式;比如，以static final变量的方式声明某个比较器实例。

* 没有额外的性能开销，因为这些转换都是必须的，并且结果也进行了链接，仅在Lambda 首次被调用时需要转换。其后所有的调用都能直接跳过这一步，直接调用之前链接的实现

将Lambda表达式的代码体填入到运行时动态创建的静态方法, 类似如下

```java
public class Lambda {
    Function<Object, String> f = [dynamic invocation of lambda$1]
    static String lambda$1(Object obj) {
        return obj.toString();
} }
```