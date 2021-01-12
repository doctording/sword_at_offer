---
title: "Java queue"
layout: page
date: 2019-05-25 00:00
---

[TOC]

# 队列

![](../../content/java_data_structure/imgs/java_queue.png)

## 阻塞队列

### LinkedBlockingQueue

LinkedBlockingQueue默认大小是Integer.MAX_VALUE，可以理解为一个缓存的有界等待队列，可以选择指定其最大容量，它是基于链表的队列，此队列按 FIFO（先进先出）排序元素。当生产者往队列中放入一个数据时，缓存在队列内部，当队列缓冲区达到最大值缓存容量时（LinkedBlockingQueue可以通过构造函数指定该值），阻塞生产者队列，直到消费者从队列中消费掉一份数据，生产者线程会被唤醒，反之对于消费者同理。

### ArrayBlockingQueue

ArrayBlockingQueue在构造时需要指定容量，并可以选择是否需要公平性，如果公平参数被设置true，等待时间最长的线程会优先得到处理（其实就是通过将ReentrantLock设置为true来 达到这种公平性的：即等待时间最长的线程会先操作）。通常，公平性会使你在性能上付出代价，只有在的确非常需要的时候再使用它。它是基于数组的阻塞循环队列，此队列按FIFO（先进先出）原则对元素进行排序。

### PriorityBlockingQueue

PriorityBlockingQueue是一个带优先级的队列，而不是先进先出队列。元素按优先级顺序被移除，该队列也没有上限（看了一下源码，PriorityBlockingQueue是对 PriorityQueue的再次包装，是基于堆数据结构的，而PriorityQueue是没有容量限制的，与ArrayList一样，所以在优先阻塞 队列上put时是不会受阻的。虽然此队列逻辑上是无界的，但是由于资源被耗尽，所以试图执行添加操作可能会导致 OutOfMemoryError），但是如果队列为空，那么取元素的操作take就会阻塞，所以它的检索操作take是受阻的。另外，往入该队列中的元 素要具有比较能力。

### SynchronousQueue

SynchronousQueue队列内部仅允许容纳一个元素。当一个线程插入一个元素后会被阻塞，除非这个元素被另一个线程消费。

### ConcurrentLinkedQueue

是一个适用于高并发场景下的队列，通过无锁的方式，实现了高并发状态下的高性能，通常ConcurrentLinkedQueue性能好于BlockingQueue。

它是一个基于链接节点的无界线程安全队列。该队列的元素遵循先进先出的原则。

头是最先加入的，尾是最近加入的，该队列不允许null元素。

#### ConcurrentLinkedQueue & LinkedBlockingQueue

1. LinkedBlockingQueue是使用锁机制，ConcurrentLinkedQueue是使用CAS算法，虽然LinkedBlockingQueue的底层获取锁也是使用的CAS算法

2. 关于取元素，ConcurrentLinkedQueue不支持阻塞去取元素，LinkedBlockingQueue支持阻塞的take()方法。

3. 关于插入元素的性能，但在实际的使用过程中，尤其在多cpu的服务器上，有锁和无锁的差距便体现出来了，ConcurrentLinkedQueue会比LinkedBlockingQueue快很多。
