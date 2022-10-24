---
title: "jvm,类加载机制,类加载器,内存布局"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# JVM规范

JVM组成：
* 类加载子系统（`.class -> loading -> linking -> Initailization`）
* 运行时数据区(方法区，堆，thread stack, native statck, pc registor)
* 执行引擎(interpreter, jit compiler, garbage collector; native method interface/Library)

<a href='https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html'>jvm oracale官方文档</a>

参考图：<a href='https://www.processon.com/view/5f4254f163768956ffb78006?fromnew=1'>https://www.processon.com/view/5f4254f163768956ffb78006?fromnew=1</a>

# `.Class`字节码文件分析

Class文件是一组以8位字节为基础单位的`二进制流`，任何一个Class文件都对应唯一一个类或接口的定义信息

参考：![Java字节码增强探秘](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA%3D%3D&chksm=bd1259af8a65d0b97809a6a8ff5afaff1be4a4232bd8527ef9d95bb7a2e768bd7d9fdc768211&idx=1&mid=2651750626&scene=21&sn=3e1ac6c41d6e1803abb32285daf0244a#wechat_redirect)

![](../../content/java_jvm/imgs/java_byte_enhance.webp)

## Class 对象

每一个类都有一个Class对象，每当加载一个新类就产生一个Class对象，基本类型 (boolean, byte, char, short, int, long, float, double)有Class对象，数组有Class对象，就连关键字void也有Class对象（void.class）。Class对象对应着`java.lang.Class`类，如果说类是对象抽象的集合的话，那么Class类就是对类的抽象的集合。Class类没有公共的构造方法，Class对象是在类加载的时候由**Java虚拟机**以及通过调用类加载器中的`defineClass`方法自动构造的，因此不能显式地声明一个Class对象

![](../../content/java_jvm/imgs/java_class.png)

## 魔数与Class文件的版本

## 常量池

主要存放两大类常量：字面量（Literal）和 符号引用（Symbolic Reference）

1. 字面量：如文本字符串，声明为final的常量值

2. 符号引用：类和接口的全限定名，字段的名称和描述符，方法的名称和描述符

Class文件中不会保存各个方法、字段的最终内存布局信息；因此这些字段，方法的符号引用不经过运行期转换的话，无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析，翻译到具体的内存地址之中。

## 访问标志

用于识别一些类或者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract类型；如果是类的话，是否被声明为final等。

## 类索引，父类索引，接口索引

## 字段表(field_info)

用来描述接口或者类中声明的变量， 可以包括的信息有：

字段的作用域(public,private,protected修饰符)，是实例变量还是类变量（static修饰符），可变性（final），并发可见性（volatile修饰符,是否强制从主内存读写），可否被序列化（transient修饰符），字段数据类型（基本类型，对象，数组），字段名称。

## 方法表

方法签名：方法名称，参数顺序，参数类型

## 属性表

上述Class文件，字段表，方法表都可以携带自己的属性表集合

# JVM运行时各内存区域

## Java7 运行时内存各区域

![](../../content/java_jvm/imgs/jvm7_heap.png)

![](../../content/java_jvm/imgs/jvm7_run.png)

## Java8 运行时内存各区域

![](../../content/java_jvm/imgs/jvm8_run.png)

## 程序计数器(PC)

* 指向下一条需要执行的字节码；记录当前线程的位置便于线程切换与恢复；
* 唯一 一个不会出现 `OOM` 的区域

## 虚拟机栈（线程stack，计算逻辑）

描述了Java方法执行的内存模型，创建栈帧，保存该本地方法的局部变量表、操作数栈、动态链接、出口信息。

1. Java虚拟机栈是线程私有的，它的生命周期与线程相同（随线程而生，随线程而灭）
2. 栈帧包括局部变量表、操作数栈、动态链接、方法返回地址和一些附加信息
3. 每一个方法被调用直至执行完毕的过程，就对应这一个栈帧在虚拟机栈中从入栈到出栈的过程

![](../../content/java_jvm/imgs/jvm_stack.png)

### 局部变量表(局部变量)

局部变量表（Local Variables）存放了编译器可知的各种Java虚拟机基本数据类型（boolean、byte、char、short、int、float、long、double、reference对象引用类型、returnAddress类型指向一条字节码指令地址）

### 操作数栈（运算中的操作数，暂存空间）

操作数栈（Operand Stack）也常被称为操作栈，是一个后入先出栈，**用于保存计算过程中的中间结果，同时作为计算过程中变量临时的存储空间**

### 动态链接

动态链接：程序运行期间，符号引用转化为直接引用

例如`Object o = new Object()`，堆中对象的指针地址放到【局部变量表】中，这个指针指向了堆中的对象，这种称为<font color='red'>直接引用</font>；运行时常量池，比如类的**全限定名**字符串，这些都是<font color='red'>符号引用</font>

* 符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能够无歧义的定位到目标即可
* 直接引用是和虚拟机的布局相关的，同一个符号引用在不同的虚拟机实例上翻译出来的直接引用一般不会相同；如果有了直接引用，那引用的目标必定已经被加载入内存中了。

每个栈帧都包含一个指向运行时常量池中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态链接（Dynamic Linking）。Class 文件的常量池中存在大量的符号引用，字节码中的方法调用指令就以常量池中指向方法的符号引用作为参数，这些符号引用一部分会在类加载阶段或第一次使用时转化为直接引用，这种转化成为静态解析。另一部分将在每一次运行期间转化为直接引用，这部分称为动态连接。

比如：**描述一个方法调用了另外的其它方法时，就是通过常量池中指向方法的符号引用来表示的，那么动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用**。比如重载方法的调用，`invokevirtual`指令调用方法的符号引用是运行时期转化的

* code

```java
abstract class Human {
    abstract void call();
}

class Father extends Human{
    @Override
    void call() {
        System.out.println("I am the Father!");
    }
}

class Mother extends Human{
    @Override
    void call() {
        System.out.println("I am the Mother!");
    }
}

public class DynamicDispatch {
    public static void main(String[] args) {
        Human father = new Father();
        Human mother = new Mother();
        father.call();
        mother.call();
    }
}
```

* 汇编

```java
public static void main(java.lang.String[]);
    Code:
       0: new           #2                  // class Father
       3: dup
       4: invokespecial #3                  // Method Father."<init>":()V
       7: astore_1
       8: new           #4                  // class Mother
      11: dup
      12: invokespecial #5                  // Method Mother."<init>":()V
      15: astore_2
      16: aload_1
      17: invokevirtual #6                  // Method Human.call:()V
      20: aload_2
      21: invokevirtual #6                  // Method Human.call:()V
      24: return
```

附：Class文件的常量池中存在有大量的符号引用,字节码中的方法调用指令就以指向常量池的引用作为参数
* 部分符号引用在类加载阶段(解析)的时候就转化为直接引用,这种转化为静态链接
* 部分符号引用在运行期间转化为直接引用,这种转化为动态链接

### 方法出口

方法返回地址，执行方法前就保存了；当一个方法开始执行后，只有两种方式可以退出这个方法

* 一种是执行引擎遇到任意一个方法返回的字节码指令，这时候可能会有返回值传递给上层方法的调用者，是否有返回值和返回值的类型将根据遇到何种方法返回指令来决定，这种退出方法的方式称为正常完成出口

* 另一种退出方式是，在方法执行过程中遇到了异常，并且这个异常没有在方法体内得到处理，无论是 Java 虚拟机内部产生的异常，还是代码中使用 athrow 字节码指令产生的异常，只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出。这种称为异常完成出口，一个方法使用异常完成出口的方式退出，是不会给上层调用者产生任何返回值的。

方法退出的过程实际上就等同于把当前栈帧出栈，因此退出时可能执行的操作有：恢复上次方法的局部变量表和操作数栈，把返回值（如果有的话）压入调用者栈帧的操作数栈中，调整 PC 计数器的值以指向方法调用指令后面的一条指令等。

### 什么情况下会发生栈内存溢出（面试题）？

* 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出`StackOverflowError`异常，方法**递归**调用产生这种结果。

* 如果Java虚拟机栈可以动态扩展，并且扩展的动作已经尝试过，但是无法申请到足够的内存去完成扩展，或者在新建立线程的时候没有足够的内存去创建对应的虚拟机栈(**虚拟机栈是每个线程必须要有的，完成计算逻辑**)，那么Java虚拟机将抛出一个`OutOfMemory`异常。(**线程启动过多**)

* 参数`-Xss`去调整JVM栈的大小

## 本地方法栈(native method stack)

描述`native`方法执行，会创建栈帧（本地方法栈）:也保存了该本地方法的局部变量表、操作数栈、动态链接、出口信息。

native会调用本地方法库的本地方法接口（JNI：Java Native Interface）

能允许JAVA程序调用C/C++写的程序，扩展性功能

## 堆(heap)（存储功能）

主要用于存放`对象`；Java8之前有【方法区】的大部分被移到堆中了，所以，堆中还放有：`运行时常量池`,`字符串常量池`

### JVM里`new`对象时，堆会发生抢占吗？怎么去设计JVM的堆的线程安全?

会，假设JVM虚拟机上，每一次new 对象时，指针就会向右移动一个对象size的距离，一个线程正在给A对象分配内存，指针还没有来的及修改，另一个为B对象分配内存的线程，引用这之前的指针指向，这就发生了抢占，也被称为指针碰撞。

解决方案：

1. 对分配内存空间的动作做同步处理，采用CAS机制，配合失败重试的方式保证更新操作的原子性。（低效率）
2. 每个线程在Java堆中预先分配一小块内存，然后再给对象分配内存的时候，直接在自己这块"私有"内存中分配，当这部分区域用完之后，再分配新的"私有"内存。

#### Thread Local Allocation Buffer，线程本地分配缓存

JVM在内存新生代Eden Space中开辟了一小块线程私有的区域TLAB（Thread-local allocation buffer），TLAB也仅作用于新生代的Eden Space。在Java程序中很多对象都是小对象且用过即丢，它们不存在线程共享也适合被快速GC，所以对于小对象通常JVM会优先分配在TLAB上，并且TLAB上的分配由于是线程私有，所以没有锁开销。也就是说，Java中每个线程都会有自己的缓冲区称作TLAB，在对象分配的时候不用锁住整个堆，而只需要在自己的缓冲区分配即可。

## 方法区

静态变量，常量，类信息（构造方法，接口定义），运行时的常量池存在方法区中，但是实例变量存在堆内存中，和方法区无关

static,final,Class,常量池

## 元空间(metaspace)（不属于JVM，使用堆外内存）

类的元数据：如方法、字段、类、包的描述信息，这些信息可以用于创建文档、跟踪代码中的依赖性、执行编译时检查

线程栈中要new对象，从元空间能获取到class信息，然后在堆中分配内存，并与栈中变量建立引用关系

**Metaspace**由两大部分组成：Klass Metaspace和NoKlass Metaspace。

1. klass Metaspace就是用来存klass的，就是class文件在jvm里的运行时数据结构，是一块连续的内存区域，紧接着Heap
2. NoKlass Metaspace专门来存klass相关的其它的内容，比如method，constantPool等，可以由多块不连续的内存组成

### 为什么去掉永久代，元空间用非堆的机器内存？

永久代是方法区的实现，使用堆内存，不好分配大小。

gc问题，OOM，应用越来越大(调优不友好)

* JVM加载的Class的总数，方法的大小等都很难确定，因此对永久代大小的指定难以确定。太小的永久代容易导致永久代内存溢出，太大的永久代则容易导致虚拟机内存紧张。
* ASM，Cglib动态生成，也导致永久代大小的指定难以确定

### 元空间调优？

元空间要限制，机器内存OOM问题

### Class实例究竟在Method Area还是在Heap中

![](../../content/java_jvm/imgs/method_area.png)

Java8中`O.class`在堆中，其中`C++对象`在方法区中

## 常量池

大体可以分为：静态常量池，运行时常量池。

* 静态常量池

静态常量池，即`*.class`文件中的常量池，class文件中的常量池不仅仅包含字符串(数字)字面量，还包含类、方法的信息，占用class文件绝大部分空间。这种常量池主要用于存放两大类常量：字面量(Literal)和符号引用量(Symbolic References)

1. 类和接口的全限定名
2. 字段名称和描述符
3. 方法名称和描述符

* 运行时常量池

运行时常量池，则是jvm虚拟机在完成类装载操作后，将class文件中的常量池载入到内存中，并保存在方法区中，我们常说的常量池，就是指方法区中的运行时常量池。

String的`intern()`方法会查找在常量池中是否存在一份equal相等的字符串,如果有则返回该字符串的引用,如果没有则添加自己的字符串进入常量池。

### 为什么需要常量池？

* 因为在不同的方法，都可能调用相同的常量或者方法，所以只需要存储一份即可，节省了空间
* 常量池的作用：就是为了提供一些符号和常量，便于指令的识别

## Java7的方法区(method area) & 永久代(Perm Space)

在JDK1.6及之前，运行时常量池是方法区的一个部分，同时方法区里面存储了类的元数据信息、静态变量、即时编译器编译后的代码（比如Spring使用IOC或者AOP创建bean时，或者使用cglib，反射的形式动态生成class信息等）等。

在JDK1.7及以后，JVM已经将运行时常量池从方法区中移了出来：部分堆、部分JVM外的元空间。

# HotSpot对象存储

* 对象在内存中存储的布局可以分为3块区域：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）

![](../../content/java_jvm/imgs/hotspot_obj.png)

<a href="http://openjdk.java.net/groups/hotspot/docs/HotSpotGlossary.html" target="_blank">hotspot 相关术语表</a>

术语 | 英文说明 | 中文解释
-|-|-
mark word | The first word of every object header. Usually a set of bitfields including synchronization state and identity hash code. May also be a pointer (with characteristic low bit encoding) to synchronization related information. During GC, may contain GC state bits. | 用于存储对象自身的运行时数据， 如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等等
klass pointer | The second word of every object header. Points to another object (a metaobject) which describes the layout and behavior of the original object. For Java objects, the "klass" contains a C++ style "vtable". | 是对象指向它的类的元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。并不是所有的虚拟机实现都必须在对象数据上保留类型指针，换句话说查找对象的元数据信息并不一定要经过对象本身。

## 对象布局

插件：

```java
// https://mvnrepository.com/artifact/org.openjdk.jol/jol-core
compile group: 'org.openjdk.jol', name: 'jol-core', version: '0.10'
```

### `new Object()`占用多少字节？16字节？

```java
java -XX:+PrintCommandLineFlags -version

-XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC
java version "1.8.0_171"
Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)
```

* +UseCompressedClassPointers：64bit机器，一个指针8个字节，如果使用压缩会只有4个字节
* -XX:+UseCompressedOops：普通对象指针，如果压缩也是4个字节

```java
public class Main {

    public static void main(String[] args) throws Exception {
        Object o = new Object();
        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
}
/*
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 00 00 00 (00000001 00000000 00000000 00000000) (1)
      4     4        (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
      8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

markword:8个字节
_kclass:4个字节
没有成员变量：instance data:0字节
紧接着4个字节，是对齐要使用4个字节，即凑成8个字节

即共16个字节
*/
```

### 对象头之markword

markword共8个字节，64bit，包括：**锁信息**，**gc信息**，**identity hashcode**

![](../../content/java_jvm/imgs/hotspot_markword.png)

* 无锁例子

```java
public class Main {

    public static void main(String[] args) throws Exception {
        Object o = new Object();
        int hashCode = o.hashCode();
        int b = hashCode % 2;
        System.out.println(hashCode + " " + Integer.toBinaryString(hashCode) + " " + b);
        System.out.println(ClassLayout.parseInstance(o).toPrintable());
    }
}
/*

2007328737 1110111101001010110011111100001 1
# WARNING: Unable to attach Serviceability Agent. You can try again with escalated privileges. Two options: a) use -Djol.tryWithSudo=true to try with sudo; b) echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0     4        (object header)                           01 e1 67 a5 (00000001 11100001 01100111 10100101) (-1519918847)
      4     4        (object header)                           77 00 00 00 (01110111 00000000 00000000 00000000) (119)
      8     4        (object header)                           e5 01 00 f8 (11100101 00000001 00000000 11111000) (-134217243)
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

markword 64bit，如下

00000000 000000 00000000 01110111 10100101 01100111 11100001 00000001

根据无锁(new)，64bit 具体如下

      unused:25bit         ｜     identity hashcode:31bit        ｜unused | age  | biased_lock  | lock
00000000 000000 00000000 0 ｜ 1110111 10100101 01100111 11100001 ｜ 0     | 0000 |     0        |   01
                           ｜                                    ｜       |      |              |
*/
```

#### 锁升级过程

无锁态(new) =》 偏向锁 =》 轻量级锁，自旋锁，无锁 =》 重量级锁

# 程序编译和代码优化

* 词法，语法分析

词法分析将源代码的字符流转变为标记(Token)集合

语法分析根据Token序列构造抽象语法树（Abstract Syntax Tree）

* Annotation，编译期间读取，修改，添加抽象语法树中的任意符号

* 语义分析与字节码生成

## 类加载过程(*)

```js
加载（即加载class文件）
  =>连接( 验证 =》 准备 =》 解析)
     => 初始化
         => 使用
            => 卸载
```

### 1 加载

这是由类加载器（ClassLoader）执行的。通过一个类的`全限定名`来获取其定义的`二进制字节流`（Class字节码），将这个字节流所代表的静态存储结构转化为运行时(Runtime data area)区域的入口，根据字节码在Java`堆`中生成一个代表这个类的`java.lang.Class`对象。

### 2-1 连接：验证（class字节流的校验）

验证是连接阶段的第一步，这一步主要的目的是确保`class`文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身安全。验证阶段主要包括四个检验过程：文件格式验证、元数据验证、字节码验证和符号引用验证。

### 2-2 连接：准备（分配内存，初始化默认值）

准备阶段是正式为类变量分配内存并设置类变量的初始值阶段，即在方法区中分配这些变量所使用的内存空间。

```java
public static int value = 12;
```

变量value在准备阶段过后的初始值为0而不是12，因为这时候尚未开始执行任何Java方法，而把value赋值为123的`putstatic`指令是程序被编译后，存放于类构造器`<clinit>()`方法之中，所以把value赋值为12的动作将在初始化阶段才会被执行。

相对于一些特殊的情况，如果类字段的字段属性表中存在ConstantValue属性，那在准备阶段变量value就会被初始化为ConstantValue属性所指定的值，例如上面类变量value定义为：

```java
public static final int value = 123;
```

编译时javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将value设置为123。

### 2-3 连接：解析

解析阶段是虚拟机常量池内的**符号引用**替换为**直接引用**的过程。

解析后的信息存储在ConstantPoolCache类实例中，如
* 类或接口的解析
* 字段解析
* 方法解析
* 接口方法解析

### 3 初始化

初始化阶段是类加载最后一个阶段，前面的类加载阶段之后，除了在加载阶段可以自定义类加载器以外，其它操作都**由JVM主导**。到了初始阶段，才开始真正执行类中定义的Java程序代码。

初始化阶段是执行类构造器`<clinit>`方法的过程（注意`<clinit>`不是类的构造函数）。

<font color='red'>`clinit`方法是由编译器自动收集类中的**类静态变量的赋值操作和静态语句块**中的语句合并而成的。JVM会保证`clinit`方法执行之前，父类的`clinit`方法已经执行完毕。</font>

#### 什么时候需要对类进行初始化?

1. 使用`new`该类实例化对象的时候
2. 读取或设置`类静态字段`的时候（但被final修饰的字段，在编译时就被放入常量池；连接-准备阶段会赋予变量常量值；所以(`static final`)的静态字段除外）
3. 调用`类的静态方法`的时候
4. 使用反射`Class.forName("xxx")`对类进行反射调用的时候，该类需要初始化；
5. 初始化一个类的时候，有父类，`先初始化父类`（注：1. 接口除外，父接口在调用的时候才会被初始化；2.子类引用父类静态字段，只会引发父类初始化）；
6. 被标明为启动类的类（即包含`main()方法`的类）要初始化；
7. 当使用JDK1.7的动态语言支持时，如果一个`java.invoke.MethodHandle`实例最后的解析结果REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

以上情况称为对一个类进行主动引用，且有且只要以上几种情况是需要对类进行初始化：

* 所有类变量静态初始化语句和静态代码块都会在编译时被前端编译器放在收集器里头，存放到一个特殊的方法中，这个方法就是`<clinit>`方法，即类/接口初始化方法，该方法只能在类加载的过程中由JVM调用；

* 编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块中只能访问到定义在静态语句块之前的变量；

* 如果超类还没有被初始化，那么优先对超类初始化，但在`<clinit>`方法内部不会显示调用超类的`<clinit>`方法，由JVM负责保证一个类的`<clinit>`方法执行之前，它的超类`<clinit>`方法已经被执行。

* JVM必须确保一个类在初始化的过程中，如果是多线程需要同时初始化它，**仅仅只能允许其中一个线程对其执行初始化操作**，其余线程必须等待，只有在活动线程执行完对类的初始化操作之后，才会通知正在等待的其它线程。(<font color='red'>所以可以利用静态内部类实现线程安全的单例模式</font>)

* 如果一个类没有声明任何的类变量，也没有静态代码块，那么可以没有类`<clinit>`方法；

#### 附：类实例初始化过程？

* 实例初始化就是执行`<init>()`方法
* `<init>()`方法可能重载有多个，有几个构造器就有几个`<init>()`方法
* <init>()方法由非静态实例变量显示赋值代码、非静态代码块、对应构造器代码组成
* 非静态实例变量显示赋值代码和非静态代码块代码从上到下顺序执行，而对应构造器代码最后执行
* 每次创建实例对象，调用对应构造器，执行的就是对应的`<init>`方法
* `<init>`方法的首行是super()或super(实参)，对应父类的`<init>`方法，即先执行父类实例初始化
* 实例创建了几次，初始化就执行了几次

#### 附：static执行时机？

static块的执行发生在**类**"初始化"的阶段（注意不是**类实例**初始化过程）。类初始化阶段，jvm会完成对静态变量的初始化，静态块执行等工作。

是否执行static块的几种情况：

* 第一次`new A()`会；因为这个过程包括了初始化

* 第一次`Class.forName("A")`会；因为这个过程相当于`Class.forName("A",true,this.getClass().getClassLoader())`;

```java
@CallerSensitive
public static Class<?> forName(String className)
            throws ClassNotFoundException {
    Class<?> caller = Reflection.getCallerClass();
    return forName0(className, true, ClassLoader.getClassLoader(caller), caller);
}
```

* 第一次`Class.forName("A",false,this.getClass().getClassLoader())`不会。因为false指明了装载类的过程中，不进行类初始化；没有类初始化，则不会执行static块。

* 类似`getSystemClassLoader().loadClass("com.other.Hello");`也不会。

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

### 类加载器分类

加载动作放到JVM外部实现，以便让应用程序决定如何获取所需的类, 类加载器大致可以分为以下3部分：

#### 1. 根加载器`Bootstrap ClassLoader`

最顶层的加载器，其没有任何父加载器，由C++编写，主要负责虚拟机核心类库的加载，例如`java.lang`包

将存放于`<JAVA_HOME>\lib`目录中的，或者被`-Xbootclasspath`参数所指定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如 rt.jar 名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用。

#### 2. 扩展类加载器`Extension ClassLoader`

扩展类加载器的父加载器是根加载器，纯Java语言实现，将`<JAVA_HOME>\lib\ext`目录下的，或者被java.ext.dirs系统变量所指定的路径中的所有类库加载。开发者可以直接使用扩展类加载器。

#### 3. 系统类加载器`System ClassLoader`(或应用类加载器`Application ClassLoader`)

负责加载用户类路径(ClassPath)上所指定的类库，通常我们自己写的类就是由其加载的

#### 4. 自定义类加载器`Custom CLassLoader`

所有自定义类加载器都是`ClassLoader`的直接子类或者间接子类（java.lang.ClassLoader是一个抽象类）

#### 双亲委派模型（*）

工作过程为：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的加载器都是如此，因此所有的类加载请求都会传给顶层的启动类加载器，只有当父加载器反馈自己无法完成该加载请求（该加载器的搜索范围中没有找到对应的类）时，子加载器才会尝试自己去加载

```java
protected synchronized Class loadClass(String name, boolean resolve)
        throws ClassNotFoundException {
    // 首先检查该name指定的class是否有被加载
    Class c = findLoadedClass(name);
    if (c == null) {
        try {
            if (parent != null) {
                // 如果parent不为null，则调用parent的loadClass进行加载
                c = parent.loadClass(name, false);
            } else {
                // parent为null，则调用BootstrapClassLoader进行加载
                c = findBootstrapClass0(name);
            }
        } catch (ClassNotFoundException e) {
            // 如果仍然无法加载成功，则调用自身的findClass进行加载
            c = findClass(name);
        }
    }
    if (resolve) {
        resolveClass(c);
    }
    return c;
}
```

##### 类加载器中findClass与loadClass的区别？

findClass（）用于写类加载逻辑、loadClass（）方法的逻辑里如果父类加载器加载失败则会调用自己的findClass（）方法完成加载，保证了双亲委派规则。

1. 如果不想打破双亲委派模型，那么只需要重写findClass方法即可
2. 如果想打破双亲委派模型，那么就需要重写整个loadClass方法

##### 为什么使用双亲委派？

1. 采用双亲委派模式的是好处是Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以**避免类的重复加载**，当父亲已经加载了该类时，就没有必要让子类ClassLoader再加载一次

2. Java核心Api中定义类型不会被随意替换，假设通过网络传递一个名为`java.lang.Integer`的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的java.lang.Integer，而直接返回已加载过的Integer.class，这样便可以**防止核心API库被随意篡改**

#### 类加载题目例子（Java7环境）

```java
class Singleton{
    private static Singleton singleton = new Singleton();
    static {
        System.out.println("static block");
    }
    public static int value1;
    public static int value2 = 0;
    public static int value3 = 10;

    private Singleton(){
        System.out.println(String.format("before Singleton constructor: value1:%d, value2:%d,value3:%d",value1,value2,value3));
        value1++;
        value2++;
        value3++;
        System.out.println(String.format("after Singleton constructor: value1:%d, value2:%d, value3:%d",value1,value2,value3));
    }

    public static Singleton getInstance(){
        return singleton;
    }

}

class Singleton2{
    static{
        System.out.println(String.format("after Singleton2 constructor: v1:%d, v2:%d, v3:%d",v1,v2,v3));
    }
    public static int v1;
    public static int v2 = 0;
    public static int v3 = 10;
    static {
        System.out.println("static block");
    }
    private static Singleton2 singleton2 = new Singleton2();

    private Singleton2(){
        System.out.println(String.format("before Singleton2 constructor: v1:%d, v2:%d, v3:%d",v1,v2,v3));
        v1++;
        v2++;
        v3++;
        System.out.println(String.format("after Singleton2 constructor: v1:%d, v2:%d, v3:%d",v1,v2,v3));
    }

    public static Singleton2 getInstance2(){
        return singleton2;
    }

}

public class MainTest {
    public static final int _1MB = 1024 * 1024;

    public static void main(String[] args) throws Exception{
        // idea 提示 不应该通过类的实例访问静态成员，不过程序本身无任何问题，结果如下：
        Singleton singleton = Singleton.getInstance();
        // 1
        System.out.println("Singleton1 value1:" + singleton.value1);
        // 0
        System.out.println("Singleton1 value2:" + singleton.value2);
        // 10
        System.out.println("Singleton1 value3:" + singleton.value3);

        Singleton2 singleton2 = Singleton2.getInstance2();
        // 1
        System.out.println("Singleton2 v1:" + singleton2.v1);
        // 1
        System.out.println("Singleton2 v2:" + singleton2.v2);
        // 11
        System.out.println("Singleton2 v3:" + singleton2.v3);
    }

}
```

##### 结果分析

Singleton类初始化的时候，先要执行`private static Singleton singleton = new Singleton();`这句静态变量的初始化，必须new对象执行构造函数了，执行完后，接着执行静态代码块

Singleton2类初始化的时候，从上往下，执行静态变量赋值，静态代码块，然后才执行`private static Singleton2 singleton2 = new Singleton2();`这句静态变量的初始化，然后new对象执行构造函数

```java
before Singleton constructor: value1:0, value2:0,value3:0
after Singleton constructor: value1:1, value2:1, value3:1
static block
Singleton1 value1:1
Singleton1 value2:0
Singleton1 value3:10
after Singleton2 constructor: v1:0, v2:0, v3:10
static block
before Singleton2 constructor: v1:0, v2:0, v3:10
after Singleton2 constructor: v1:1, v2:1, v3:11
Singleton2 v1:1
Singleton2 v2:1
Singleton2 v3:11
```

* 《Java编程思想》第5章：调用构造函数是编译器的责任，必须要让编译器知道调用的是哪个方法；分配内存空间后，就会调用构造函数，确保使用对象前，对象已经被初始化了。

对象创建的过程：（**类初始化 和 类实例化**）

1. 首次创建对象时，类中的静态方法/静态字段首次被访问时，java解释器必须先查找类路径，以定位.class文件
2. 然后载入`.class`(这将创建一个class对象)，有关静态初始化的所有动作都会执行，按顺序执行。因此，**静态初始化只在Class对象首次加载的时候进行一次**
3. 当用`new XXX()`创建对象时，首先在**堆上为对象分配足够的存储空间**
4. 这块存储空间会被清零，这就自动地将对象中的所有基本类型数据都设置成了缺省值（对数字来说就是0，对布尔型和字符型也相同），而引用则被设置成了null。
5. 执行所有出现于字段定义处的初始化动作（**非静态对象的初始化**）
6. 执行构造器。

###### 附：Singleton 结果详细分析

1. 首先执行main中的`Singleton singleton = Singleton.getInstance()`,

2. 访问了静态方法`访问静态方法`, 开始加载类`Singleton`

3. 随后进行类的链接过程(验证 =》 准备 =》 解析)

这里会将为singleton(引用类型)设置为null,value1,value2,value3（基本数据类型）设置默认值0 ）

4. 类的初始化（按照赋值语句进行修改）

```java
private static Singleton singleton = new Singleton();
public static int value1;
public static int value2 = 0;
public static int value3 = 10;
```

`new Singleton()` 不在进行类加载过程(上面已经类加载过了)，直接对象初始化, 如下

```js
before Singleton constructor: value1:0, value2:0, value3:0
after Singleton constructor: value1:1, value2:1, value3:1
```

然后是如下的初始化语句，（注意如下的静态操作只会进行一次）

```java
static {
    System.out.println("static block");
}
public static int value1; // 没有赋值，取在对象实例的构造函数执行后的值
public static int value2 = 0; // 类初次赋值
public static int value3 = 10;  // 类初次赋值
```

所以最后main函数中打印

```js
Singleton1 value1:1
Singleton1 value2:0
Singleton1 value3:10
```

###### 静态初始化再次验证

类中静态初始化语句，只会在类加载的时候执行一次

* code1

```java
Singleton2 singleton2 = Singleton2.getInstance2();
System.out.println("Singleton2 v1:" + singleton2.v1);
System.out.println("Singleton2 v2:" + singleton2.v2);
System.out.println("Singleton2 v3:" + singleton2.v3);

Singleton2 singleton22 = Singleton2.getInstance2();
System.out.println("Singleton2 v1:" + singleton22.v1);
System.out.println("Singleton2 v2:" + singleton22.v2);
System.out.println("Singleton2 v3:" + singleton22.v3);
```

* output1

```java
static block
before Singleton2 constructor: v1:0, v2:0, v3:10
before Singleton2 constructor: v1:1, v2:1, v3:11
Singleton2 v1:1
Singleton2 v2:1
Singleton2 v3:11
Singleton2 v1:1
Singleton2 v2:1
Singleton2 v3:11
```

* code2

```java
Singleton singleton = Singleton.getInstance();
System.out.println("Singleton1 value1:" + singleton.value1);
System.out.println("Singleton1 value2:" + singleton.value2);
System.out.println("Singleton1 value3:" + singleton.value3);

Singleton singleton2 = new Singleton();
System.out.println("Singleton1 value1:" + singleton2.value1);
System.out.println("Singleton1 value2:" + singleton2.value2);
System.out.println("Singleton1 value3:" + singleton2.value3);
```

* output2

```java
before Singleton constructor: value1:0, value2:0, value3:0
after Singleton constructor: value1:1, value2:1, value3:1
static block
Singleton1 value1:1
Singleton1 value2:0
Singleton1 value3:10
before Singleton constructor: value1:1, value2:0, value3:10
after Singleton constructor: value1:2, value2:1, value3:11
Singleton1 value1:2
Singleton1 value2:1
Singleton1 value3:11
```

# 自定义类加载器

## 自定义类加载器的一般步骤

1. 继承`ClassLoader`

2. 重写`loadClass()`方法（默认是双亲委派，重写可能会打破双亲委派模型）

3. 重写`findClass()`方法
    * class文件路径判断和获取
    * 将class文件载入内存
    * 对载入内存的字节码数据，调用`defineClass()`方法将字节码转化为类

## MyClassLoader 实践

* `Hello.java`编译成`Hello.class`

```java
package com.other;

/**
 * @Author mubi
 * @Date 2020/12/25 07:21
 */
public class Hello {
    public String test() {
        System.out.println("Loader Class is:" + getClass().getClassLoader().getClass());
        return "hello";
    }
}
```

* MyComOtherClassLoader

```java
package com.mb;


import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.lang.reflect.Method;

/**
 * @Author mubi
 * @Date 2019/3/20 10:54 PM
 */
public class MyComOtherClassLoader extends ClassLoader{
    public String path;
    public String packageName;
    public String className;

    public MyComOtherClassLoader() {
        super(ClassLoader.getSystemClassLoader());
    }

    private String classNameToPath() {
        // 得到类文件的URL
        return path + "/" + packageName.replace('.', '/')
                + "/" + className + ".class";
    }

    @Override
    public Class loadClass(String name) throws ClassNotFoundException {
        // 非com.test package下面的类，都用默认的双亲委派模型去加载，否则用自定义的加载方法
        if (!name.contains("com.other")) {
            // 是否已经被加载
            Class loadedClass = findLoadedClass(name);
            if (loadedClass == null) {
                // 用父类去加载该类
                loadedClass = getParent().loadClass(name);
                return loadedClass;
            } else {
                return loadedClass;
            }
        }

        // 自定义加载
        int i = name.lastIndexOf('.');
        packageName = "";
        if (i != -1) {
            packageName = name.substring(0, i);
//            System.out.println("package: " + packageName);
            className = name.substring(i + 1);
//            System.out.println("class name: " + name);
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                sm.checkPackageAccess(packageName);
            }
        }
        //依然调用父类的方法
//        return super.loadClass(name);
        return this.findClass(name);
    }

    @Override
    public Class<?> findClass(String name) throws ClassNotFoundException {
        int i = name.lastIndexOf('.');
        packageName = "";
        if (i != -1) {
            packageName = name.substring(0, i);
//            System.out.println("package: " + packageName);
            className = name.substring(i + 1);
//            System.out.println("class name: " + name);
            SecurityManager sm = System.getSecurityManager();
            if (sm != null) {
                sm.checkPackageAccess(packageName);
            }
        }
        Class<?> clazz = this.findLoadedClass(name); // 父类已加载
        if(null != clazz){
            return clazz;
        }

//        System.out.println("findClass param name: " + name);
        byte[] b = this.getClassBytes();
//        System.out.println("b len:" + b.length);
        // 把字节流转化为Class对象
        clazz = defineClass(null, b, 0, b.length);
        return clazz;
    }
    
    /**
     * .class加载成byte数组，即获取到二进制数据流
     */
    public byte[] getClassBytes() {
        String classPath = classNameToPath();
//        System.out.println("classPath:" + classPath);
        File file=new File(classPath);
        try(FileInputStream fis = new FileInputStream(file);ByteArrayOutputStream bos=new ByteArrayOutputStream();) {
            byte[] b=new byte[1024*2];
            int n;
            while((n=fis.read(b))!=-1){
                bos.write(b, 0, n);
            }
            return bos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    static void testMyClassLoaderHello() throws Exception{
        String path = "/Users/mubi/git_workspace/test/build/classes/java/main";
        MyComOtherClassLoader myClassLoader = new MyComOtherClassLoader();
        myClassLoader.path = path;
        Class clazz = myClassLoader.loadClass("com.other.Hello");
        // 通过自定义加载获取到Class对象，接着就能用反射进行各种操作了
        Object obj = clazz.newInstance();
        System.out.println("===" + obj.getClass());
        Method method = clazz.getDeclaredMethod("test", null);
        Object c = method.invoke(obj, null);
        if(c != null){
            System.out.println("method return: " + c.getClass() + ": " + c);
        }else {
            System.out.println("null method return:" + c);
        }
    }

    static void testMyClassLoaderString() throws Exception{
        MyComOtherClassLoader myClassLoader = new MyComOtherClassLoader();
        Class clazz = myClassLoader.loadClass("java.lang.String");
        Object obj = clazz.newInstance();
        System.out.println("----" + obj.getClass());
        Method method = clazz.getDeclaredMethod("length", null);
        Object c = method.invoke(obj, null);
        if(c != null){
            System.out.println("method return: " + c.getClass() + ": " + c);
        } else {
            System.out.println("null method return:" + c);
        }
    }

    public static void main(String[] args) throws Exception {
        testMyClassLoaderHello();
        testMyClassLoaderString();
    }

}
/*
===class com.other.Hello
Loader Class is:class com.mb.MyComOtherClassLoader
method return: class java.lang.String: hello
----class java.lang.String
method return: class java.lang.Integer: 0
 */
```

### 用不同类加载器实现类多次加载

* Hello.java

```java
package com.other;

/**
 * @Author mubi
 * @Date 2020/12/25 07:21
 */
public class Hello {

    static {
        System.out.println("---------Hello static");
    }

    public String test() {
        System.out.println("Loader Class is:" + getClass().getClassLoader().getClass());
        return "hello";
    }
}
```

* 测试程序

```java
public static void main(String[] args) throws Exception {
    testMyClassLoaderHello();

    Hello hello = new Hello();
    String rs = hello.test();
    System.out.println("hello test:" + rs);

    Hello hello2 = new Hello();
    String rs2 = hello2.test();
    System.out.println("hello test2:" + rs2);
}
```

* 程序输出

```java
---------Hello static
===class com.other.Hello
Loader Class is:class com.mb.MyComOtherClassLoader
method return: class java.lang.String: hello
---------Hello static
Loader Class is:class sun.misc.Launcher$AppClassLoader
hello test:hello
Loader Class is:class sun.misc.Launcher$AppClassLoader
hello test2:hello
```

### MyClassLoader实现热加载

```java
public static void testHotDeploy() throws Exception{
    while (true){
        String rootPath = "/Users/mubi/git_workspace/java8/java8-api/src/main/java";
        MyComOtherClassLoader myClassLoader = new MyComOtherClassLoader();
        myClassLoader.path = rootPath;
        Class clazz = myClassLoader.loadClass("com.hotload.Test");
        Object obj = clazz.newInstance();
        String helloRs = (String)clazz.getMethod("hello").invoke(obj);
        System.out.println("Test:hello() return:" + helloRs);

        TimeUnit.SECONDS.sleep(2);
    }
}
```

只需要修改`Test.class`文件，即可自动加载，而不需要重新部署项目

```java
mubi@mubideMacBook-Pro hotload $ ll
total 32
-rw-r--r--  1 mubi  staff   1.6K  7 27 09:13 LoadTestMain.java
-rw-r--r--  1 mubi  staff   3.2K  7 27 08:58 MyComOtherClassLoader.java
-rw-r--r--  1 mubi  staff   300B  7 27 09:01 Test.class
-rw-r--r--  1 mubi  staff   198B  7 27 08:58 Test.java
mubi@mubideMacBook-Pro hotload $ vim Test.java
mubi@mubideMacBook-Pro hotload $ javac Test.java
mubi@mubideMacBook-Pro hotload $
```

![](../../content/java_jvm/imgs/jvm_classload.png)

### 如何改变 new 的类加载器？

1. 双亲委派

保证被加载类只会加载一次；自定义类加载器可打破双亲委派

2. 全盘委托

当一个classloader加载一个Class的时候，这个Class所依赖的和引用的其它Class通常也由这个classloader负责载入。

3. 类加载还采用了cache机制

如果cache中保存了这个Class就直接返回它，如果没有才从文件中读取和转换成Class，并存入cache，这就是为什么修改了Class但是必须重新启动JVM才能生效，并且类只加载一次的原因

## 自定义类加载器的优缺点 // TODO

* 加密

* 从非标准的来源加载代码

* 部署在同一个服务器上的两个Web应用程序所使用的Java类库可以实现相互隔离。

* 部署在同一个服务器上的两个Web应用程序所使用的Java类库可以相互共享

* 支持热替换(特殊的动态加载机制)

## com.mysql.jdbc.Driver（打破`parents delegate`）

## 例子代码

```java
public static void main(String[] args) {
    Connection conn = null;
    Statement stmt = null;
    try{
        //STEP 1: Register JDBC driver
        Class.forName("com.mysql.jdbc.Driver");
        //STEP 2: Open a connection
        conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test",
                "root", "");
        //STEP 3: Execute a query
        stmt = conn.createStatement();
        ResultSet rs = stmt.executeQuery("select * from tb_user limit 5");
        //STEP 4: Get results
        while(rs.next()){
            System.out.println(rs.getString("id") + " " + rs.getString("name"));
        }
        rs.close();
    }catch(Exception e){

    }//end try
}
```

## 源码逻辑说明

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    public Driver() throws SQLException {
    }

    static {
        try {
            // 静态方法将Driver实例注册到DriverManager中
            DriverManager.registerDriver(new Driver());
        } catch (SQLException var1) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

```java
/**
    * Registers the given driver with the {@code DriverManager}.
    * A newly-loaded driver class should call
    * the method {@code registerDriver} to make itself
    * known to the {@code DriverManager}. If the driver is currently
    * registered, no action is taken.
    *
    * @param driver the new JDBC Driver that is to be registered with the
    *               {@code DriverManager}
    * @exception SQLException if a database access error occurs
    * @exception NullPointerException if {@code driver} is null
    */
public static synchronized void registerDriver(java.sql.Driver driver)
    throws SQLException {

    registerDriver(driver, null);
}
```

### Connection的`getConnection`方法

```java
//  Worker method called by the public getConnection() methods.
private static Connection getConnection(
    String url, java.util.Properties info, Class<?> caller) throws SQLException {
    /*
        * When callerCl is null, we should check the application's
        * (which is invoking this class indirectly)
        * classloader, so that the JDBC driver class outside rt.jar
        * can be loaded from here.
        */
    // 得到线程上下文类加载器
    ClassLoader callerCL = caller != null ? caller.getClassLoader() : null;
    synchronized(DriverManager.class) {
        // synchronize loading of the correct classloader.
        if (callerCL == null) {
            callerCL = Thread.currentThread().getContextClassLoader();
        }
    }

    if(url == null) {
        throw new SQLException("The url cannot be null", "08001");
    }

    println("DriverManager.getConnection(\"" + url + "\")");

    // Walk through the loaded registeredDrivers attempting to make a connection.
    // Remember the first exception that gets raised so we can reraise it.
    SQLException reason = null;

    // 递归DruidManager中已经注册的驱动类，然后验证数据库驱动是否可以被制定的类加载器加载
    // 如果验证通过则返回Connection,此刻返回的Connection则属数据库厂商提供的实例
    for(DriverInfo aDriver : registeredDrivers) {
        // If the caller does not have permission to load the driver then
        // skip it.
        if(isDriverAllowed(aDriver.driver, callerCL)) {
            try {
                println("    trying " + aDriver.driver.getClass().getName());
                Connection con = aDriver.driver.connect(url, info);
                if (con != null) {
                    // Success!
                    println("getConnection returning " + aDriver.driver.getClass().getName());
                    return (con);
                }
            } catch (SQLException ex) {
                if (reason == null) {
                    reason = ex;
                }
            }

        } else {
            println("    skipping: " + aDriver.getClass().getName());
        }

    }

    // if we got here nobody could connect.
    if (reason != null)    {
        println("getConnection failed: " + reason);
        throw reason;
    }

    println("getConnection: no suitable driver found for "+ url);
    throw new SQLException("No suitable driver found for "+ url, "08001");
}
```

```java
private static boolean isDriverAllowed(Driver driver, ClassLoader classLoader) {
    boolean result = false;
    if(driver != null) {
        Class<?> aClass = null;
        try {
            // 使用线程上下文类加载器进行数据库驱动的加载和初始化
            aClass = Class.forName(driver.getClass().getName(), true, classLoader);
        } catch (Exception ex) {
            result = false;
        }

        result = ( aClass == driver.getClass() ) ? true : false;
    }

    return result;
}
```

数据库驱动加载接口被作为JDK核心标准类库的一部分，由于JVM类加载的双亲委托(`parents delegate`)机制的限制，启动类加载器不可能加载得到第三方厂商提供的具体实现。如何解决？

自定义类加载器，线程上下文类加载器：有了线程上下文类加载器，启动类加载器(根加载器)反倒需要委托子类加载器去加载厂商提供的对JDK定义的SPI(Service Provider Interface)的实现

## Java类加载机制：到底能不能自己自定义java.lang.String类

* 不可以加载自定义的java.开头的任何类。
* 因为JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的算法，当在loadClass方法中搜索不到类时，loadClass方法就会调用findClass方法来搜索类，所以我们只需重写该方法即可。如没有特殊的要求，一般不建议重写loadClass搜索类的算法。
* 如果不想打破双亲委派模型，那么只需要重写findClass方法即可
* 如果想打破双亲委派模型，那么就重写整个loadClass方法

### defineClass?

不论自定义类加载器怎么写，都会调用defineClass方法

```java
protected final Class<?> defineClass(String name, byte[] b, int off, int len,
                                         ProtectionDomain protectionDomain)
        throws ClassFormatError
{
    protectionDomain = preDefineClass(name, protectionDomain);
    String source = defineClassSourceLocation(protectionDomain);
    Class<?> c = defineClass1(this, name, b, off, len, protectionDomain, source);
    postDefineClass(c, protectionDomain);
    return c;
}


private ProtectionDomain preDefineClass(String name,
                                            ProtectionDomain pd)
{
    if (!checkName(name))
        throw new NoClassDefFoundError("IllegalName: " + name);

    // Note:  Checking logic in java.lang.invoke.MemberName.checkForTypeAlias
    // relies on the fact that spoofing is impossible if a class has a name
    // of the form "java.*"
    if ((name != null) && name.startsWith("java.")
            && this != getBuiltinPlatformClassLoader()) {
        throw new SecurityException
            ("Prohibited package name: " +
                name.substring(0, name.lastIndexOf('.')));
    }
    if (pd == null) {
        pd = defaultDomain;
    }

    if (name != null) {
        checkCerts(name, pd.getCodeSource());
    }

    return pd;
}
```

从上面代码中可以看到，如果全限定名中是`java.`开头，则直接报错：Prohibited package name。

# Java安全模型--沙箱(sandbox)

沙箱是一个限制程序运行的环境。沙箱机制就是将 Java 代码限定在虚拟机(JVM)特定的运行范围中，并且严格限制代码对本地系统资源访问，通过这样的措施来保证对代码的有效隔离，防止对本地系统造成破坏。沙箱主要限制系统资源访问，那系统资源包括什么？——CPU、内存、文件系统、网络。不同级别的沙箱对这些资源访问的限制也可以不一样。

所有的Java程序运行都可以指定沙箱，可以定制安全策略。

<a href='https://developer.ibm.com/zh/articles/j-lo-javasecurity/'>ibm:Java中的安全模型</a>
