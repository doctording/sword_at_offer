---
title: "Java 数据结构"
layout: page
date: 2019-05-25 00:00
---

[TOC]

# Java常用数据结构和原理

## Map Map.Entry<K,V>(interface)

## AbstractMap(abstract class)

```java
public abstract class AbstractMap<K,V> implements Map<K,V> {
```

实现一些Map的基础方法

## HashMap(class)

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
```

### 底层实现是`数组+链表`（Java8中链表长度超过8个会转换为树结构）

```java
 /* ---------------- Fields -------------- */

    /**
     * The table, initialized on first use, and resized as
     * necessary. When allocated, length is always a power of two.
     * (We also tolerate length zero in some operations to allow
     * bootstrapping mechanics that are currently not needed.)
     */
    transient Node<K,V>[] table;

    /**
     * Holds cached entrySet(). Note that AbstractMap fields are used
     * for keySet() and values().
     */
    transient Set<Map.Entry<K,V>> entrySet;

    /**
     * The number of key-value mappings contained in this map.
     */
    transient int size;

    /**
     * The number of times this HashMap has been structurally modified
     * Structural modifications are those that change the number of mappings in
     * the HashMap or otherwise modify its internal structure (e.g.,
     * rehash).  This field is used to make iterators on Collection-views of
     * the HashMap fail-fast.  (See ConcurrentModificationException).
     */
    transient int modCount;

    /**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

    /**
     * The load factor for the hash table.
     *
     * @serial
     */
    final float loadFactor;
```

#### 单链表结构

```java
 static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

#### 树结构

```java
/**
* Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
* extends Node) so can be used as extension of either regular or
* linked node.
*/
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
```

### null key(可以)， null val(可以) 与 实际例子

```java
Map<String,String> mp = new HashMap<>();
mp.put(null, null);
mp.put(null, "a");
mp.put("a", null);
mp.forEach((key,val)->{
    System.out.println(String.format("<%s,%s>", key, val));
});
```

* output

```java
<null,a>
<a,null>
```

### key的hash方法(null的hash值设置成了0)

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

### put 方法

```java
 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null) // 数组的位置上是否存在元素，否则数组的位置上创建节点
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode) // 数组的位置上节点不是单链表了，而是树
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 超过8个的链表 转换为树结构
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold) // 超过平衡因子，数组扩容
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

### 非线程安全

### 遍历的无序性

```java
Map<String,String> mp = new HashMap<>();
        mp.put(null, null);
        mp.put(null, "a");
        mp.put("b", null);
        mp.put("a", "a");
        Set<Map.Entry<String, String>> entries = mp.entrySet();
        Iterator<Map.Entry<String, String>> iteratorMap = entries.iterator();
        while (iteratorMap.hasNext()){
            Map.Entry<String, String> next = iteratorMap.next();
            System.out.println(next);
        }
        mp.forEach((key, val)->{
            System.out.println(String.format("%s=%s", key,val));
        });
/* output
null=a
a=a
b=null
*/
```

## Hashtable

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
```

### 实现了Map，继承了Dictionary，底层数组(散列表)+链表(拉链法)

```java
/**
    * The hash table data.
    */
private transient Entry<?,?>[] table;

/**
    * The total number of entries in the hash table.
    */
private transient int count;

/**
    * The table is rehashed when its size exceeds this threshold.  (The
    * value of this field is (int)(capacity * loadFactor).)
    *
    * @serial
    */
private int threshold;

/**
    * The load factor for the hashtable.
    *
    * @serial
    */
private float loadFactor;

/**
    * The number of times this Hashtable has been structurally modified
    * Structural modifications are those that change the number of entries in
    * the Hashtable or otherwise modify its internal structure (e.g.,
    * rehash).  This field is used to make iterators on Collection-views of
    * the Hashtable fail-fast.  (See ConcurrentModificationException).
    */
private transient int modCount = 0;

/** use serialVersionUID from JDK 1.0.2 for interoperability */
private static final long serialVersionUID = 1421746759512286392L;
```

### null key, null value（不可以）

* Java8程序

```java
Map<String,String> mp = new Hashtable<>();
// key value 都不能为null，否则java.lang.NullPointerException
//        mp.put(null, null);
//        mp.put(null, "a");
//        mp.put("a", null);
mp.put("a", "a");
mp.forEach((key,val)->{
    System.out.println(String.format("<%s,%s>", key, val));
});
```

### 线程安全，基本上操作方法都加上了`synchronized`关键字

### put方法

