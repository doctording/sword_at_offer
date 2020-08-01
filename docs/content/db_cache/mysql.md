---
title: "数据库/表/查询"
layout: page
date: 2020-03-08 18:00
---

[TOC]

# MySQL

<a href='https://dev.mysql.com/doc/' target='_blank'>官方文档</a>

<a href='https://dev.mysql.com/doc/refman/5.6/en/' target='_blank'>mysql 5.6</a>

<a href='https://blog.csdn.net/qq_26437925/category_5779305.html' target='_blank'>我的博客专题</a>

## mysql的数据库引擎有哪些？

1. InnoDB
2. Myisam
3. Memory

* InnoDB,Myisam的默认索引是`B+Tree`，Memory的默认索引是`hash`
* InnoDB支持**事务**，支持**外键**，支持**行锁**，写入数据时操作快，MySQL5.6版本以上才支持**全文索引**
* Myisam不支持事务。不支持外键，支持表锁，支持全文索引，读取数据快
* Memory所有的数据都保留在内存中,不需要进行磁盘的IO所以读取的速度很快, 但是一旦关机,表的结构会保留但是数据就会丢失,表支持Hash索引，因此查找速度很快

## 数据库范式

### 第一范式

每个属性都不可再分

![](../../content/db_cache/imgs/1nf.png)

### 第二范式

![](../../content/db_cache/imgs/bug.png)

1. 每一名学生的学号、姓名、系名、系主任这些数据重复多次。每个系与对应的系主任的数据也重复多次——数据冗余过大

2. 假如学校新建了一个系，但是暂时还没有招收任何学生（比如3月份就新建了，但要等到8月份才招生），那么是无法将系名与系主任的数据单独地添加到数据表中去的 （注１）——插入异常

    * 注１：根据三种关系完整性约束中实体完整性的要求，关系中的码（注２）所包含的任意一个属性都不能为空，所有属性的组合也不能重复。为了满足此要求，图中的表，只能将学号与课名的组合作为码，否则就无法唯一地区分每一条记录

    * 码：关系中的某个属性或者某几个属性的组合，用于区分每个元组（可以把“元组”理解为一张表中的每条记录，也就是每一行）。

3. 假如将某个系中所有学生相关的记录都删除，那么所有系与系主任的数据也就随之消失了（一个系所有学生都没有了，并不表示这个系就没有了）。——删除异常

4. 假如李小明转系到法律系，那么为了保证数据库中数据的一致性，需要修改三条记录中系与系主任的数据。——修改异常

“若在一张表中，在属性（或属性组）X的值确定的情况下，必定能确定属性Y的值，那么就可以说Y函数依赖于X，写作`X → Y`”

**第二范式：在1NF的基础上，非码属性必须完全依赖于候选码（在1NF基础上消除非主属性对主码的部分函数依赖）**

---

* 选课（学号，课名，分数）
* 学生（学号，姓名，系名，系主任）

对于选课表，其码是（学号，课名），主属性是学号和课名，非主属性是分数，学号确定，并不能唯一确定分数，课名确定，也不能唯一确定分数，所以不存在非主属性分数对于码 （学号，课名）的部分函数依赖，所以此表符合2NF的要求。

对于学生表，其码是学号，主属性是学号，非主属性是姓名、系名和系主任，因为码只有一个属性，所以不可能存在非主属性对于码 的部分函数依赖，所以此表符合2NF的要求。

### 第三范式

**在2NF基础上，任何非主属性不依赖于其它非主属性（在2NF基础上消除传递依赖）**

* 学生（学号，姓名，系名，系主任）

对于学生表，主码为学号，主属性为学号，非主属性为姓名、系名和系主任。因为 `学号 → 系名`，同时 `系名 → 系主任`，所以存在非主属性`系主任`对于码`学号`的**传递函数**依赖，所以学生表的设计，不符合3NF的要求。

---

* 选课（学号，课名，分数）
* 学生（学号，姓名，系名）
* 系（系名，系主任）

![](../../content/db_cache/imgs/3nf.png)

### BCNF范式

巴斯-科德范式（BCNF）是第三范式（3NF）的一个子集，即满足巴斯-科德范式（BCNF）必须满足第三范式（3NF）。通常情况下，巴斯-科德范式被认为没有新的设计规范加入，只是对第二范式与第三范式中设计规范要求更强，因而被认为是修正第三范式，也就是说，它事实上是对第三范式的修正，使数据库冗余度更小。这也是BCNF不被称为第四范式的原因

## 数据表join操作

* INNER JOIN: 内连接是最常见的一种连接，只连接匹配的行

* LEFT JOIN: 返回左表的全部行和右表满足ON条件的行，如果左表的行在右表中没有匹配，那么这一行右表中对应数据用NULL代替

* RIGHT JOIN: 返回右表的全部行和左表满足ON条件的行，如果右表的行在左表中没有匹配，那么这一行左表中对应数据用NULL代替

