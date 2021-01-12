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
LRUCache cache = new LRUCache( 2 );  /* 缓存容量 */ 

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

## 自己构造双向链表(带头节点) + Map

![](../../content/db_cache/imgs/lru.png)

* <font color='red'>注意更新链表的同时，要更新Map内容(更新，删除，新增操作不能漏掉)</font>

```java
// 普通双向链表,有一个初始的dummyNode
class DoubleList{
    class DoubleNode{
        public int key;
        public int val;
        public DoubleNode pre;
        public DoubleNode next;
        public DoubleNode(int k, int v) {
            this.key = k;
            this.val = v;
        }
    }

    DoubleNode head;
    int size;

    public DoubleList(){
        head = new DoubleNode(-1,-1);
        DoubleNode tail = head;
        head.next = tail;
        tail.pre = head;
        size = 0;
    }

    DoubleNode getHead(){
        if(size == 0){
            return null;
        }
        return head.next;
    }

    void addHead(DoubleNode node){
        head.next.pre = node;
        node.next = head.next;
        head.next = node;
        node.pre = head;
        size ++;
    }

    void removeNode(DoubleNode node){
        node.next.pre = node.pre;
        node.pre.next = node.next;
        size --;
    }

    DoubleNode removeTailNode(){
        if(isEmpty()){
            return null;
        }
        DoubleNode node = head.pre;
        node.next.pre = node.pre;
        node.pre.next = node.next;
        size --;
        return node;
    } 

    int getSize(){
        return size;
    }

    boolean isEmpty(){
        return size == 0;
    }
  
    public DoubleNode newDoubleNode(int key, int val){
        return new DoubleNode(key, val);
    }
}

class LRUCache {

    DoubleList doubleList; // 最近访问的加入到头部即可
    Map<Integer, DoubleList.DoubleNode> mp;
    int cap;

    public LRUCache(int capacity) {
        doubleList = new DoubleList();
        cap = capacity;
        mp = new HashMap();
    }

    public int get(int key) {
        if(mp.containsKey(key)){
            DoubleList.DoubleNode node = mp.get(key);
            int val = node.val;

            doubleList.removeNode(node);
            doubleList.addHead(node);
            mp.put(key, doubleList.getHead());

            return val;
        }
        return -1;
    }

    public void put(int key, int value) {
        if(mp.containsKey(key)){
            DoubleList.DoubleNode node = mp.get(key);

            doubleList.removeNode(node);

            node.val = value;
            doubleList.addHead(node);
            mp.put(key, doubleList.getHead());
        }else{
            if(doubleList.getSize() == cap){
                DoubleList.DoubleNode tailNode = doubleList.removeTailNode();
                mp.remove(tailNode.key);
            }
            DoubleList.DoubleNode node = doubleList.newDoubleNode(key, value);
            doubleList.addHead(node);
            mp.put(key, doubleList.getHead());
        }
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

插入之后的`removeEldestEntry`方法(默认返回:false)，删除最久远的访问元素，LinkedHashMap默认直接返回false,即不会进行删除操作，可以重写该方法

```java
/**
* Returns <tt>true</tt> if this map should remove its eldest entry.
* This method is invoked by <tt>put</tt> and <tt>putAll</tt> after
* inserting a new entry into the map.  It provides the implementor
* with the opportunity to remove the eldest entry each time a new one
* is added.  This is useful if the map represents a cache: it allows
* the map to reduce memory consumption by deleting stale entries.
*
* <p>Sample use: this override will allow the map to grow up to 100
* entries and then delete the eldest entry each time a new entry is
* added, maintaining a steady state of 100 entries.
* <pre>
*     private static final int MAX_ENTRIES = 100;
*
*     protected boolean removeEldestEntry(Map.Entry eldest) {
*        return size() &gt; MAX_ENTRIES;
*     }
* </pre>
*
* <p>This method typically does not modify the map in any way,
* instead allowing the map to modify itself as directed by its
* return value.  It <i>is</i> permitted for this method to modify
* the map directly, but if it does so, it <i>must</i> return
* <tt>false</tt> (indicating that the map should not attempt any
* further modification).  The effects of returning <tt>true</tt>
* after modifying the map from within this method are unspecified.
*
* <p>This implementation merely returns <tt>false</tt> (so that this
* map acts like a normal map - the eldest element is never removed).
*
* @param    eldest The least recently inserted entry in the map, or if
*           this is an access-ordered map, the least recently accessed
*           entry.  This is the entry that will be removed it this
*           method returns <tt>true</tt>.  If the map was empty prior
*           to the <tt>put</tt> or <tt>putAll</tt> invocation resulting
*           in this invocation, this will be the entry that was just
*           inserted; in other words, if the map contains a single
*           entry, the eldest entry is also the newest.
* @return   <tt>true</tt> if the eldest entry should be removed
*           from the map; <tt>false</tt> if it should be retained.
*/
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```
