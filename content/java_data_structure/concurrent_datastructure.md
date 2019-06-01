---
title: "Java8 concurrent"
layout: page
date: 2019-05-25 00:00
---

[TOC]

# concurrent数据结构

## ConcurrentMap(interface)

```java
public interface ConcurrentMap<K, V> extends Map<K, V> {
```

## ConcurrentHashMap(class)

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
```

### 底层，数组+链表/红黑树，CAS + synchronized控制并发

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

1. 先判断key和value是否为null，为null扔出异常

2. 判断table是否初始化，如果没有则进行初始化

3. 计算key的hash值，并得到插入的数组索引。

4. 找到table[i]的位置，如果为null直接插入，如果不为null判断此key是否存在，如果存在直接覆盖，如果不存在进行判断如果head节点是树节点，按照红黑树的方式插入新的节点，如果不是则按照链表的方式插入，同时会判断当前的链表长度是否大于8，如果大于则转为红黑树再插入，否则直接插入，插入采用的CAS自旋的方式。

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

* 引入了一个ForwardingNode类，在一个线程发起扩容的时候，就会改变sizeCtl这个值

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
原文：https://blog.csdn.net/u010454030/article/details/82458413
版权声明：本文为博主原创文章，转载请附上博文链接！

## ConcurrentLinkedQueue(class)

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {
    private static final long serialVersionUID = 196745693267521676L;
```
