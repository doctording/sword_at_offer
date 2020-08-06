---
title: "MySQL"
layout: page
date: 2020-03-08 18:00
---

[TOC]

# MySQL

<a href='https://dev.mysql.com/doc/' target='_blank'>官方文档</a>

<a href='https://dev.mysql.com/doc/refman/5.6/en/' target='_blank'>mysql 5.6</a>

<a href='https://blog.csdn.net/qq_26437925/category_5779305.html' target='_blank'>我的博客专题</a>

## 数据库引擎有哪些？

* InnoDB
* Myisam
* Memory

说明：

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

### 页（Page）是 Innodb 存储引擎用于管理数据的最小磁盘单位

![](../db_cache/imgs/page.png)

1. File Header: 文件头部，页的一些通用信息（38字节）
2. page Header: 页面头部,数据页专有的一些信息（56字节）
3. infimum+supremum: 行记录最小值和最大值，两个虚拟的行记录（26字节）
4. user recorders: 实际存储的行记录内容（不确定）
5. free space: 页中尚未使用的空间（不确定）
6. Page Directory: 页中的某些记录的相对位置（不确定）
7. File Tailer: 校验页是否完整（8字节）

#### 记录在页中的存储

1.当一个记录需要插入页的时候，会从free space划分空间到user recorders
2.Free Space部分的空间全部被User Records部分替代掉之后，也就意味着这个页使用完了，如果还有新的记录插入的话，就需要去申请新的页了。

### innodb ibd文件

ibd文件是以`页`为单位进行管理的，页通常是以16k为单位，所以ibd文件通常是16k的整数倍

#### innodb 页类型

名称 | 十六进制 | 解释
:---:|:---:|:---:
FIL_PAGE_INDEX | 0x45BF | B+树叶节点
FIL_PAGE_UNDO_LOGO | 0x0002 | UNDO LOG页
FIL_PAGE_INODE | 0x0003 | 索引节点
FIL_PAGE_IBUF_FREE_LIST | 0x0004 | InsertBuffer空闲列表
FIL_PAGE_TYPE_ALLOCATED | 0x0000 | 该页的最新分配
FIL_PAGE_IBUF_BITMAP | 0x0005 | InsertBuffer位图
FIL_PAGE_TYPE_SYS | 0x0006 | 系统页
FIL_PAGE_TYPE_TRX_SYS | 0x0007 | 事务系统数据
FIL_PAGE_TYPE_FSP_HDR | 0x0008 | FILE SPACE HEADER
FIL_PAGE_TYPE_XDES | 0x0009 | 扩展描述页
FIL_PAGE_TYPE_BLOB | 0x000A | BLOB页

#### 实践分析`ibd`文件

```java
mubi@mubideMacBook-Pro bin $ mysql -uroot -p123456
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 109
Server version: 5.6.40 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use test;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from tb_user;
+------------+--------+----------+-------+-------------+----------+
| id         | userID | password | name  | phone       | address  |
+------------+--------+----------+-------+-------------+----------+
|          1 | 00001  | 123456   | zhang | 15133339999 | Shanghai |
|          2 | 00002  | 123456   | wang  | 15133339999 | Beijing  |
|          4 | 0003   | NULL     | abc   | NULL        | NULL     |
|          6 | 0003   | NULL     | abc   | NULL        | NULL     |
|          7 | 0004   | 123456   | tom   | 110         | beijing  |
|         10 | 0004   | 123456   | tom   | 110         | beijing  |
| 2147483647 | 0004   | 123456   | tom   | 110         | beijing  |
+------------+--------+----------+-------+-------------+----------+
7 rows in set (0.00 sec)

mysql>
```

#### 查看数据表的行格式

```java
mysql> show table status like 'tb_user'\G;
*************************** 1. row ***************************
           Name: tb_user
         Engine: InnoDB
        Version: 10
     Row_format: Compact
           Rows: 6
 Avg_row_length: 2730
    Data_length: 16384
Max_data_length: 0
   Index_length: 16384
      Data_free: 0
 Auto_increment: 2147483647
    Create_time: 2020-03-21 16:10:06
    Update_time: NULL
     Check_time: NULL
      Collation: utf8_general_ci
       Checksum: NULL
 Create_options:
        Comment:
1 row in set (0.00 sec)

ERROR:
No query specified

mysql>
```

#### 查看ibd文件

![](../db_cache/imgs/innodb-1.png)

使用`py_innodb_page_info`工具（`https://github.com/happieme/py_innodb_page_info`）

![](../db_cache/imgs/innodb-2.png)

