---
title: "Java8 HashTable"
layout: page
date: 2020-03-08 12:00
---

[TOC]

# HashTable

* 数组 + 链表（拉链）

* Hashtable 继承于Dictionary，实现了Map、Cloneable、java.io.Serializable接口

* Hashtable 的函数都是同步(synchronized)的，这意味着它是线程安全的。它的key、value都不可以为null。此外，Hashtable中的映射不是有序的

## 链表结构

```java
/**
     * Hashtable bucket collision list entry
     */
    private static class Entry<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Entry<K,V> next;

        protected Entry(int hash, K key, V value, Entry<K,V> next) {
            this.hash = hash;
            this.key =  key;
            this.value = value;
            this.next = next;
        }

        @SuppressWarnings("unchecked")
        protected Object clone() {
            return new Entry<>(hash, key, value,
                                  (next==null ? null : (Entry<K,V>) next.clone()));
        }

        // Map.Entry Ops

        public K getKey() {
            return key;
        }

        public V getValue() {
            return value;
        }

        public V setValue(V value) {
            if (value == null)
                throw new NullPointerException();

            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;

            return (key==null ? e.getKey()==null : key.equals(e.getKey())) &&
               (value==null ? e.getValue()==null : value.equals(e.getValue()));
        }

        public int hashCode() {
            return hash ^ Objects.hashCode(value);
        }

        public String toString() {
            return key.toString()+"="+value.toString();
        }
    }
```

## put方法

```java
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    // key（如果为 null，计算哈希值时会抛异常），value 不允许为 null
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    // 计算对应的桶位置
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    // 如果桶位置上对应的链表不为 null，则遍历该链表
    for(; entry != null ; entry = entry.next) {
        // key 重复，value 替换，返回老的 value
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    // 添加新的键值对
    addEntry(hash, key, value, index);
    // 添加成功返回 null
    return null;
}
```

```java
private void addEntry(int hash, K key, V value, int index) {
    modCount++;

    Entry<?,?> tab[] = table;
    // 延迟 rehash？先判断是否需要扩容再 count++
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();

        tab = table;
        hash = key.hashCode();
        // 扩容后，新的键值对对应的桶位置可能会发生变化，因此要重新计算桶位置
        index = (hash & 0x7FFFFFFF) % tab.length;
    }

    // Creates the new entry.
    @SuppressWarnings("unchecked")
            // 获取桶位置上的链表
    Entry<K,V> e = (Entry<K,V>) tab[index];
    // 头插法插入键值对
    tab[index] = new Entry<>(hash, key, value, e);
    // count++
    count++;
}
```

## rehash

```java
protected void rehash() {
    // 获取老哈希表容量
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;

    // overflow-conscious code
    // 新哈希表容量为原容量的 2 倍 + 1，与 HashMap 不同
    int newCapacity = (oldCapacity << 1) + 1;
    // 容量很大特殊处理
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            // Keep running with MAX_ARRAY_SIZE buckets
            return;
        newCapacity = MAX_ARRAY_SIZE;
    }
    // 初始化新哈希表数组
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

    modCount++;
    // 重置扩容阈值，与 HashMap 不同，HashMap 直接把 threshold 也扩大为原来的两倍
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    // 重置哈希表数组
    table = newMap;

    // 从底向上进行 rehash
    for (int i = oldCapacity ; i-- > 0 ;) {
        // 获取旧哈希表对应桶位置上的链表
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            // 链表
            Entry<K,V> e = old;
            // 重置继续遍历
            old = old.next;

            // 获取在新哈希表中的桶位置，键值对逐个进行 rehash
            // HashMap 会构造一个新的链表然后整个链表进行 rehash
            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            // 头插法 rehash
            e.next = (Entry<K,V>)newMap[index];
            // 自己做头节点
            newMap[index] = e;
        }
    }
}
```
