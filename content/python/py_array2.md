---
title: "python二维数组"
layout: page
date: 2018-05-30 00:00
---

[TOC]

# 一维 list

```python
# -*- coding:utf-8 -*-

li = [0] * 5

print li # [0, 0, 0, 0, 0]

li[0] = 1
li[1] = 2

print li # [1, 2, 0, 0, 0]
```

# 二维

list的每一项都是list
```
[
    [ ],
    [ ]
]
```

```python
# -*- coding:utf-8 -*-

li = [0] * 5
li[0] = 1
li[1] = 2

li2 = [li] * 3

print li2

li2[1][2] = 5  # 事实上改变了li2[0][2], li2[1][2], li2[2][2]

print li2
'''
[1, 2, 5, 0, 0],
[1, 2, 5, 0, 0],
[1, 2, 5, 0, 0]]
'''
```

问题： **matrix = [array] * 3操作中，只是创建3个指向array的引用，所以一旦array改变，matrix中3个list也会随之改变。**

## 二维定义1

```python
# -*- coding:utf-8 -*-
li2 = [ [0 for i in range(5)] for i in range(3)]

print li2

li2[1][2] = 5

print li2
```

## 二维定义2

```python
# -*- coding:utf-8 -*-

li2 = []
for i in range(3):
    tmp = [0] * 5
    li2.append(tmp)

print li2

li2[1][2] = 5
print li2
```
