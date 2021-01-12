---
title: "python可变对象与不可变对象,引用"
layout: page
date: 2018-05-30 00:00
---

[TOC]

# 可变对象与不可变对象, 引用

Python在heap中分配的对象分成两类：**可变对象**(对象的内容可变)和**不可变对象**(对象内容不可变)。

* 不可变（immutable）：字符串(string)、数值型(number)、元组(tuple)

该对象所指向的内存中的值不能被改变。当改变某个变量时候，由于其所指的值不能被改变，相当于把原来的值复制一份后再改变，这会开辟一个新的地址，变量再指向这个新的地址。

* 可变（mutable）：字典型(dictionary)、列表型(list)

对象所指向的内存中的值可以被改变。变量（准确的说是引用）改变后，实际上是其所指的值直接发生改变，并没有发生复制行为，也没有开辟新的出地址，通俗点说就是原地改变。

* `id()`函数用于获取对象的内存地址

```python
i = 1   # i -> [地址1]

i += 1  # [地址1] 被回收
        # i -> [地址2]
```

例如

```python
i = 1
print(id(i))

i += 1
print(id(i))  # 不同上

li = [1]
print(id(li))

li += [2]
print(id(li))  # 同上
```

li改变了内容，地址却没有改变。[2]这个内存是需要重新开辟的, 加到了[1]的后面

---

* **可变类型**传递的类似【引用】，**不可变类型**传递的类似【内容】。

函数调用后改变了list, 而没有改变int

```python
# -*- coding:utf-8 -*-


def change(a_int, b_str, c_list):
    print("func start")
    a_int += 1
    b_str += "c"
    c_list += [1]
    # a b 地址改变，c地址不变
    print(id(a_int), id(b_str), id(c_list))
    print("func end")

if __name__ == '__main__':
    a = 2
    b = "ab"
    c = [2]

    print(id(a), id(b), id(c))
    print(a, b, c)  # (2, 'ab', [2])

    change(a, b, c)

    print(id(a), id(b), id(c))
    print(a, b, c)  # (2, 'ab', [2, 1])

```

---

如下图，**在函数中改变[可变对象的引用]是无效的**， **改变[可变对象的内容]是OK的**

<div align="center">
<img  style="border:1px solid #000;"  src="../../content/python/imgs/py_01.png"
height="400" width="340" >
<img style="border:1px solid #000;"   src="../../content/python/imgs/py_02.png" height="400" width="340" >
</div>
