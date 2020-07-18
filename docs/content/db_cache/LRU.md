---
title: "LRU"
layout: page
date: 2020-03-08 18:00
---

[TOC]

# LRU(Least Recently Used)

选择最近最久未使用的页面予以淘汰

leetcode 146题，eg

```java
LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得关键字 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得关键字 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/lru-cache
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。
```

* Java的一个求解如下

双向链表(带头节点) + Map

```java
// 普通双向链表,有一个初始的dummyNode
class DoubleList {
    private DoubleNode head, tail; // 头尾虚节点
    private int size; // 链表当前size

    public DoubleList() {
        head = new DoubleNode(0, 0);
        tail = new DoubleNode(0, 0);
        head.next = tail;
        tail.prev = head;
        size = 0;
    }

    // O(1)
    // 头插
    public void addFirst(DoubleNode x) {
        x.next = head.next;
        x.prev = head;
        head.next.prev = x;
        head.next = x;
        size++;
    }

    // O(1)
    // 删除链表中的 x 节点（x 一定存在）
    public void remove(DoubleNode x) {
        x.prev.next = x.next;
        x.next.prev = x.prev;
        size--;
    }

    // O(1)
    // 删除链表中最后一个节点，并返回该节点
    public DoubleNode removeLast() {
        if (tail.prev == head) {
            return null;
        }
        DoubleNode last = tail.prev;
        remove(last);
        return last;
    }

    public DoubleNode getFirst() {
        if (tail.prev == head) {
            return null;
        }
        return head.next;
    }


    public DoubleNode putFirst(DoubleNode node) {
        remove(node);
        addFirst(node);
        return head.next;
    }


    // 返回链表长度
    public int size() {
        return size;
    }

    // 双向链表的节点
    class DoubleNode {
        public int key, val;
        public DoubleNode next, prev;
        public DoubleNode(int k, int v) {
            this.key = k;
            this.val = v;
        }
    }
}

class LRUCache {

    DoubleList doubleList;
    Map<Integer, DoubleList.DoubleNode> mp;

    int cap;

    public LRUCache(int capacity) {
        cap = capacity;
        mp = new HashMap<>();
        doubleList = new DoubleList();
    }

    public int get(int key) {
        if (mp.containsKey(key)) {
            // 已存在，位置要重置
            DoubleList.DoubleNode node = mp.get(key);
            DoubleList.DoubleNode node1 = doubleList.putFirst(node);
            mp.put(key, node1);
            return node1.val;
        } else {
            // 不存在返回-1
            return -1;
        }
    }

    public void put(int key, int value) {
        if ( mp.containsKey(key) ) {
            // 已存在，位置要重置，且要修改value
            DoubleList.DoubleNode node = mp.get(key);
            doubleList.remove(node);

            node.val = value;
            doubleList.addFirst(node);
            mp.put(key, doubleList.getFirst());
        } else {
            if( doubleList.size() == cap) {
                DoubleList.DoubleNode lastNode = doubleList.removeLast();
                mp.remove(lastNode.key);
            }
            // 添加新节点
            DoubleList.DoubleNode newNode = doubleList.new DoubleNode(key,value);
            doubleList.addFirst(newNode);
            mp.put(key, doubleList.getFirst());
        }
    }
}
```
