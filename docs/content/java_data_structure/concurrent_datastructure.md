---
title: "Java8 concurrent数据结构"
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

底层：数组+链表/红黑树，CAS + synchronized控制并发

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {
```

![](../../content/java_data_structure/imgs/concurrent_hashmap.png)

每个桶可能是`链表`结构或者`红黑树`结构，锁针对桶的头节点加，`锁粒度小`

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

* `CopyOnWriteArrayList`是线程安全的`ArrayList`, 读方法不加锁；写方法加锁
* 读写分离：写时复制出一个新的数组，完成插入、修改或者移除操作后将新数组赋值给array
* 如果读的时候有多个线程正在向CopyOnWriteArrayList添加数据；能读到旧的数据，因为写的时候不会锁住旧的数组
* volatile 修饰的成员变量在每次被线程访问时，都强迫从共享内存中重读该成员变量的值。而且，当成员变量发生变化时，强迫线程将变化值回写到共享内存。这样在任何时刻，两个不同的线程总是看到某个成员变量的同一个值。

### add方法使用`ReentrantLock`，`new`新数组

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

#### 内存占用问题

因为`CopyOnWrite`的写时复制机制，所以在进行写操作的时候，内存里会同时驻扎两个对象的内存，旧的对象和新写入的对象（注意:在复制的时候只是复制容器里的引用，只是在写的时候会创建新对象添加到新容器里，而旧容器的对象还在使用，所以有两份对象内存）。如果这些对象占用的内存比较大，比如说200M左右，那么再写入100M数据进去，内存就会占用300M，那么这个时候很有可能造成频繁的Yong GC和Full GC。之前我们系统中使用了一个服务由于每晚使用`CopyOnWrite`机制更新大对象，造成了每晚15秒的Full GC，应用响应时间也随之变长。针对内存占用问题，可以通过压缩容器中的元素的方法来减少大对象的内存消耗，比如，如果元素全是10进制的数字，可以考虑把它压缩成36进制或64进制。或者不使用CopyOnWrite容器，而使用其他的并发容器，如ConcurrentHashMap。

#### 数据一致性问题(只保证`最终一致性`)

CopyOnWrite容器只能保证数据的`最终一致性`，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

## CopyOnWriteArraySet(class 线程安全的无序的集合)

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

1. 它最适合于具有以下特征的应用程序：Set 大小通常保持很小，只读操作远多于可变操作，需要在遍历期间防止线程间的冲突。
2. 它是线程安全的。
3. 因为通常需要复制整个基础数组，所以可变操作（add()、set() 和 remove() 等等）的开销很大。
4. 迭代器支持hasNext(), next()等不可变操作，但不支持可变 remove()等 操作。
5. 使用迭代器进行遍历的速度很快，并且不会与其他线程发生冲突。在构造迭代器时，迭代器依赖于不变的数组快照。

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

### 链表Node结构

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

/** Number of items in the deque */
private transient int count;

/** Maximum number of items in the deque */
private final int capacity;

/** Main lock guarding all access */
final ReentrantLock lock = new ReentrantLock();

/** Condition for waiting takes */
private final Condition notEmpty = lock.newCondition();

/** Condition for waiting puts */
private final Condition notFull = lock.newCondition();
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