注意到文件大小`114688`字节（114688 = 16 * 1024 * 7）即有`7`个页（要分析哪个页直接定位到二进制文件到开始，然后分析即可）

#### 分析第4个页：`B-tree Node`类型

`page offset 00000003, page type <B-tree Node>, page level <0000>`

```java
>>> hex(3 * 16 * 1024)
'0xc000'
>>> hex(4 * 16 * 1024)
'0x10000'
>>>
```

##### 先分析File Header(38字节-描述页信息)

![](../db_cache/imgs/innodb-3.png)

* 2D A1 2D 57 -> 数据页的checksum值
* 00 00 00 03 -> 页号（偏移量）,当前是第3页
* FF FF FF FF -> 目前只有一个数据页，无上一页
* FF FF FF FF -> 目前只有一个数据页，无下一页
* 00 00 00 04 6F 65 24 CF -> 该页最后被修改的LSN
* 45 BF -> 页的类型，0x45BF代表数据页，刚好这页是数据页
* 00 00 00 00 00 00 00 00 -> 独立表空间，该值为0
* 00 00 00 06 -> 表空间的SPACE ID

##### 再分析Page Header（56字节-记录页的状态信息）

参见：<a href="https://dev.mysql.com/doc/internals/en/innodb-page-header.html" target='_blank'>innodb-page-header</a>

![](../db_cache/imgs/innodb-4.png)

