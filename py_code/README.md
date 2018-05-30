# 目的：

* python基础语法巩固

* 剑指offer基础算法巩固

---

# python基础

## 标准数据类型

### Numbers 数字

	int, long, float, double

### String 字符串

读取顺序

	* 从左到右索引默认0开始的，最大范围是字符串长度少1

	* 从右到左索引默认-1开始的，最大范围是字符串开头

### List 列表

	用```[]```表示

* 基础操作

```
len(list) 
max(list)
min(list)
list.reverse()
list.sort(cmp=None, key=None, reverse=False)
```

* 增

```
list.append(obj)
list.insert(index, obj)
```

* 删
```
list.pop([index=-1])
list.remove(obj)
```

* 查找

```
list.count(obj) // 统计某个元素在列表中出现的次数
list[2] // 读取列表中第三个元素
list[-2] // 读取列表中倒数第二个元素
list[1:] // 从第二个元素开始截取列表
list.index(obj) // 从列表中找出某个值第一个匹配项的索引位置
```

### Tuple 元组

### Dictionary 字典

	用```{}```表示, ```key:value```, 通过key来读取,而非偏移

# 条件语句

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

# for, while

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

# #