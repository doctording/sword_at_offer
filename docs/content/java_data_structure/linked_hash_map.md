---
title: "Java8 LinkedHashMap"
layout: page
date: 2020-08-08 11:00
---

[TOC]

# LinkedHashMap

HashMap + 双向链表

## 双向链表的定义

节点继承了`HashMap.Node<K,V>`,再加上双向链表的`before`节点和`after`节点

```java
//扩展一个双向链表的entry
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

//链表头
transient LinkedHashMap.Entry<K,V> head;

//链表尾
transient LinkedHashMap.Entry<K,V> tail;
```

## 使用和说明

LinkedHashMap 的特性， 每个节点间由一个before引用 和 after 引用串联起来成为一个双向链表。链表节点按照访问时间进行排序，最近访问过的链表放在链表尾。

```java
// 10 是初始大小，0.75 是装载因子，true 是表示按照访问时间排序
HashMap<Integer, Integer> m = new LinkedHashMap<>(10, 0.75f, true);
m.put(3, 11);
m.put(1, 12);
m.put(5, 23);
m.put(2, 22);

m.put(3, 26);
m.get(5);

for (Map.Entry e : m.entrySet()) {
    System.out.println(e.getKey());
}
/*
输出的结果就是 1，2，3，5
*/
```

## get方法(类似LRU)

会将当前被访问到的节点e，移动至内部的双向链表的尾部。

```java
/**
    * Returns the value to which the specified key is mapped,
    * or {@code null} if this map contains no mapping for the key.
    *
    * <p>More formally, if this map contains a mapping from a key
    * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
    * key.equals(k))}, then this method returns {@code v}; otherwise
    * it returns {@code null}.  (There can be at most one such mapping.)
    *
    * <p>A return value of {@code null} does not <i>necessarily</i>
    * indicate that the map contains no mapping for the key; it's also
    * possible that the map explicitly maps the key to {@code null}.
    * The {@link #containsKey containsKey} operation may be used to
    * distinguish these two cases.
    */
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```
