---
title: "迭代器模式"
layout: page
date: 2020-05-11 00:00
---

[TOC]

# 迭代器模式

如：集合中常见的迭代器

定义：迭代器模式提供了一种方法顺序访问一个聚合对象中的各个元素，而又无需暴露该对象的内部实现，这样既可以做到不暴露集合的内部结构，又可让外部代码透明地访问集合内部的数据

比如LeetCode:`173. 二叉搜索树迭代器`,可以内部各种方式实现，但是对外就是如下两个方法

```java
public int next()

public boolean hasNext()
```

* ac

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class BSTIterator {
    // O(h)的存储
    Stack<TreeNode> sta;

    private void left2Stack(TreeNode root){
        while (root != null){
            sta.push(root);
            root = root.left;
        }
    }

    public BSTIterator(TreeNode root) {
        sta = new Stack<>();
        left2Stack(root);
    }

    /** @return the next smallest number */
    public int next() {
        TreeNode node = sta.pop();
        if(node.right != null){
            left2Stack(node.right);
        }
        return node.val;
    }

    /** @return whether we have a next smallest number */
    public boolean hasNext() {
        return ! sta.isEmpty();
    }
}

/**
 * Your BSTIterator object will be instantiated and called as such:
 * BSTIterator obj = new BSTIterator(root);
 * int param_1 = obj.next();
 * boolean param_2 = obj.hasNext();
 */
```