```java
 /**
     * Maps the specified <code>key</code> to the specified
     * <code>value</code> in this hashtable. Neither the key nor the
     * value can be <code>null</code>. <p>
     *
     * The value can be retrieved by calling the <code>get</code> method
     * with a key that is equal to the original key.
     *
     * @param      key     the hashtable key
     * @param      value   the value
     * @return     the previous value of the specified key in this hashtable,
     *             or <code>null</code> if it did not have one
     * @exception  NullPointerException  if the key or value is
     *               <code>null</code>
     * @see     Object#equals(Object)
     * @see     #get(Object)
     */
    public synchronized V put(K key, V value) {
        // Make sure the value is not null
        if (value == null) {
            throw new NullPointerException();
        }

        // Makes sure the key is not already in the hashtable.
        Entry<?,?> tab[] = table;
        int hash = key.hashCode();
        int index = (hash & 0x7FFFFFFF) % tab.length;
        @SuppressWarnings("unchecked")
        Entry<K,V> entry = (Entry<K,V>)tab[index];
        for(; entry != null ; entry = entry.next) {
            if ((entry.hash == hash) && entry.key.equals(key)) {
                V old = entry.value;
                entry.value = value;
                return old;
            }
        }

        addEntry(hash, key, value, index);
        return null;
    }
```

## LinkedHashMap

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{
```

### 底层是HashMap,并维持一个双向链表

```java
 /**
     * HashMap.Node subclass for normal LinkedHashMap entries.
     */
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }

    private static final long serialVersionUID = 3801124242820219131L;

    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;

    /**
     * The iteration ordering method for this linked hash map: <tt>true</tt>
     * for access-order, <tt>false</tt> for insertion-order.
     *
     * @serial
     */
    final boolean accessOrder;

    // internal utilities

    // link at the end of list
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }

    // apply src's links to dst
    private void transferLinks(LinkedHashMap.Entry<K,V> src,
                               LinkedHashMap.Entry<K,V> dst) {
        LinkedHashMap.Entry<K,V> b = dst.before = src.before;
        LinkedHashMap.Entry<K,V> a = dst.after = src.after;
        if (b == null)
            head = dst;
        else
            b.after = dst;
        if (a == null)
            tail = dst;
        else
            a.before = dst;
    }
```

### 遍历的有序性

```java
Map<String,String> mp = new LinkedHashMap<>();
        mp.put(null, null);
        mp.put(null, "a");
        mp.put("b", null);
        mp.put("a", "a");
        Set<Map.Entry<String, String>> entries = mp.entrySet();
        Iterator<Map.Entry<String, String>> iteratorMap = entries.iterator();
        while (iteratorMap.hasNext()){
            Map.Entry<String, String> next = iteratorMap.next();
            System.out.println(next);
        }
        mp.forEach((key, val)->{
            System.out.println(String.format("%s=%s", key,val));
        });
/* output
null=a
b=null
a=a
*/
```

### 非线程安全，可以null key,null value等

## SortedMap（interface）

```java
public interface SortedMap<K,V> extends Map<K,V> {
```

## NavigableMap (interface)

```java
public interface NavigableMap<K,V> extends SortedMap<K,V> {
```

### 方法

```java
// 找到第一个比指定的key小的值
Map.Entry<K,V> lowerEntry(K key);

// 找到第一个比指定的key小的key
K lowerKey(K key);

// 找到第一个小于或等于指定key的值
Map.Entry<K,V> floorEntry(K key);

// 找到第一个小于或等于指定key的key
K floorKey(K key);

//  找到第一个大于或等于指定key的值
Map.Entry<K,V> ceilingEntry(K key);

K ceilingKey(K key);

// 找到第一个大于指定key的值
Map.Entry<K,V> higherEntry(K key);

K higherKey(K key);

// 获取最小值
Map.Entry<K,V> firstEntry();

// 获取最大值
Map.Entry<K,V> lastEntry();

// 删除最小的元素
Map.Entry<K,V> pollFirstEntry();

// 删除最大的元素
Map.Entry<K,V> pollLastEntry();

//返回一个倒序的Map
NavigableMap<K,V> descendingMap();

// 返回一个Navigable的key的集合，NavigableSet和NavigableMap类似
NavigableSet<K> navigableKeySet();

// 对上述集合倒序
NavigableSet<K> descendingKeySet();
```

## TreeMap(类)

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{
```

### 底层基于红黑树

红黑树能保证增、删、查等基本操作的时间复杂度为O(lgN)

```java
 /**
     * The comparator used to maintain order in this tree map, or
     * null if it uses the natural ordering of its keys.
     *
     * @serial
     */
    private final Comparator<? super K> comparator;

    private transient Entry<K,V> root;

    /**
     * The number of entries in the tree
     */
    private transient int size = 0;

    /**
     * The number of structural modifications to the tree.
     */
    private transient int modCount = 0;
```

### 不允许 null key， 可以 null value

### key排序，可以指定comparator

```java
Comparator comparator =  (a, b) -> {
    int vala = ((Integer)a);
    int valb = ((Integer)b);
    if(vala > valb){
        return -1;
    }
    if(vala < valb){
        return 1;
    }
    return 0;
};
Map<Integer,String> mp = new TreeMap(comparator);
mp.put(1, null);
mp.put(3, "a");
mp.put(2, "a");
mp.forEach((key, val)->{
    System.out.println(String.format("%d=%s", key,val));
});
```

## 