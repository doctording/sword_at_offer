---
title: "剑指Offer解题思路(全)"
layout: page
date: 2018-05-30 00:00
---

[TOC]

# 1 二维数组中的查找

* 有序查找
* 二分

# 2 替换空格

* 先统计增大的空间，然后从后往前替换

# 3 从尾到头打印链表

* 头插法将链表反转一下，然后遍历（这种方式会改变链表的结构）

* 当链表只读时，采用递归的方法（栈）来操作，就可以实现链表数据的反转了

# 4 重建二叉树

* 这类题目都采用递归，属于二叉树基础题

# 5 用两个栈实现队列

* 保持一个stack为空，通过两个栈转储，实现数据进出是队列形式的

# 6 旋转数组的最小数字

* 头尾往中间逼近的方法,直接遍历一遍，O(n)的做法，显然太low

* 二分查找的方法，不过有特殊情况，比如 [1 0 1 1 1] ，[1 1 1 0 1] 这种，需要单独顺序遍历

# 7 斐波那契数列 (*import*)

* 最直接的递归/动归， O(2^n), n的指数次方增长

* 利用变量,非递归，时间复杂度O(n)

* 快速幂的做法  f(n) = f(n - 1) + f(n - 2), 二阶的快速幂，时间复杂度O(logn),需要推算一下矩阵公式

# 8 跳台阶

* 同 斐波那契数列

# 9 变态跳台阶

数学公式推导, 同斐波那契数列

# 10 矩形覆盖

* 同 斐波那契数列

# 11 二进制中1的个数

* 经典问题，采用位运算`n & (n-1)`得到最右边的一个1

# 12 数值的整数次方

* 快速幂方法

* 边界，符号，double/float判零问题

* 位运算比除法，取模等效率高

# 13 调整数组顺序使奇数位于偶数前面 （*important*）

* 注意题目要求无序和有序的区别

* 无序
	1. 快排思路，维持头/尾两个指针，不断的交换到相遇,时间O(n), 空间O(1)

* 要求保持数据相对位置不变(无序的最优解不再满足需求，注意快排是不稳定的)
	1. 直接插入排序， 时间复杂度O(n^2), 空间复杂度O(1)
	2. 归并排序， 时间复杂度O(nlogn), 空间复杂度O(n) ?；归并排序是稳定的排序，归并排序需要O(n)的辅助空间
	3. 利用复杂空间，空间复杂度O(n), 时间复杂度O(n)（直接遍历一遍）

# 14 链表中倒数第k个结点

* 两个指针，保持距离k；那么一个链表尾部，一个就是倒数k

# 15 反转链表

* 直接遍历，头插法
* 递归法

# 16 合并两个排序的链表

* 每个链表一个游标，边比较/边连接，最后多的直接连接

* 递归思路，因为链表总是有序的，对两个链表头节点处理后，剩下的操作更前面的一样的

# 17 树的子结构（*important*）

* 注意`子结构`和`子树`的区别

**子树**的意思是包含了一个结点，就得包含这个结点下的所有节点，一棵大小为n的二叉树有n个子树，就是分别以每个结点为根的子树。

**子结构**的意思是包含了一个结点，可以只取左子树或者右子树，或者都不取。

* 子树问题 可以采用 序列化方法，而子结构不能采用序列化方法

此题 递归求解，要么根，要么与左子树匹配，要么与右子树匹配，递归需要注意

# 18 二叉树的镜像

递归： 子树先镜像，然后上层根再镜像

# 19 顺时针打印矩阵

* 一圈一圈的打印，可以封装成一个方法

# 20 包含min函数的栈

* 辅助栈方法
	1. 维持两个栈，一个普通栈，一个存放min的栈，两个栈对应起来

* 不用辅助栈
	1. push(int elem)函数在栈中压入当前元素与当前栈中最小元素的差值，然后通过比较当前元素与当前栈中最小元素大小，并将它们中间的较小值压入。
	2. pop()函数执行的时候，先pop出栈顶的两个值，这两个值分别是当前栈中最小值min和最后压入的元素与栈中最小值的差值diff

# 21 栈的压入、弹出序列

* 直接利用STL stack等结构, 去模拟栈的压入、弹出过程

# 22 从上往下打印二叉树

* 直接层次遍历

# 23 二叉搜索树的后序遍历序列

* 判断后续是否合法
* 问清楚有没有重复数字，搜索树是怎样的排序
* 根据后序：左 右 根的顺序，递归判断是否满足二叉树的大小关系

# 24 二叉树中和为某一值的路径

DFS 或 BFS

# 25 复杂链表的复制 (*important*)

next 和 random

首先`A B C D` 变成  `A A' B B' C C' D D'`

通过1，则有：`A'.random` = `A.random.next`

然后遍历改变`random`, 遍历改变`next`，最后输出即可

# 26 二叉搜索树与双向链表

* 二叉树的非递归遍历(可以使用辅助栈)

# 27 字符串的排列

* 递归法，需要判断重复的串
* 可排序，然后判断，也可以采用set等数据结构

# 28 数组中出现次数超过一半的数字

* cur, count，（每次取两个不一样的数删除)　最后验证

# 29 最小的K个数

* 快排思路

* 堆排序思路 O(nlogk)时间复杂度，适合处理海量数据,采用红黑树（stl中的set,multiset都是基于红黑树的）

# 30 连续子数组的最大和

* 动态规划

# 31 整数中1出现的次数（从1到n整数中1出现的次数）

* 遍历1到n,对每个数采用mod 10 取得1的个数，时间复杂度达到O(nlogn)

* 找规律的方法，时间复杂度达到O(logn), 参考如下的讲解
http://blog.csdn.net/yi_afly/article/details/52012593

# 32 把数组排成最小的数

* 两两组合排序，注意0

