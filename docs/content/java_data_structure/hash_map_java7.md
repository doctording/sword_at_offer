---
title: "Java7 HashMap"
layout: page
date: 2020-03-08 11:00
---

[TOC]

# Java1.7 HashMap

## 底层结构(很纯粹的hash表+拉链法解决冲突:头插法)

```java
/**
* The table, resized as necessary. Length MUST Always be a power of two.
*/
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;

static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    /**
        * Creates new entry.
        */
    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

}
```

## 如何定位某个原始在table中的下标

hash函数求key的hash值（key的hashcode相关），然后 hash & (table.length - 1) 就是在table中的下标

```java
int hash = hash(key);
int i = indexFor(hash, table.length);

/**
* Returns index for hash code h.
*/
static int indexFor(int h, int length) {
    // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
    return h & (length-1);
}
```

## put方法

要点：

0. 如果table为空，进行初始化；key如果是null，则 hashcode 是0
1. 如何key存储，会返回oldValue
2. 存在hash冲突（即定位了table的同一个下标），则需要遍历Entry链表
3. 存在hash冲突，Entry链表使用头插法
4. 在插入节点前会先进行扩容判断，扩容为原来的两倍
5. 扩容完成后，再进行Entry插入操作：重新定位

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```

* 新增Entry方法

```java
void addEntry(int hash, K key, V value, int bucketIndex) {
    if ((size >= threshold) && (null != table[bucketIndex])) {
        resize(2 * table.length);
        hash = (null != key) ? hash(key) : 0;
        bucketIndex = indexFor(hash, table.length);
    }

    createEntry(hash, key, value, bucketIndex);
}
```

* 扩容是遍历旧table,然后拷贝到新table

```java
/**
* Transfers all entries from current table to newTable.
*/
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

## get

定位到table位置，然后遍历链表返回即可

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
    *
    * @see #put(Object, Object)
    */
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}
```
