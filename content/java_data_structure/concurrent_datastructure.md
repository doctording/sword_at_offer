---
title: "Java8 concurrent数据结构"
layout: page
date: 2019-05-25 00:00
---

[TOC]

# `concurrent`数据结构

## ConcurrentMap(interface)

```java
public interface ConcurrentMap<K, V> extends Map<K, V> {
```

## ConcurrentHashMap(class)

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
```

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_jvm/imgs/concurrent_hashmap.png)

### 底层:数组+链表/红黑树，CAS + synchronized控制并发

```java
 /* ---------------- Fields -------------- */

/**
    * The array of bins. Lazily initialized upon first insertion.
    * Size is always a power of two. Accessed directly by iterators.
    */
transient volatile Node<K,V>[] table;

/**
    * The next table to use; non-null only while resizing.
    */
private transient volatile Node<K,V>[] nextTable;

/**
    * Base counter value, used mainly when there is no contention,
    * but also as a fallback during table initialization
    * races. Updated via CAS.
    */
private transient volatile long baseCount;

/**
    * Table initialization and resizing control.  When negative, the
    * table is being initialized or resized: -1 for initialization,
    * else -(1 + the number of active resizing threads).  Otherwise,
    * when table is null, holds the initial table size to use upon
    * creation, or 0 for default. After initialization, holds the
    * next element count value upon which to resize the table.
    */
private transient volatile int sizeCtl;

/**
    * The next table index (plus one) to split while resizing.
    */
private transient volatile int transferIndex;

/**
    * Spinlock (locked via CAS) used when resizing and/or creating CounterCells.
    */
private transient volatile int cellsBusy;

/**
    * Table of counter cells. When non-null, size is a power of 2.
    */
private transient volatile CounterCell[] counterCells;