# 33 丑数

# 34 第一个只出现一次的字符

* 直接求解：每个字符都与后面的字符比较:空间O(1), 时间(O(n^2)

* hash记录，空间O(n), 时间O(n)

# 35 数组中的逆序对（*important*）

* 直接求解： 时间复杂度是O(n^2)

* 归并排序思路：O(n)的空间，时间复杂度为O(nlogn)

# 36 两个链表的第一个公共结点

* 单链表是否存在环
 走一步，走两步,进一步可以找到第一个公共节点

* 直接利用map

* 将两个链表先处理成一样长的，然后再判断

# 37 数字在排序数组中出现的次数

* 二分查找，然后左右扩散开统计

# 38 二叉树的深度

* 递归

* 非递归实现(可以层次遍历)

# 39 输入一棵二叉树，判断该二叉树是否是平衡二叉树

1. 每个节点都采用求深度的方法，这样遍历求解，重复计算量太大

2. 后序遍历，递归判断每个节点是否平衡，因为先访问了左右子树，后访问根，所以需要用变量保存每个节点的深度

# 40 数组中只出现一次的数字

一个整型数组里除了两个数字之外，其他的数字都出现了两次，利用数的性质，异或运算

# 41 和为S的连续正数序列（**important**）

* 数学公式

# 42 和为S的两个数字

* 排序(看题目是否是有序的) + 二分查找， 时间O(nlogn)，空间O(1)

* hash；有序的话，前后逼近

扩展：**寻找和为定值的任意多个数**，递归/动归

# 43 左旋转字符串

* 三步翻转

* 改进三步翻转，用块交换

# 44 翻转单词顺序列

* 同 43 左旋转字符串

# 45 扑克牌顺子

* 统计个数为5，大小王个数
* 排序，逐一判断，减去大小王数目

# 46 孩子们的游戏(圆圈中最后剩下的数)

* 约瑟夫环

`f(N,M)=(f(N−1,M)+M)%N`

`f(N,M)`表示，`N`个人报数，每报到`M`时杀掉那个人，最终胜利者的编号
`f(N−1,M)`表示，`N-1`个人报数，每报到M时杀掉那个人，最终胜利者的编号

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/solved_by_java/imgs/joseph.png)

`f(11, 3) = 7`, 可以看到`7`不断的往前面移动`3`即每杀掉一个人，其实就是把这个数组(环)向前移动了M位.

假设：`n`个人，最后胜利者是`Pn`位置；则经过一轮以后:变成`n-1`个人，最后胜利者位置是`Pn-1`
那么显然有： `Pn = ( Pn-1 + k ) % n`
所以有：`f(N,M) = ( f(N−1, M ) + M ) % N`

# 47 求1+2+3+...+n

* 数学公式
* 递归

考虑并发，并行流

# 48 不用加减乘除做加法

* 位运算模拟

异或得到想加结果（不进位）
与然后左移得到进位结果

```java
5:  0101
7:  0111

  0101				 0101
^ 0111			   & 0111
= 0010 （加的结果）  = 0101  << 1 : 1010(进位情况)

  0010				 0010
^ 1010			   & 1010
= 1000 （加的结果）  = 0010  << 1 : 0100(进位情况)

  1000				 1000
^ 0100			   & 0100
= 1100 （加的结果）  = 0000  << 1 : 0000(进位情况)

最后结果： 1100 即 12
```

# 49 把字符串转换成整数 (**important**)

* 注意最大负整数，最大正整数等特殊情况

```js
2147483647
-2147483648
```

# 50 数组中重复的数字

* 时间复杂度/空间复杂度考虑

* 二分法思路
* hash辅助

# 51 构建乘积数组

* 构建左部分乘积 和 右部分乘积序列

# 52 正则表达式匹配(**important**)

* 递归 ？，.

# 53 表示数值的字符串

* 各种特殊情况

# 54 字符流中第一个不重复的字符

# 55 链表中环的入口结点

* 链表环的判断

# 56 删除链表中重复的结点

* 链表删除节点模拟, 注意各种特殊情况

1. 使用map遍历一边获取要删的原始值，再遍历一遍真正取删除节点，构造返回
2. 递归
3. 加辅助头节点,一次遍历

# 57 二叉树的下一个结点， 中序遍历顺序的下一个结点并且返回

# 58 对称的二叉树

* 对称二叉树概念：将一棵二叉树沿着根节点对折，如果两棵子树完全重合（对称节点要么都为null，要么数据域完全相等），那么该二叉树是一个对称二叉树。

* 1 递归求解 相当于在判断root->left, root->right 两棵树 是否对折相等

* 2 非递归，用两个队列，维持root->left, root->right,节点入队列顺序不一样，判断完全

# 59 按之字形顺序打印二叉树

* 按照层次遍历，加上标记, 奇数层(偶数层)左reverse操作（reverse不可取，太low）

* 偶数层栈，奇数层栈，利用栈和二叉树的性质处理（推荐）

# 60 把二叉树打印成多行

# 61 序列化二叉树

* DFS
* BFS

# 62 二叉搜索树的第k个结点

* 非递归遍历(使用辅助栈和不使用辅助栈),中序遍历

# 63 数据流中的中位数

对于有限的数集，可以通过把所有观察值高低排序后找出正中间的一个作为中位数。如果观察值有偶数个，通常取最中间的两个数值的平均数作为中位数。

* 大顶堆和小顶堆。插入的时间效率是O(logn)，找中位数的时间效率是O(1)。

# 64 滑动窗口的最大值

* 利用滑动窗口的性质，双端队列处理

* 两个栈实现`pop-push-max`都是O(1)的空间换时间

# 65 矩阵中的路径

* DFS

# 66 机器人的运动范围

* DFS

# 67 剪绳子

* 动态规划