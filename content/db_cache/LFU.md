---
title: "LRU"
layout: page
date: 2020-03-08 18:00
---

[TOC]

# LFU

Least Frequently Used ,最近最少使用算法

这个缓存算法使用一个计数器来记录条目被访问的频率。通过使用LFU缓存算法，最低访问数的条目首先被移除。这个方法并不经常使用，因为它无法对一个拥有最初高访问率之后长时间没有被访问的条目缓存负责。

![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcwMjIxMTA0NjU3NjM1?x-oss-process=image/format,png)

* leetcode java (460题)

<font color='red'>注意capacity=0的特殊情况，以及get,put 频率改变的情况</font>

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

    public boolean isEmpty() {
        return size == 0;
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

class LFUCache {

    // 频率对应的双向链表
    Map<Integer, DoubleList> freDoubleListMap;
    // key 对应的 频率
    Map<Integer, Integer> keyFreMap;
    // key 对应 在链表中的位置
    Map<Integer, DoubleList.DoubleNode> keyNodeMap;

    int cap;
    int minFre;

    public LFUCache(int capacity) {
        cap = capacity;
        freDoubleListMap = new HashMap<>();
        keyFreMap = new HashMap<>();
        keyNodeMap = new HashMap<>();
        minFre = 0;
    }

    public int get(int key) {
        if(cap == 0){
            return -1;
        }
        if (keyFreMap.containsKey(key)) {
            // 频率加1, 删除老的，添加新的
            DoubleList.DoubleNode node = keyNodeMap.get(key);
            int oldFre = keyFreMap.get(key);
            freDoubleListMap.get(oldFre).remove(node);

            // 最小频率不会骤然增加, 如下情况最小频率要加1，否则还保持最小频率
            if(minFre == oldFre && freDoubleListMap.get(oldFre).isEmpty()) {
                minFre ++;
            }

            int newFre = oldFre + 1;
            if (freDoubleListMap.containsKey(newFre)) {
                freDoubleListMap.get(newFre).addFirst(node);
            } else {
                // 新的双向链表
                DoubleList doubleList = new DoubleList();
                doubleList.addFirst(node);
                freDoubleListMap.put(newFre, doubleList);
            }
            keyNodeMap.put(key, freDoubleListMap.get(newFre).getFirst());
            keyFreMap.put(key, newFre);

            return node.val;
        } else {
            return -1;
        }
    }

    public void put(int key, int value) {
        if(cap == 0){
            return;
        }
        if (keyFreMap.containsKey(key)) {
            // 频率加1, 删除老的，添加新的
            DoubleList.DoubleNode node = keyNodeMap.get(key);
            int oldFre = keyFreMap.get(key);
            freDoubleListMap.get(oldFre).remove(node);

            // 最小频率不会骤然增加, 如下情况最小频率要加1，否则还保持最小频率
            if(minFre == oldFre && freDoubleListMap.get(oldFre).isEmpty()) {
                minFre ++;
            }

            node.val = value;
            int newFre = oldFre + 1;
            if (freDoubleListMap.containsKey(newFre)) {
                freDoubleListMap.get(newFre).addFirst(node);
            } else {
                // 新的双向链表
                DoubleList doubleList = new DoubleList();
                doubleList.addFirst(node);
                freDoubleListMap.put(newFre, doubleList);
            }
            keyNodeMap.put(key, freDoubleListMap.get(newFre).getFirst());
            keyFreMap.put(key, newFre);

        } else {
            // 容量满了，删除最小频率，最后访问的那个节点
            if (keyFreMap.size() == cap) {
                DoubleList.DoubleNode rmNode = freDoubleListMap.get(minFre).removeLast();
                keyNodeMap.remove(rmNode.key);
                keyFreMap.remove(rmNode.key);
                if(freDoubleListMap.get(minFre).isEmpty()){
                    minFre ++;
                }
            }
            DoubleList doubleList;
            if (freDoubleListMap.containsKey(1)) {
                doubleList = freDoubleListMap.get(1);
            } else {
                // new doubleList
                doubleList = new DoubleList();
            }
            // new node
            DoubleList.DoubleNode newNode = doubleList.new DoubleNode(key, value);
            doubleList.addFirst(newNode);

            freDoubleListMap.put(1, doubleList);
            keyNodeMap.put(key, freDoubleListMap.get(1).getFirst());
            keyFreMap.put(key, 1);
            minFre = 1;
        }
    }
}
```
