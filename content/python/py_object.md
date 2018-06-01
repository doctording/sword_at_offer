---
title: "python可变对象与不可变对象,引用"
layout: page
date: 2018-05-30 00:00
---

[TOC]

# python基础

Python在heap中分配的对象分成两类：**可变对象**(对象的内容可变)和**不可变对象**(对象内容不可变)。

* 不可变（immutable）：字符串(string)、数值型(number)、元组(tuple)

* 可变（mutable）：字典型(dictionary)、列表型(list)

```python
i = 1   # i -> [地址1]

i += 1  # [地址1] 被回收
        # i -> [地址2]
```

例如

```python
i = 1
print id(i)

i += 1
print id(i) # 不同上

li = [1]
print id(li)

li += [2]
print id(li) # 同上
```

li改变了内容，地址却没有改变。，[2]这个内存是需要重新开辟的

---

* 函数调用后改变了list,而没有改变int

```python
# -*- coding:utf-8 -*-

def change(a_int, b_list):
    a_int += 1
    b_list += [1]

a = 2
b = [2]

print a, b # 2 [2]
change(a, b)
print a, b # 2, [2, 1]
```

---

如下图，**在函数中改变引用是无效的**；python函数参数传递：不是所谓传指，也不是所谓的传引用

<div align="center">
<img  style="border:1px solid #000;"  src="https://raw.githubusercontent.com/doctording/sword_at_offer/master/imgs/py_01.png" 
height="400" width="340" >
<img style="border:1px solid #000;"   src="https://raw.githubusercontent.com/doctording/sword_at_offer/master/imgs/py_02.png" height="400" width="340" >
</div>

---