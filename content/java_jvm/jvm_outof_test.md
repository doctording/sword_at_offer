---
title: "Java Jvm各内存区域溢出测试"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# java 内存区域和内存溢出异常

## OutOfMemoryError

java vm参数

* -Xms20M

表示设置`堆容量的最小值`为20M，必须以M为单位

* -Xmx20M

表示设置堆容量的最大值为20M，必须以M为单位。将-Xmx和-Xms设置为一样可以避免堆自动扩展。

```java
package java7;

import java.util.ArrayList;
import java.util.List;

/**
 * 参数设置：-verbose:gc -Xms20M -Xmx20M -XX:+HeapDumpOnOutOfMemoryError
 * @Author mubi
 * @Date 2019/2/20 11:05 PM
 */
public class HeapOOM {
    static class OOMObject{

    }
    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>(16);
        while (true){
            list.add(new OOMObject());
        }
    }
}
```

* output

```java
[GC 5499K->3757K(20480K), 0.0069130 secs]
[GC 9901K->8805K(20480K), 0.0089900 secs]
[Full GC 18460K->13805K(20480K), 0.1417260 secs]
[Full GC 17849K->17803K(20480K), 0.1140610 secs]
[Full GC 17803K->17791K(20480K), 0.0981060 secs]
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid42185.hprof ...
Heap dump file created [30470683 bytes in 0.161 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:2245)
	at java.util.Arrays.copyOf(Arrays.java:2219)
	at java.util.ArrayList.grow(ArrayList.java:242)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:216)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:208)
	at java.util.ArrayList.add(ArrayList.java:440)
	at java7.HeapOOM.main(HeapOOM.java:17)
```

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_jvm/imgs/stream_operate.png)

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_jvm/imgs/oom_analyse.png)

* Shallow heap是一个对象消费的内存数量。每个对象的引用需要32（或者64 bits，基于CPU架构）。

* Retained Heap显示的是那些当垃圾回收时候会清理的所有对象的Shallow Heap的总和。

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_jvm/imgs/histogram.png)

* 参考

1. 《Java 虚拟机》（第二版）

2. https://www.cnblogs.com/cnmenglang/p/6261435.html

3. https://www.jianshu.com/p/759e02c1feee

## 虚拟机栈和本地方法栈溢出

* Java程序中，每个线程都有自己的Stack Space(堆栈)。

* -Xss128k：

设置每个线程的堆栈大小。JDK5.0以后每个线程堆 栈大小为1M，以前每个线程堆栈大小为256K。根据应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右

Stack Space用来做方法的递归调用时压入Stack Frame(栈帧)。所以当递归调用太深的时候，就有可能耗尽Stack Space，爆出StackOverflow的错误。

* code

```java
package java7;

/**
 * 参数设置：-Xss160k
 * @Author mubi
 * @Date 2019/2/20 11:05 PM
 */
public class JavaVMStackSOF {
    private int stackLength = 1;
    public void stackLeak(){
        stackLength ++;
        stackLeak();
    }
    public static void main(String[] args) {
       JavaVMStackSOF javaVMStackSOF = new JavaVMStackSOF();
       try{
           javaVMStackSOF.stackLeak();
       }catch (Throwable e){
           System.out.println("stackLength:" + javaVMStackSOF.stackLength);
           throw e;
       }
    }
}
```

output

```java
stackLength:774
Exception in thread "main" java.lang.StackOverflowError
	at java7.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:10)
	at java7.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:11)
	at java7.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:11)
	at java7.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:11)
	at java7.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:11)
	at java7.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:11)
```

参考：

1. https://blog.csdn.net/z69183787/article/details/79228215

## 方法区和运行时常量池溢出

方法区用于存放Class的相关信息，如：类名，访问修饰符，常量池，字符描述，方法描述等。对于这个区域的测试，基本思路是运行时产生大量的类去填满方法区，直到溢出。

* Java7中

* Java7 永久代仍存在；对比java6，其中：符号引用(Symbols)转移到了native heap；字面量(interned strings)转移到了java heap；类的静态变量(class statics)转移到了java heap。

### Java7 常量池 仍是`java.lang.OutOfMemoryError: Java heap space`

```java
package java7;

import java.util.ArrayList;
import java.util.List;

/**
 * VM args: -XX:PermSize=10M -XX:MaxPermSize=10M
 *
 */
public class JavaMethodAreaOOM {
	/**
	 * 在JDK1.6中，intern()方法会把首次遇到的字符串复制到永久代中，
	 * 返回的也是永久代中这个字符串的引用，
	 * 而由StringBuilder创建的字符串实例在Java堆中，所以必然不是同一个引用，将返回false。
	 * 
	 * 而JDK1.7(以及部分其他虚拟机，例如JRockit)的intern()实现不会再复制实例，而是在常量池中记录首次出现的实例引用，
	 * 因此intern()返回的引用和由StringBuilder创建的那个字符串是同一个。
	 */
	static void testStringIntern() {
		String str1 = new StringBuilder("计算机").append("软件").toString();
        System.out.println(str1.intern() == str1); // true

        String str2 = new StringBuilder("ja").append("va").toString();
        System.out.println(str2.intern() == str2); // false
	}

	static String  base = "string";
    public static void main(String[] args) {
        List<String> list = new ArrayList<String>();
        // 不断生成字符串常量
        for (int i=0;i< Integer.MAX_VALUE;i++){
            String str = base + base;
            base = str;
            list.add(str.intern());
        }
    }
}
```

* output

```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:2367)
	at java.lang.AbstractStringBuilder.expandCapacity(AbstractStringBuilder.java:130)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:114)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:415)
	at java.lang.StringBuilder.append(StringBuilder.java:132)
	at java7.JavaMethodAreaOOM.main(JavaMethodAreaOOM.java:31
```

参考：

1. https://blog.csdn.net/tlk20071/article/details/77841841

## 直接内存溢出

NIO的Buffer提供了一个可以不经过JVM内存直接访问系统物理内存的类——DirectBuffer。DirectBuffer类继承自ByteBuffer，但和普通的ByteBuffer不同，普通的ByteBuffer仍在JVM堆上分配内存，其最大内存受到最大堆内存的限制；而DirectBuffer直接分配在物理内存中，并不占用堆空间，其可申请的最大内存受操作系统限制。

直接内存的读写操作比普通Buffer快，但它的创建、销毁比普通Buffer慢。

因此直接内存使用于需要大内存空间且频繁访问的场合，不适用于频繁申请释放内存的场合。

* `-XX:MaxDirectMemorySize`，该值是有上限的，默认是64M，最大为`sun.misc.VM.maxDirectMemory()`，此参数的含义是当Direct ByteBuffer分配的堆外内存到达指定大小后，即触发Full GC

```java
package java7;

import java.nio.ByteBuffer;

/**
 * 参数设置：-verbose:-Xmx20M -XX:MaxDirectMemorySize=10M
 * @Author mubi
 * @Date 2019/2/20 11:05 PM
 */
public class DirectMemoryOOM {
	 public static final int _1MB = 1024 * 1024;

	public static void main(String[] args) throws Exception{
		ByteBuffer.allocateDirect(11 * _1MB);
	}
}
```

* output

```java
Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
	at java.nio.Bits.reserveMemory(Bits.java:658)
	at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:123)
	at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:306)
	at java7.DirectMemoryOOM.main(DirectMemoryOOM.java:14)
```
