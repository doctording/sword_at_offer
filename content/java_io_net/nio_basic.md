---
title: "Java nio"
layout: page
date: 2019-04-07 00:00
---

[TOC]

# NIO

https://howtodoinjava.com/java-nio-tutorials/

Java NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。

## NIO API 的抽象

### 缓冲区 Buffer

* ByteBuffer
* MappedByteBuffer
* CharBuffer
* DoubleBuffer
* FloatBuffer
* IntBuffer
* LongBuffer
* ShortBuffer
* ByteOrder

它们是数据容器

### 字符集及其相关解码器 和编码器

它们在字节和Unicode字符之间进行转换

### 通道 Channel

Channel是一个对象，可以通过它读取和写入数据。拿 NIO 与原来的 I/O 做个比较，通道就像是流，而且他们面向缓冲区的。

通道与流的不同之处在于通道是双向的。而流只是在一个方向上移动(一个流必须是 InputStream 或者 OutputStream 的子类)， 而 通道 可以用于读、写或者同时用于读写。

通道可以以阻塞(blocking)或非阻塞(nonblocking)模式运行。非阻塞模式的通道永远不会让调用的线程休眠。请求的操作要么立即完成,要么返回一个结果表明未进行任何操作。只有面向流的(stream-oriented)的通道,如 sockets 和 pipes 才能使用非阻塞模式。

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_io/imgs/buffer_channel.png)

### Selector

Selector能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的响应处理。

这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销。

## NIO与IO的区别

* NIO是以`块`的方式处理数据，但是IO是以最基础的字节流的形式去写入和读出的。所以在效率上的话，肯定是NIO效率比IO效率会高出很多。

* NIO不在是和IO一样用OutputStream和InputStream输入流的形式来进行处理数据的，但是又是基于这种流的形式，而是采用了通道和缓冲区的形式来进行处理数据的。还有一点就是NIO的通道是可以`双向`的，但是IO中的流只能是单向的。

* NIO的缓冲区（其实也就是一个字节数组）还可以进行`分片`，可以建立只读缓冲区、直接缓冲区和间接缓冲区，只读缓冲区很明显就是字面意思，直接缓冲区是为加快I/O速度，而以一种特殊的方式分配其内存的缓冲区。

作者：依本多情
来源：CSDN
原文：https://blog.csdn.net/qq_36520235/article/details/81318189 
版权声明：本文为博主原创文章，转载请附上博文链接！

## Scatter/Gather

scatter/gather指的在多个缓冲区上实现一个简单的I/O操作，比如从通道中读取数据到多个缓冲区，或从多个缓冲区中写入数据到通道；

* scatter（分散）：指的是从通道中读取数据分散到多个缓冲区Buffer的过程，该过程会将每个缓存区填满，直至通道中无数据或缓冲区没有空间；

* gather（聚集）：指的是将多个缓冲区Buffer聚集起来写入到通道的过程，该过程类似于将多个缓冲区的内容连接起来写入通道；