标识 | 字节数 | 解释 | 本次值:说明
:---:|:---:|:---:|:---:|:---:
PAGE_N_DIR_SLOTS | 2 | number of directory slots in the Page Directory part; initial value = 2| 00 02,2个槽位
PAGE_HEAP_TOP | 2 | record pointer to first record in heap | 01 ED，堆第一个开始位置的偏移量，也即空闲偏移量
PAGE_N_HEAP | 2 | number of heap records; initial value = 2 | 80 0A
PAGE_FREE | 2 | record pointer to first free record | 01 1C
PAGE_GARBAGE | 2 | "number of bytes in deleted records" | 00 20，删除的记录字节
PAGE_LAST_INSERT | 2 | record pointer to the last inserted record | 01 C5，最后插入记录的位置偏移
PAGE_DIRECTION | 2 | either PAGE_LEFT, PAGE_RIGHT, or PAGE_NO_DIRECTION | 00 02，自增长的方式进行行记录的插入，方向向右
PAGE_N_DIRECTION | 2 | number of consecutive inserts in the same direction, for example, "last 5 were all to the left" | 00 02
PAGE_N_RECS | 2 | number of up[ser records | 00 07，共7条有效记录数
PAGE_MAX_TRX_ID | 8 | the highest ID of a transaction which might have changed a record on the page (only set for secondary indexes) | 00 00 00 00 00 00 00 00
PAGE_LEVEL | 2 | level within the index (0 for a leaf page) | 00 00
PAGE_INDEX_ID | 8 | identifier of the index the page belongs to | 00 00 00 00 00 00 00 16
PAGE_BTR | 10 | "file segment header for the leaf pages in a B-tree" (this is irrelevant here) | 00 00 00 06 00 00 00 02 00 F2
PAGE_LEVEL | 10 | "file segment header for the non-leaf pages in a B-tree" (this is irrelevant here) | 00 00 00 06 00 00 00 02 00 32

* `0xc000 + 01 ED` = `0xC1ED`地址后面的都是空闲的

![](../db_cache/imgs/innodb-7.png)

* `0xc000 + 01 C5` = `0xC1C5`最后一条记录

![](../db_cache/imgs/innodb-8.png)

##### 分析Infimum + Supremum Record (26字节-两个虚拟行记录)

infimum: n. 下确界;
supremum: n. 上确界;

Infimum和Suprenum Record用来限定记录的边界，Infimum是比页中任何主键值都要小的值，Suprenum 是指比任何可能大值还要大的值，这两个值在页创建时被建立，并且在任何情况下都不会被删除。Infimum和Suprenum与行记录组成单链表结构，查询记录时，从Infimum开始查找，如果找不到结果会直到查到最后的suprenum为止，然后通过Page Header中的FIL_PAGE_NEXT指针跳转到下一个page继续从Infimum开始逐个查找

![](../db_cache/imgs/innodb-5.png)

```java
#Infimum伪行记录
01 00 02 00 20/*recorder header*/
69 6E 66 69 6D 75 6D 00/*只有一个列的伪行记录，记录内容就是Infimum（多了一个0x00字节）
*/
#Supremum伪行记录
08 00 0B 00 00/*recorder header*/
73 75 70 72 65 6D 75 6D/*只有一个列的伪行记录，记录内容就是Supremum*/
```

infimum行记录的recorder header部分，最后2个字节位`00 20`表示下一个记录的位置的偏移量

##### User Record(表中的数据记录)

用户所有插入的记录都存放在这里，默认情况下记录跟记录之间没有间隙，但是如果重用了已删除记录的空间，就会导致空间碎片。每个记录都有指向下一个记录的指针，但是没有指向上一个记录的指针。记录按照主键顺序排序。即，用户可以从数据页最小记录开始遍历，直到最大的记录，这包括了所有正常的记录和所有被delete-marked记录，但是不会访问到被删除的记录(PAGE_FREE)

###### COMPACT行记录格式

![](../db_cache/imgs/innodb-9.jpg)

* 行格式的首部是一个非NULL变长字段长度列表，而且是按照列的顺序逆序放置的。当列的长度小于255字节，用1字节表示，若大于255个字节，用2个字节表示，变长字段的长度最大不可以超过2个字节（这也很好地解释了为什么MySQL中varchar的最大长度为65535，因为2个字节为16位，即`pow(2,16)-1=65536`）。第二个部分是NULL标志位，该位指示了该行数据中是否有NULL值，1个字节表示。该部分所占的字节应该为bytes。接下去的部分是为记录头信息（record header），固定占用5个字节（40位），每位的含义如下

* 预留位1	1（bit位）	没有使用
* 预留位2	1	没有使用
* delete_mask	1	标记该记录是否被删除
* min_rec_mask	1	标记该记录是否为B+树的非叶子节点中的最小记录
* n_owned	4	表示当前槽管理的记录数
* heap_no	13	表示当前记录在记录堆的位置信息
* record_type	3	表示当前记录的类型，0表示普通记录，1表示B+树非叶节点记录，2表示最小记录，3表示最大记录
* next_record	16	表示下一条记录的相对位置

![](../db_cache/imgs/innodb-10.png)

```java
0000c070: 73 75 70 72 65 6D 75 6D 08 0B 05 06 05 00 00 00    supremum........
0000c080: 10 00 3F 80 00 00 01 00 00 00 00 27 06 86 00 00    ..?........'....
0000c090: 01 39 01 10 30 30 30 30 31 31 32 33 34 35 36 7A    .9..00001123456z
0000c0a0: 68 61 6E 67 31 35 31 33 33 33 33 39 39 39 39 53    hang15133339999S
0000c0b0: 68 61 6E 67 68 61 69 07 0B 04 06 05 00 00 00 18    hanghai.........

08 0B 05 06 05: 8(address字段) 11(phone字段) 5(name字段) 6(password字段) 5(userID字段)，一个逆序的方式表示可变长字段列表
00 : Null值列表
00 00 10 00 3F：记录头信息 5个字节，40bit
    0000 0000 0000 0000 00001 0000 0000 0000 0011 1111
80 00 00 01：自增主键（有符号的int类型），1
00 00 00 00 27 06：隐藏列DB_TRX_ID
86 00 00 01 39 01 10：隐藏列DB_ROLL_PTR
30 30 30 30 31 ： 00001
31 32 33 34 35 36 ： 123456
7A 68 61 6E 67 : zhang
31 35 31 33 33 33 33 39 39 39 39 : 15133339999
53 68 61 6E 67 68 61 69 : beijing
```

##### File Tailer(最后8字节)

![](../db_cache/imgs/innodb-6.png)

`7E 75 29 30 6F 65 24 CF`

注意到File Header该页最后被修改的LSN：`00 00 00 04 6F 65 24 CF`，可以看到后4个字节和`File Tailer`的后4个字节相同

##### 附：二进制文件查看小工具

* 使用python可以方便的进行二进制相关的转换
    1. hex(16) # 10进制转16进制
    2. oct(8) # 10进制转8进制
    3. bin(8) # 10进制转2进制

```java
>>> hex(6 * 16 * 1024)
'0x18000'
>>> hex(3 * 16 * 1024)
'0xc000'
>>>
```

* vscode可以安装`hexdump for VSCode`插件

### 页分裂/页合并

<a href="https://www.percona.com/blog/2017/04/10/innodb-page-merging-and-page-splitting/" href='_blank'>innodb-page-merging-and-page-splitting</a>

InnoDB不是按行的来操作的，它可操作的最小粒度是页，页加载进内存后才会通过扫描页来获取行/记录。

#### innodb数据的存储(.frm & .ibd)

在 InnoDB 存储引擎中，所有的数据都被逻辑地存放在表空间中，表空间（tablespace）是存储引擎中最高的存储逻辑单位，在表空间的下面又包括段（segment）、区（extent）、页（page）, 页中存放实际的数据记录行

![](../db_cache/imgs/innodb-11.jpg)

MySQL 使用 InnoDB 存储表时，会将表的定义和数据索引等信息分开存储，其中前者存储在`.frm`文件中，后者存储在`.ibd`文件中(ibd文件既存储了数据也存储了索引)

![](../db_cache/imgs/innodb-12.jpg)

在创建表时，会在磁盘上的 datadir 文件夹中生成一个 `.frm` 的文件，这个文件中包含了表结构相关的信息

##### 页合并

删除记录时会设置record的flaged标记为删除，当一页中删除记录超过<font color='red'>MERGE_THRESHOLD（默认页体积的50%）</font>时，InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。例如：合并操作使得页#5保留它之前的数据，并且容纳来自页#6的数据。页#6变成一个空页，可以接纳新数据。

##### 页分裂

页可能填充至100%，在页填满了之后，下一页会继续接管新的记录。但如果下一页页没有足够空间去容纳新（或更新）的记录，那么

1. 创建新页
2. 判断当前页（页#10）可以从哪里进行分裂（记录行层面）
3. 移动记录行
4. 重新定义页之间的关系

例如：页#10没有足够空间去容纳新记录，页#11也同样满了, #10要分列为两列, 且页的前后指针关系要发生改变

```java
#9 #10 #11 #12 #13 ...

#8 #10 #14 #11 #12 #13 ...
```

## 索引

### MySql默认索引（InnoDB）

```sql
mysql> show variables like '%storage_engine%';
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    112
Current database: test

+----------------------------+--------+
| Variable_name              | Value  |
+----------------------------+--------+
| default_storage_engine     | InnoDB |
| default_tmp_storage_engine | InnoDB |
| storage_engine             | InnoDB |
+----------------------------+--------+
3 rows in set (0.06 sec)

mysql>
```

### 为什么要用`B+ Tree`而不是`B Tree`?

1行记录假如有1KB；如果`B Tree`（其非叶子节点是要存储数据的，显然存储有限）；B-Tree为了存储大量数据，不得不提高树的高度，这就会导致IO次数增多。所以有`B+ Tree`

### MySql中B+树索引可以分为聚集索引和非聚集索引

#### 聚集索引（clustered index）

索引中键值的逻辑顺序决定了表中相应行的物理顺序（索引中的数据物理存放地址和索引的顺序是一致的），可以这么理解：只要是索引是连续的，那么数据在存储介质上的存储位置也是连续的。

聚集索引就像我们根据拼音的顺序查字典一样，可以大大的提高效率。在经常搜索一定范围的值时，通过索引找到第一条数据，根据物理地址连续存储的特点，然后检索相邻的数据，直到到达条件截至项。

聚集索引：叶子节点包含了完整的数据记录

Cluster index is a type of index which sorts the data rows in the table on their key values. In the Database, there is only one clustered index per table.

A clustered index defines the order in which data is stored in the table which can be sorted in only one way. So, there can be an only a single clustered index for every table. In an RDBMS, usually, the primary key allows you to create a clustered index based on that specific column.

#### 非聚集索引

索引的逻辑顺序与磁盘上的物理存储顺序不同。非聚集索引的键值在逻辑上也是连续的，但是表中的数据在存储介质上的物理顺序是不一致的，即记录的逻辑顺序和实际存储的物理顺序没有任何联系。索引的记录节点有一个数据指针指向真正的数据存储位置。

A Non-clustered index stores the data at one location and indices at another location. The index contains pointers to the location of that data. A single table can have many non-clustered indexes as an index in the non-clustered index is stored in different places.

For example, a book can have more than one index, one at the beginning which displays the contents of a book unit wise while the second index shows the index of terms in alphabetical order.

A non-clustering index is defined in the non-ordering field of the table. This type of indexing method helps you to improve the performance of queries that use keys which are not assigned as a primary key. A non-clustered index allows you to add a unique key for a table.

#### 对比和注意点

<a href="https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html" target='_blank'>innodb-index-types</a>

* 如果一个主键被定义了，那么这个主键就是作为聚集索引
* 如果没有主键被定义，那么该表的第一个唯一非空索引会被作为聚集索引
* 如果没有主键也没有合适的唯一索引，那么innodb内部会生成一个隐藏的主键作为聚集索引，这个隐藏的主键是一个6个字节的列，该列的值会随着数据的插入进行自增

* 聚集索引的特点

1. 聚集索引表记录的排列顺序和索引的排列顺序保持一致，所以查询效率相当快。只要找到第一个索引记录的值，其余的连续性的记录也一定是连续存放的。
2. 聚集索引的缺点就是修改起来比较慢，因为它需要保持表中记录和索引的顺序一致，在插入新记录的时候就会对数据也重新做一次排序。
3. InnoDB表数据本身就是一个按B+Tree组织的一个索引结构文件，叶节点包含了完整的数据记录（.ibd文件）;MyIsam数据和索引文件是分开的（.MYD文件，.MYI文件）

* myisam

![](../db_cache/imgs/myisam.png)

* innodb

![](../db_cache/imgs/innodb.png)

#### 为什么InnoDB要有主键？并且推荐使用整型自增主键？

为什么InnoDB要有主键？（InnoDB设计如此）

* 如果一个主键被定义了，那么这个主键就是作为聚集索引
* 如果没有主键被定义，那么该表的第一个唯一非空索引会被作为聚集索引
* 如果没有主键也没有合适的唯一索引，那么innodb内部会生成一个隐藏的主键作为聚集索引，这个隐藏的主键是一个6个字节的列，该列的值会随着数据的插入进行自增

推荐使用整型？

* B+Tree搜索比对，显然整型比字符串比较快（原因1），整型占用空间小（原因2）

推荐使用自增？

补充：hash索引，直接定位到记录的磁盘地址（等值查找）**区间查找**用hash行不同，所以hash索引用的少

自增是页不断的创建新增，后面加，调整小；如果非自增，涉及到页分裂/创建，B+Tree调整大

<a href='https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html' href='_blank'>B+Tree 可视化操作</a>

### 索引类型

#### 联合索引

![](../db_cache/imgs/index_union.png)

仍然是B+Tree, 索引中包含多个字段，按照联合索引的先后顺序

### 联合索引为什么是最左前缀匹配？

数据结构底层决定（严格的按照第一个，第二个，第三个字段一个一个匹配），不符合最左匹配则需要全局扫描了

#### MySql索引概述

<a href>='https://dev.mysql.com/doc/refman/5.7/en/mysql-indexes.html' target='_blank'>https://dev.mysql.com/doc/refman/5.7/en/mysql-indexes.html</a>

Indexes are used to find rows with specific column values quickly. Without an index, MySQL must begin with the first row and then read through the entire table to find the relevant rows. The larger the table, the more this costs. If the table has an index for the columns in question, MySQL can quickly determine the position to seek to in the middle of the data file without having to look at all the data. This is much faster than reading every row sequentially.

Most MySQL indexes (PRIMARY KEY, UNIQUE, INDEX, and FULLTEXT) are stored in B-trees. Exceptions: Indexes on spatial data types use R-trees; MEMORY tables also support hash indexes; InnoDB uses inverted lists for FULLTEXT indexes.

MySQL uses indexes for these operations:

* To find the rows matching a WHERE clause quickly.

* To eliminate rows from consideration. If there is a choice between multiple indexes, MySQL normally uses the index that finds the smallest number of rows (**the most selective** index).

* If the table has a multiple-column index, any **leftmost prefix** of the index can be used by the optimizer to look up rows. For example, if you have a three-column index on (col1, col2, col3), you have indexed search capabilities on (col1), (col1, col2), and (col1, col2, col3). 

* To retrieve rows from other tables when performing joins. MySQL can use indexes on columns more efficiently if they are declared as the same type and size. In this context, VARCHAR and CHAR are considered the same if they are declared as the same size. For example, VARCHAR(10) and CHAR(10) are the same size, but VARCHAR(10) and CHAR(15) are not.

* To find the MIN() or MAX() value for a specific indexed column key_col. This is optimized by a preprocessor that checks whether you are using WHERE key_part_N = constant on all key parts that occur before key_col in the index. In this case, MySQL does a single key lookup for each MIN() or MAX() expression and replaces it with a constant. If all expressions are replaced with constants, the query returns at once.

* To sort or group a table if the sorting or grouping is done on a leftmost prefix of a usable index (for example, ORDER BY key_part1, key_part2). If all key parts are followed by DESC, the key is read in reverse order.

* In some cases, a query can be optimized to retrieve values without consulting the data rows. (An index that provides all the necessary results for a query is called a covering index.) If a query uses from a table only columns that are included in some index, the selected values can be retrieved from the index tree for greater speed

最后：Indexes are less important for queries on small tables, or big tables where report queries process most or all of the rows. When a query needs to access most of the rows, reading sequentially is faster than working through an index. Sequential reads minimize disk seeks, even if not all the rows are needed for the query.

## InnoDb锁


## 慢查询优化？
