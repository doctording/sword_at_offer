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

假设变量 a 为 10, b为 20:

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

### 类@property,@classmethod,staticmethod

#### @property

装饰器（decorator）可以给函数动态加上功能,对于类的方法，装饰器一样起作用。Python内置的@property装饰器就是负责把一个方法变成属性调用的

```python
class Student(object):
    def __init__(self, name, score):
        self.name = name
        self.__score = score

    @property
    def score(self):
        return self.__score

    @score.setter
    def score(self, score):
        if score < 0 or score > 100:
            raise ValueError('invalid score')
        self.__score = score

if __name__ == '__main__':
    stu1 = Student("sun", 60)
    stu1.score = 101
```

#### @classmethod & @staticmethod

[详细参考](https://blog.csdn.net/yxwb1253587469/article/details/52250719)

1. classmethod：类方法
2. staticmethod：静态方法

在python中，静态方法和类方法都是可以通过类对象和类对象实例访问。但是区别是：

* @classmethod 是一个函数修饰符，它表示接下来的是一个类方法，而对于平常我们见到的则叫做实例方法。 类方法的第一个参数cls，而实例方法的第一个参数是self，表示该类的一个实例。

* 普通对象方法至少需要一个self参数，代表类对象实例

* 类方法有类变量cls传入，从而可以用cls做一些相关的处理。并且有子类继承时，调用该类方法时，传入的类变量cls是子类，而非父类。 对于类方法，可以通过类来调用，就像C.f()，有点类似C＋＋中的静态方法, 也可以通过类的一个实例来调用，就像C().f()，这里C()，写成这样之后它就是类的一个实例了。

* 类方法有类变量cls传入，从而可以用cls做一些相关的处理。并且有子类继承时，调用该类方法时，传入的类变量cls是子类，而非父类。 对于类方法，可以通过类来调用，就像C.f()，有点类似C＋＋中的静态方法, 也可以通过类的一个实例来调用，就像C().f()，这里C()，写成这样之后它就是类的一个实例了。

##### python单例模式

```python
# -*- coding:utf-8 -*-


class SingleInstance(object):
    """Single Instance

    """

    __instance = None

    def __init__(self):
        pass

    def __new__(cls, *args, **kwargs):
        if cls.__instance is None:
            cls.__instance = object.__new__(cls, *args, **kwargs)
        return cls.__instance

    @staticmethod
    def s_func():
        print("static method")

    @classmethod
    def func(cls):
        cls.s_func()
        print("hello", cls)

if __name__ == '__main__':
    a = SingleInstance()
    b = SingleInstance()
    print(a)
    print(b)

    a.func()
    b.func()

```

##### classmethod 使用场景之一

[参考Stackoverflow讲解](https://stackoverflow.com/questions/12179271/meaning-of-classmethod-and-staticmethod-for-beginner)

重构类的时候不必要修改构造函数，只需要额外添加你要处理的函数，然后使用装饰符 @classmethod 就可以了

```python
# -*- coding:utf-8 -*-


class DateTest(object):
    """时间类

    """
    day = 0
    month = 0
    year = 0

    def __init__(self, year=0, month=0, day=0):
        self.day = day
        self.month = month
        self.year = year

    def out_date(self):
        print("year :", self.year)
        print("month :", self.month)
        print("day :", self.day)

    @classmethod
    def get_date(cls, string_date):
        year, month, day = map(int, string_date.split('-'))
        # 返回新的类实例
        return cls(year, month, day)

if __name__ == '__main__':
    d1 = DateTest(2018, 9, 1)
    d1.out_date()

    d2 = DateTest.get_date("2018-9-1")
    d2.out_date()

```

@staticmethod 其实它跟@classmethod非常相似，只是它没有任何必需的参数。

假设我们要去检验一个日期的字符串是否有效。这个任务与Date类相关，但是又不需要Date实例对象，在这样的情况下@staticmethod就可以派上用场了。如下：

```python
@staticmethod
def is_date_valid(date_as_string):
    day, month, year = map(int, date_as_string.split('-'))
    return day <= 31 and month <= 12 and year <= 3999

# usage:
is_date = Date.is_date_valid('11-09-2012')
```

### 类的__metaclass_

```python
# -*- coding:utf-8 -*-


class MyClass:
    """Summary of class here.

    Longer class information....
    Longer class information....

    Attributes:
        likes_spam: A boolean indicating if we like SPAM or not.
        eggs: An integer count of the eggs we have laid.
    """
    i = 12345

    def f(self):
        """test func

        :return:
        """
        return 'hello world'

if __name__ == '__main__':
    print(MyClass.__doc__)      # 类的说明文档
    print(MyClass.__module__)   # 类来自的模块，看在那里使用

    print(MyClass.__dict__)
    m = MyClass()
    m.i = 54321
    print(m.__dict__)           # 将对象内的属性和值用字典的方式显示出来

    print(m.__class__)          # 对象来源于哪个类
    print(m.f)


```

#### __new__ 与 __init__的区别

_init__是在类实例创建之后调用，而 __new__方法正是**创建这个类实例的方法**。

1. __init__ 通常用于初始化一个新实例，控制这个初始化的过程，比如添加一些属性，做一些额外的操作，发生在类实例被创建完以后。它是实例级别的方法。

2. __new__ 通常用于控制生成一个新实例的过程。它是类级别的方法。

__new__方法主要是当你继承一些不可变的class时(比如int, str, tuple)， 提供给你一个自定义这些类的实例化过程的途径。还有就是实现自定义的metaclass。

如下，int是不可变对象，通过继承int并重载__new__方法，实现一个永远都是正数的整数类型

```python
class PositiveInteger(int):
    def __init__(self, value):
        super(PositiveInteger, self).__init__(self, abs(value))


class PositiveIntegerAbs(int):
    def __new__(cls, value):
        return super(PositiveIntegerAbs, cls).__new__(cls, abs(value))


if __name__ == '__main__':
    i = PositiveInteger(-3)
    print(i)

    i = PositiveIntegerAbs(-3)
    print(i)
```

* 单例

```python
class Singleton(object):
    def __new__(cls):
        # 关键在于这，每一次实例化的时候，我们都只会返回这同一个instance对象
        if not hasattr(cls, 'instance'):
            cls.instance = super(Singleton, cls).__new__(cls)
        return cls.instance


if __name__ == '__main__':
    obj1 = Singleton()
    obj2 = Singleton()

    obj1.attr1 = 'value1'
    print(obj1.attr1, obj2.attr1)
    print(obj1 is obj2)     # True

```

## 类继承

[参考](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386820044406b227b3e751cc4d5190420d17a2dc6353000)

```python
# -*- coding:utf-8 -*-


class Animal(object):

    def run(self):
        print('Animal is running...')


class Dog(Animal):
    pass


class Cat(Animal):
    pass


if __name__ == '__main__':
    d = Dog()
    c = Cat()

    d.run()
    c.run()
```

继承有什么好处？最大的好处是子类获得了父类的全部功能。由于Animial实现了run()方法，因此，Dog和Cat作为它的子类，什么事也没干，就自动拥有了run()方法：

继承的第二个好处需要我们对代码做一点改进。可以重写父类的方法

```python
class Dog(Animal):

    def run(self):
        print('Dog is running...')


class Cat(Animal):

    def run(self):
        print('Cat is running...')
```

当子类和父类都存在相同的run()方法时，我们说，子类的run()覆盖了父类的run()，在代码运行的时候，总是会调用子类的run()。这样，我们就获得了继承的另一个好处：多态。

```python
# -*- coding:utf-8 -*-


class Animal(object):

    def run(self):
        print('Animal is running...')


class Dog(Animal):

    def run(self):
        print('Dog is running...')


class Cat(Animal):

    def run(self):
        print('Cat is running...')


def run_twice(animal):
    """
    新增一个Animal的子类，不必对run_twice()做任何修改，
    实际上，任何依赖Animal作为参数的函数或者方法都可以不加修改地正常运行，原因就在于多态。
    :param animal: 
    :return: 
    """
    animal.run()
    animal.run()

if __name__ == '__main__':

    a = Animal()
    d = Dog()

    run_twice(a)
    run_twice(d)
```

对于一个变量，我们只需要知道它是Animal类型，无需确切地知道它的子类型，就可以放心地调用run()方法，而具体调用的run()方法是作用在Animal、Dog、Cat还是Tortoise对象上，由运行时该对象的确切类型决定，这就是多态真正的威力：调用方只管调用，不管细节，而当我们新增一种Animal的子类时，只要确保run()方法编写正确，不用管原来的代码是如何调用的。这就是著名的“开闭”原则：

* 对扩展开放：允许新增Animal子类；

* 对修改封闭：不需要修改依赖Animal类型的run_twice()等函数。


## #