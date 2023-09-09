---
title: "ThreadLocal"
layout: page
date: 2019-02-15 00:00
---

[TOC]

# ThreadLocal

`ThreadLocal`并不是一个线程，而是线程的一个局部变量(属于线程)

`ThreadLocal`用于保存某个线程共享变量：对于同一个static ThreadLocal，不同线程只能从中get，set，remove自己的变量，而不会影响其它线程的变量

## 实现思路

* `Thread`类有一个类型为`ThreadLocal.ThreadLocalMap`的实例变量`threadLocals`，也就是说每个线程有一个自己的`ThreadLocalMap`(是线程自己拥有的一个map，ThreadLocalMap有个静态类Entry使用了弱引用`Entry extends WeakReference<ThreadLocal<?>>`)。

### 使用例子

```java
 public static void main(String[] args) throws Exception {
    Thread t = new Thread(()->{
        ThreadLocal<String> threadLocal = new ThreadLocal<>();
        ThreadLocal<Integer> threadLocal1 = new ThreadLocal<>();
        threadLocal.set("abc");
        threadLocal1.set(100);

        System.out.println(Thread.currentThread().getName() + ":" + threadLocal.get());
        System.out.println(Thread.currentThread().getName() + ":" + threadLocal1.get());
    });
    t.setName("myThread");
    t.start();
}
/** 输出
myThread:abc
myThread:100
*/
```

### threadLocal.set源码

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

/**
 * Create the map associated with a ThreadLocal. Overridden in
 * InheritableThreadLocal.
 *
 * @param t the current thread
 * @param firstValue value for the initial entry of the map
 */
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}


/**
 * Construct a new map initially containing (firstKey, firstValue).
 * ThreadLocalMaps are constructed lazily, so we only create
 * one when we have at least one entry to put in it.
 */
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

ThreadLocalMap的set方法：

```java
private void set(ThreadLocal<?> key, Object value) {

    // We don't use a fast path as with get() because it is at
    // least as common to use set() to create new entries as
    // it is to replace existing ones, in which case, a fast
    // path would fail more often than not.

    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);

    for (Entry e = tab[i];
            e != null;
            e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        if (k == key) {
            e.value = value;
            return;
        }

        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    // 新增一个Entry,key就是`ThreadLocal.this`
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

### 例子代码说明

例子代码Thread对象t的成员变量ThreadLocal.ThreadLocalMap对象threadLocals的ThreadLocalMap.Entry数组包括2个元素

```bash
t:Thread[myThread,5,main]
  threadLocals:ThreadLocal$ThreadLocalMap，两个元素，key:value如下
    entry[0] key: threadLocal, value: abc
    entry[1] key: threadLocal1, value: 100
```

* 注意到`Entry`是个弱引用`WeakReference<ThreadLocal<?>`（`class Entry extends WeakReference<ThreadLocal<?>> {`），如下图所示
![](../../content/java_thread_concurrent/imgs/thread_local.png)

1. 如果当定义出来的`ThreadLocal`对象tl1不用了，即指向`ThreadLocal`对象实例的引用没了，那么这个`ThreadLocal`对象实例要被回收的
2. 假如说this是个强引用，那么显然只要线程不结束，线程的成员变量threadLocals指向的`ThreadLocalMap`实例对象就存在，map里面的引用关系就存在；那么`ThreadLocal`就永远不会被回收，即有内存泄漏了
3. 所以this被设计成为一个弱引用，只要gc发现不用了，就会将`ThreadLocal`对象tl1进行回收；这样当`ThreadLocal`对象被回收后，`ThreadLocalMap`里面的key是个null被回收，这使得value就访问不到,需要回收；但是value的引用是存在的，这就会导致内存泄漏
4. 所以需要显示的remove掉value，即执行`tl1.remove();`

## 使用场景

1. 在进行对象跨层传递的时候，可以考虑`ThreadLocal`，避免方法多次传递，打破层次间的约束

2. 线程间数据隔离

3. 进行事务操作，用于存储线程事务信息

### 自己项目中使用到的实际例子?

比如一个查询，要经过一层一层处理，最后还要记录此次的查询记录；其中有个queryId，可以放到`ThreadLocal`中，这样打点&记录随时可以取出queryId

## ThreadLocal会产生内存泄漏吗?

<font color='red'>会</font>

弱引用做了一道工作，但是仍需要显示的remove掉ThreadLocal的value

## ThreadLocal父子线程之间的数据传递

### 子线程不能获取父线程threadLocal

```java
public static void main(String[] args) throws Exception {
    ThreadLocal<String> mainThreadLocal = new ThreadLocal();
    mainThreadLocal.set("a");

    Thread t = new Thread(()->{
        System.out.println(Thread.currentThread() + ":"
                + mainThreadLocal.get());
    });
    t.setName("myThread");
    t.start();

    System.out.println(Thread.currentThread() + ":"
            + mainThreadLocal.get());
}
/** 输出如下：
Thread[main,5,main]:a
Thread[myThread,5,main]:null
*/
```

### InheritableThreadLocal

```java
public static void main(String[] args) throws Exception {
    InheritableThreadLocal<String> mainInheritThreadLocal = new InheritableThreadLocal();
    mainInheritThreadLocal.set("a");

    Thread t = new Thread(()->{
        System.out.println(Thread.currentThread() + ":"
                + mainInheritThreadLocal.get());
    });
    t.setName("myThread");
    t.start();

    System.out.println(Thread.currentThread() + ":"
            + mainInheritThreadLocal.get());
}
/** 输出
Thread[main,5,main]:a
Thread[myThread,5,main]:a
*/
```

InheritableThreadLocal 继承了 ThreadLocal

```java

public class InheritableThreadLocal<T> extends ThreadLocal<T> {
    /**
     * Computes the child's initial value for this inheritable thread-local
     * variable as a function of the parent's value at the time the child
     * thread is created.  This method is called from within the parent
     * thread before the child is started.
     * <p>
     * This method merely returns its input argument, and should be overridden
     * if a different behavior is desired.
     *
     * @param parentValue the parent thread's value
     * @return the child thread's initial value
     */
    protected T childValue(T parentValue) {
        return parentValue;
    }

    /**
     * Get the map associated with a ThreadLocal.
     *
     * @param t the current thread
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    /**
     * Create the map associated with a ThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the table.
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}
```

在父线程中new子线程的时候,如果`parent.inheritableThreadLocals`不为空，直接copy一份给了子线程；不过之后父子线程的threadLocal就互不想干了

```java
if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
```

eg:

```java
public static void main(String[] args) throws Exception {
    InheritableThreadLocal<String> mainInheritThreadLocal = new InheritableThreadLocal();
    mainInheritThreadLocal.set("a");

    System.out.println(Thread.currentThread() + ":"
            + mainInheritThreadLocal.get());


    Thread t = new Thread(()->{
        System.out.println(Thread.currentThread() + ":"
                + mainInheritThreadLocal.get());

        try{
            TimeUnit.SECONDS.sleep(4);
        }catch (Exception e){

        }

    System.out.println(Thread.currentThread() + ":"
            + mainInheritThreadLocal.get());

    });
    t.setName("myThread");
    t.start();


    mainInheritThreadLocal.remove();
    try{
        TimeUnit.SECONDS.sleep(1);
    }catch (Exception e){

    }
    mainInheritThreadLocal.remove();
    System.out.println(Thread.currentThread() + ":"
            + mainInheritThreadLocal.get());
}
/** 输出
Thread[main,5,main]:a
Thread[myThread,5,main]:a
Thread[main,5,main]:null
Thread[myThread,5,main]:a
*/
```