* FULL OUTER JOIN: 会从左表 和右表 那里返回所有的行。如果其中一个表的数据行在另一个表中没有匹配的行，那么对面的数据用NULL代替

* CROSS JOIN: 把表A和表B的数据进行一个N*M的组合，即笛卡尔积

### 多表连接的三种方式详解 hash join、merge join、nested loop

#### nested loop（嵌套循环）

**驱动表**(也叫外表)和被驱动表(也叫**非驱动表**，还可以叫匹配表，亦可叫内表)，简单来说，驱动表就是主表，left join 中的左表就是驱动表，right join 中的右表是驱动表。一个是驱动表，那另一个就只能是非驱动表了

在 join 的过程中，其实就是从驱动表里面依次(注意理解这里面的依次)取出每一个值，然后去非驱动表里面进行匹配，那具体是怎么匹配的呢？这就是我们接下来讲的这三种连接方式：

1. (Simple Nested-Loop Join )暴力匹配的方式；如果 table A 有10行，table B 有10行，总共需要执行10 x 10 = 100次查询

2. (Index Nested-Loop Join)这个 Index 是要求非驱动表上要有索引，有了索引以后可以减少匹配次数，匹配次数减少了就可以提高查询的效率了,eg:左边就是普通列的存储方式，右边是树结构索引, 能减少查询次数

3. (Block Nested-Loop Join)

理想情况下，用索引匹配是最高效的一种方式，但是在现实工作中，并不是所有的列都是索引列，这个时候就需要用到 Block Nested-Loop Join 方法了，这种方法与第一种方法比较类似，唯一的区别就是:会把驱动表中 left join 涉及到的所有列(**不止是用来on的列，还有select部分的列**)先取出来放到一个**缓存区域**，然后再去和非驱动表进行匹配，这种方法和第一种方法相比所需要的匹配次数是一样的，差别就在于驱动表的列数不同，也就是数据量的多少不同。所以虽然匹配次数没有减少，但是总体的查询性能还是有提升的。

---

适用于小表与小表的连接

#### Hash Join

hash join仅仅在join的字段上`没有索引`时才起作用，在此之前，我们不建议在没有索引的字段上做join操作，因为通常中这种操作会执行得很慢，但是有了hash join，它`能够创建一个内存中的hash表`，代替之前的nested loop，使得没有索引的`等值join`性能提升很多。

1. 配置hash join功能是否开启：

    * optimizer_switch 中的 hash_join=on/off，默认为on
    * sql语句中指定HASH_JOIN或者NO_HASH_JOIN
限制：

2. hash join只能在没有索引的字段上有效
    * hash join只在等值join条件中有效
    * hash join不能用于left join和right join

---

适用于小表与大表的连接

#### merge join

merge join第一个步骤是确保两个关联表都是按照关联的字段进行排序。如果关联字段有可用的索引，并且排序一致，则可以直接进行merge join操作；

两个表都按照关联字段排序好之后，merge join操作从每个表取一条记录开始匹配，如果符合关联条件，则放入结果集中；否则，将关联字段值较小的记录抛弃，从这条记录对应的表中取下一条记录继续进行匹配，直到整个循环结束。

```java
 function sortMerge(relation left, relation right, attribute a)
     var relation output
     var list left_sorted := sort(left, a) // Relation left sorted on attribute a
     var list right_sorted := sort(right, a)
     var attribute left_key, right_key
     var set left_subset, right_subset // These sets discarded except where join predicate is satisfied
     advance(left_subset, left_sorted, left_key, a)
     advance(right_subset, right_sorted, right_key, a)
     while not empty(left_subset) and not empty(right_subset)
         if left_key = right_key // Join predicate satisfied
             add cartesian product of left_subset and right_subset to output
             advance(left_subset, left_sorted, left_key, a)
             advance(right_subset, right_sorted, right_key, a)
         else if left_key < right_key
            advance(left_subset, left_sorted, left_key, a)
         else // left_key > right_key
            advance(right_subset, right_sorted, right_key, a)
     return output

 // Remove tuples from sorted to subset until the sorted[1].a value changes
 function advance(subset out, sorted inout, key out, a in)
     key := sorted[1].a
     subset := emptySet
     while not empty(sorted) and sorted[1].a = key
         insert sorted[1] into subset
         remove sorted[1]
```

merge join操作本身是非常快的，但是merge join前进行的排序可能会相当耗时

它首先根据R和S的join key分别对两张表进行排序，然后同时遍历排序后的R和S

其I/O复杂度可以表示为O[p(R) + p(S) + p(R) · logp(R) + p(S) · logp(S)]

附：归并排序是稳定排序，最好，最坏，平均时间复杂度均为O(nlogn)。

## InnoDb

<a href='https://dev.mysql.com/doc/refman/5.6/en/innodb-storage-engine.html' target="_blank">Mysql innodb refman</a>

## 索引？

## MySql有哪些锁？

## 慢查询优化