// views
private transient KeySetView<K,V> keySet;
private transient ValuesView<K,V> values;
private transient EntrySetView<K,V> entrySet;
```

### put方法

```java
/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key, value 都不能为null
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // CAS方式进行添加结点
            if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // synchronized 同步操作
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                    (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                        value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

1. 先判断key和value是否为null，为null则抛出异常

2. 判断table是否初始化，如果没有则进行初始化

3. 计算key的hash值，并得到插入的数组索引。

4. 找到table[i]的位置，如果为null直接插入，如果不为null判断此key是否存在，如果存在直接覆盖，如果不存在进行判断如果head节点是树节点，按照红黑树的方式插入新的节点，如果不是则按照链表的方式插入，同时会判断当前的链表长度是否大于`TREEIFY_THRESHOLD=8`，如果大于则转为红黑树再插入，否则直接插入，插入采用的CAS自旋的方式。

5. 最后判断table的size是否需要扩容，如果需要则扩容，否则就结束。在扩容的时候会在链表头部插入forward，如果其他线程检测到需要插入的位置被forward节点占有，就帮助进行扩容。

### 获取size

```java
 /**
    * Returns the number of mappings. This method should be used
    * instead of {@link #size} because a ConcurrentHashMap may
    * contain more mappings than can be represented as an int. The
    * value returned is an estimate; the actual count may differ if
    * there are concurrent insertions or removals.
    *
    * @return the number of mappings
    * @since 1.8
    */
public long mappingCount() {
    long n = sumCount();
    return (n < 0L) ? 0L : n; // ignore transient negative values
}
```

一个大概的数值，因为可能在统计的时候有其他线程正在执行插入或删除操作

### 对比Java7分段锁优劣

#### 锁粒度小

只需要锁住这个链表的head节点，并不会影响其他的table元素的读写，影响更小

#### 扩容不足与改进

* 在于并发扩容的时候，由于操作的table都是同一个，不像JDK7中分段控制，所以这里需要等扩容完之后，所有的读写操作才能进行，所以扩容的效率就成为了整个并发的一个瓶颈点

* 引入了一个ForwardingNode类，在一个线程发起扩容的时候，就会改变`sizeCtl`这个值

```java
sizeCtl ：默认为0，用来控制table的初始化和扩容操作，具体应用在后续会体现出来。
-1 代表table正在初始化
-N 表示有N-1个线程正在进行扩容操作
其余情况：
1、如果table未初始化，表示table需要初始化的大小。
2、如果table初始化完成，表示table的容量，默认是table大小的0.75倍
```

扩容时候会判断这个值，如果超过阈值就要扩容，首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素f，初始化一个forwardNode实例fwd，如果f == null，则在table中的i位置放入fwd，否则采用头插法的方式把当前旧table数组的指定任务范围的数据给迁移到新的数组中，然后 给旧table原位置赋值fwd。直到遍历过所有的节点以后就完成了复制工作，把table指向nextTable，并更新sizeCtl为新数组大小的0.75倍 ，扩容完成。在此期间如果其他线程的有读写操作都会判断head节点是否为forwardNode节点，如果是就帮助扩容。

* 参考

作者：葬月魔帝
来源：CSDN
原文：<a href="https://blog.csdn.net/u010454030/article/details/82458413" target="_blank">理解Java7和8里面HashMap+ConcurrentHashMap的扩容策略
</a>

版权声明：本文为博主原创文章，转载请附上博文链接！

## ConcurrentLinkedQueue(class) 无界线程安全

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {
    private static final long serialVersionUID = 196745693267521676L;
```

## CopyOnWriteArrayList(class) 适合读多写少的并发场景

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
```

* CopyOnWriteArrayList是线程安全的`ArrayList`, 读方法不加锁，写方法加锁
* 读写分离，写时复制出一个新的数组，完成插入、修改或者移除操作后将新数组赋值给array
* 如果读的时候有多个线程正在向CopyOnWriteArrayList添加数据，读还是会读到旧的数据，因为写的时候不会锁住旧的
* volatile 修饰的成员变量在每次被线程访问时，都强迫从共享内存中重读该成员变量的值。而且，当成员变量发生变 化时，强迫线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。

### add方法使用`ReentrantLock`

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

### 缺点

* 内存占用问题

因为CopyOnWrite的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能造成频繁的Yong GC和Full GC。之前我们系统中使用了一个服务由于每晚使用CopyOnWrite机制更新大对象，造成了每晚15秒的Full GC，应用响应时间也随之变长。针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。或者不使用CopyOnWrite容器，而使用其他的并发容器，如ConcurrentHashMap。

* 数据一致性问题(只保证`最终一致性`)

CopyOnWrite容器只能保证数据的`最终一致性`，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

## CopyOnWriteArraySet(class 基于 CopyOnWriteArrayList)

```java
public class CopyOnWriteArraySet<E> extends AbstractSet<E>
        implements java.io.Serializable {
    private static final long serialVersionUID = 5457747651344034263L;

    private final CopyOnWriteArrayList<E> al;

    /**
     * Creates an empty set.
     */
    public CopyOnWriteArraySet() {
        al = new CopyOnWriteArrayList<E>();
    }

    /**
     * Creates a set containing all of the elements of the specified
     * collection.
     *
     * @param c the collection of elements to initially contain
     * @throws NullPointerException if the specified collection is null
     */
    public CopyOnWriteArraySet(Collection<? extends E> c) {
        if (c.getClass() == CopyOnWriteArraySet.class) {
            @SuppressWarnings("unchecked") CopyOnWriteArraySet<E> cc =
                (CopyOnWriteArraySet<E>)c;
            al = new CopyOnWriteArrayList<E>(cc.al);
        }
        else {
            al = new CopyOnWriteArrayList<E>();
            al.addAllAbsent(c);
        }
    }
```

add操作采用`CopyOnWriteArrayList`的`addIfAbsent`方法，加了锁保护，并创建一个新的Object数组；每次add都要进行数组的遍历，性能低

## ArrayBlockingQueue(class) 基于数组的阻塞队列

```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
```

基于数组，先进先出，现场安全的集合类，特点是：可执行时间的阻塞读写，并且容量有限

## LinkedBlockingDeque(class) 基于链表的阻塞队列

```java
public class LinkedBlockingDeque<E>
    extends AbstractQueue<E>
    implements BlockingDeque<E>, java.io.Serializable {
```

### 链表

```java
/**
* Linked list node class
*/
static class Node<E> {
    E item;

    /**
        * One of:
        * - the real successor Node
        * - this Node, meaning the successor is head.next
        * - null, meaning there is no successor (this is the last node)
        */
    Node<E> next;

    Node(E x) { item = x; }
}

/** The capacity bound, or Integer.MAX_VALUE if none */
private final int capacity;

/** Current number of elements */
private final AtomicInteger count = new AtomicInteger();

/**
    * Head of linked list.
    * Invariant: head.item == null
    */
transient Node<E> head;

/**
    * Tail of linked list.
    * Invariant: last.next == null
    */
private transient Node<E> last;
```

### put方法，添加元素

```java
/**
    * Inserts the specified element at the tail of this queue, waiting if
    * necessary for space to become available.
    *
    * @throws InterruptedException {@inheritDoc}
    * @throws NullPointerException {@inheritDoc}
    */
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        /*
            * Note that count is used in wait guard even though it is
            * not protected by lock. This works because count can
            * only decrease at this point (all other puts are shut
            * out by lock), and we (or some other waiting put) are
            * signalled if it ever changes from capacity. Similarly
            * for all other uses of count in other wait guards.
            */
        while (count.get() == capacity) {
            notFull.await();
        }
        enqueue(node);
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}
```

使用了`ReentrantLock`,若向队尾添加元素的时候发现队列已经满了会发生阻塞一直等待空间，以加入元素。

### offer方法，添加元素

```java
public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        if (e == null) throw new NullPointerException();
        long nanos = unit.toNanos(timeout);
        int c = -1;
        final ReentrantLock putLock = this.putLock;
        final AtomicInteger count = this.count;
        putLock.lockInterruptibly();
        try {
            while (count.get() == capacity) {
                if (nanos <= 0)
                    return false;
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(new Node<E>(e));
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        } finally {
            putLock.unlock();
        }
        if (c == 0)
            signalNotEmpty();
        return true;
    }
```

如果发现队列已满无法添加的话，等待指定时间后会直接返回false

## take 方法，取元素

```java
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

若队列为空，则发生阻塞，一直等待有元素

### 基于`LinkedBlockingQueue`的生产者和消费者

```java

import java.util.Random;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

class Basket{
    LinkedBlockingQueue<String> linkedBlockingQueue;
    public Basket(int size){
        linkedBlockingQueue = new LinkedBlockingQueue<>(size);
    }

    public void addToBasket(String s) throws InterruptedException{
        linkedBlockingQueue.put(s);
    }

    public String getFromBasket() throws InterruptedException{
        return linkedBlockingQueue.take();
    }
}

class Producer{
    private Basket basket;

    public Producer(Basket basket){
        this.basket = basket;
    }

    public void produce(String s){
        try {
            this.basket.addToBasket(s);
        }catch (Exception e){
            e.printStackTrace();
        }
    }

    public void print(){
       System.out.println(this.basket.linkedBlockingQueue);
    }
}

class Consumer{
    private Basket basket;

    public Consumer(Basket basket){
        this.basket = basket;
    }

    public String consume(){
        try {
            return this.basket.getFromBasket();
        }catch (Exception e){
            e.printStackTrace();
        }
        return null;
    }
    public void print(){
        System.out.println(this.basket.linkedBlockingQueue);
    }
}

class Main {

    public static void main(String[] args) throws Exception {
        Basket basket = new Basket(3);
        Producer producer = new Producer(basket);
        Consumer consumer = new Consumer(basket);
        new Thread(()->{
            while (true) {
                try{
                    TimeUnit.SECONDS.sleep(2);
                }catch (Exception e){

                }
                int r = new Random().nextInt(10);
                String s = "" + r;
                producer.produce(s);
                System.out.print("produce:" + s + " queue: ");
                producer.print();
            }
        }).start();

        new Thread(()->{
            while (true) {
                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (Exception e) {

                }
                String s = consumer.consume();
                System.out.print("consumer:" + s + " queue: ");
                consumer.print();
            }
        }).start();
    }

}
```

## TODO
