---
title: "python基础知识"
layout: page
date: 2018-05-30 00:00
---

[TOC]

# python基础

## 数据类型

Python的`inspect`模块包含了大量的与反射、元数据相关的工具函数

```python
from inspect import isfunction

  print type("aa") == str
  print type([1,2]) == list
  print type({"id":1}) == dict
  print type((1,)) == tuple
  print type(1.23) == float
  print type(1) == int
  a = None
  print a == None
  print isfunction(test1)
```

### Numbers 数字

	int, long, float, double

### String 字符串

读取顺序

	* 从左到右索引默认0开始的，最大范围是字符串长度少1

	* 从右到左索引默认-1开始的，最大范围是字符串开头

转义字符用反斜杠 `\`

操作符 & 格式化

```
+
*
[]
[:]
in
not in
r/R
%   // 格式化
```

常用方法

```python
string.find(str, beg=0, end=len(string))

string.format()

string.replace(str1, str2, num=string.count(str1))

string.count(str, beg=0, end=len(string))
```

python三引号\```允许一个字符串跨多行，字符串中可以包含换行符、制表符以及其他特殊字符。

### List 列表

用```[]```表示

#### 基础操作

```python
len(list)
max(list)
min(list)
list.reverse()
list.sort(cmp=None, key=None, reverse=False)
```

#### 增

```python
list.append(obj)
list.insert(index, obj)
```

#### 删

```python
list.pop([index=-1])
list.remove(obj)
```

#### 查找

```python
list.count(obj) # 统计某个元素在列表中出现的次数
list[2] # 读取列表中第三个元素
list[-2] # 读取列表中倒数第二个元素
list[1:] # 从第二个元素开始截取列表
list.index(obj) # 从列表中找出某个值第一个匹配项的索引位置
```

#### 遍历（三种方法）

```python
# -*- coding:utf-8 -*-

if __name__ == '__main__':
    li = [{"id": 1},  "name", 1]

    print "==="
    for i in range(len(li)):
        print i, li[i]

    print "==="
    for val in li:
        print li.index(val), val

    print "==="
    for i, val in enumerate(li):
        print i, val

    print "==="
    for i, val in enumerate(li,1):
        print i, val
```

### Tuple 元组

### Dictionary 字典

	用```{}```表示, ```key:value```, 通过key来读取,而非偏移

#### dict遍历

```python
# -*- coding:utf-8 -*-

if __name__ == '__main__':

    di = {
        "a": 1,
        "b": "time",
        "c": [1, 2, 3],
    }

    for k, v in di.items():
        print(k, v)

    for k in di.keys():
        print(k, di[k])

    for v in di.values():
        print(v)
```

#### dict取值注意点

```python
# -*- coding:utf-8 -*-

if __name__ == '__main__':

    di = {
        "a": 1,
        "b": "time",
        "c": [1, 2, 3],
    }

    # print(di["d"]) # 报错

    # 先判断 再取
    print("a" in di)  # True
    print("d" in di)  # False
    print(1 in di.values())  # True

    if "a" in di.keys():
        print(di["a"])

    # depressed
    if di.has_key("a"):
        print(di["a"])

    # get方法，取不到，取默认值
    print(di.get("d", "empty"))
```

### 其它类型

    True，False，None

#### False的情况

1. False
2. None
3. 所有值为零的数
    * 0（整数）
    * 0.0（浮点数）
    * 0L （长整数）
    * 0.0+0.0j （复数）
4. “”（空字符串）
5. [] （空列表）
6. () （空元组）
7. {} ｛空字典｝

## 条件语句

* 注意条件后面的冒号```:```，else后面的冒号```:```

```python
if 判断条件：
    执行语句……
else：
    执行语句……
```

```python
if 判断条件1:
    执行语句1……
elif 判断条件2:
    执行语句2……
elif 判断条件3:
    执行语句3……
else:
    执行语句4……
```

## for, while

控制语句 | 描述
| :- | :- |
break | 在语句块执行过程中终止循环，并且跳出整个循环
continue | 在语句块执行过程中终止当前循环，跳出该次循环，执行下一次循环。
pass | pass是空语句，是为了保持程序结构的完整性。

* 注意冒号```:```不要漏掉

examples :

```python
for letter in 'Python':
   print '当前字母 :', letter

fruits = ['banana', 'apple',  'mango']
for fruit in fruits:
   print '当前水果 :', fruit
```

```python
fruits = ['banana', 'apple',  'mango']
for index in range(len(fruits)):
   print '当前水果 :', fruits[index]
```

```python
for num in range(10,20):  # 迭代 10 到 20 之间的数字
   for i in range(2,num): # 根据因子迭代
      if num%i == 0:      # 确定第一个因子
         j=num/i          # 计算第二个因子
         print '%d 等于 %d * %d' % (num,i,j)
         break            # 跳出当前循环
   else:                  # 循环的 else 部分
      print num, '是一个质数'
```

```python
i = 2
while(i < 100):
   j = 2
   while(j <= (i/j)):
      if not(i%j): break
      j = j + 1
   if (j > i/j) : print i, " 是素数"
   i = i + 1
```

## 成员运算符(in,not in)

```python
in
not in
```

字符串，列表或元组等，`x in y`: x在y序列中, 如果x在y序列中返回`True`

## 逻辑运算符(and,or,not)

```python
and
or
not
```

运算符	| 逻辑表达式	| 描述	| 实例
-|-|-|-
and	| x and y|	布尔"与" - 如果 x 为 False，x and y 返回 False，否则它返回 y 的计算值。	| (a and b) 返回 20。
or	| x or y	|布尔"或"	- 如果 x 是非 0，它返回 x 的值，否则它返回 y 的计算值。	| (a or b) 返回 10。
not	| not x|	布尔"非" - 如果 x 为 True，返回 False 。如果 x 为 False，它返回 True。	|not(a and b) 返回 False

## 类 Class

使用 class 语句来创建一个新类，class 之后为类的名称并以冒号结尾，简单的类例子如下：

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

class Employee:
   '所有员工的基类'
   empCount = 0

   def __init__(self, name, salary):
      self.name = name
      self.salary = salary
      Employee.empCount += 1

   def displayCount(self):
     print "Total Employee %d" % Employee.empCount

   def displayEmployee(self):
      print "Name : ", self.name,  ", Salary: ", self.salary
```

### 实例化对象

```python
# 创建 Employee 类的第一个对象
emp1 = Employee("Zara", 2000)
# 创建 Employee 类的第二个对象
emp2 = Employee("Manni", 5000)
```

### 访问属性

使用点号 . 来访问对象的属性。使用如下类的名称访问类变量:

```python
emp1.displayEmployee()
emp2.displayEmployee()
print "Total Employee %d" % Employee.empCount
```

### python对象销毁(垃圾回收-引用计数)

Python 使用了引用计数这一简单技术来跟踪和回收垃圾

```python
a = 40      # 创建对象  <40>
b = a       # 增加引用， <40> 的计数
c = [b]     # 增加引用.  <40> 的计数

del a       # 减少引用 <40> 的计数
b = 100     # 减少引用 <40> 的计数
c[0] = -1   # 减少引用 <40> 的计数
```

### self代表类的实例，而非类

类的方法与普通的函数只有一个特别的区别——它们必须有一个额外的第一个参数名称, 按照惯例它的名称是 self。

### #

## #