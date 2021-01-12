---
title: "MySQL优化"
layout: page
date: 2020-03-08 18:00
---

[TOC]

# MySQL优化

回顾mysql架构

![](../../content/db_cache/imgs/mysql_frame.png)

## 查询优化

* 慢查询
* explain

### explain

* 查看索引：`show index from mytable;`

* `alter table mytable drop index key_name;`

```java
mysql> explain select a from test where a=1;
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | test  | NULL       | ref  | a             | a    | 5       | const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain select a from test where b=1;
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type  | possible_keys | key  | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | test  | NULL       | index | NULL          | a    | 15      | NULL |    1 |   100.00 | Using where; Using index |
+----+-------------+-------+------------+-------+---------------+------+---------+------+------+----------+--------------------------+
1 row in set, 1 warning (0.00 sec)

mysql>
```

* id：选择标识符
* select_type：表示查询的类型
* table：输出结果集的表
* partitions：匹配的分区
* type：表示表的连接类型
* possible_keys：表示查询时，可能使用的索引
* key：表示实际使用的索引
* key_len：索引字段的长度
* ref：列与索引的比较
* rows：扫描出的行数(估算的行数)
* filtered：按表条件过滤的行百分比
* Extra：执行情况的描述和说明

## 索引优化

## 存储优化

## 数据库结构优化

## 硬件优化和缓存

## 索引失效？

### `or`导致的全表扫描

```sql
CREATE TABLE `user` (
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`),
  KEY `index_name` (`name`),
  KEY `index_age` (`age`),
  KEY `index_address` (`address`)
) ENGINE=InnoDB AUTO_INCREMENT=19 DEFAULT CHARSET=utf8mb4;

CREATE TABLE `job` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `userId` int(11) DEFAULT NULL,
  `job` varchar(255) DEFAULT NULL,
  `name` varchar(25) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `name_index` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=42 DEFAULT CHARSET=utf8mb4;
```

如下会全表扫描

```sql
mysql> explain SELECT name,age,address FROM user where name = 'aa' or age=1;
+----+-------------+-------+------------+------+----------------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys        | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+----------------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | user  | NULL       | ALL  | index_name,index_age | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+----------------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

### like模糊查询会导致全表扫描

### 查询的列上有运算或者函数的导致的全表扫描

```sql
mysql> explain SELECT name,age,address FROM user where substr(name,-2)='aa';
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | user  | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> explain SELECT name,age,address FROM user where name='aa';
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | user  | NULL       | ref  | index_name    | index_name | 1023    | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.01 sec)

mysql>
```

### 字符串列，要在条件中将数据使用引号，否则不使用索引

```sql
mysql> explain SELECT name,age,address FROM user where name = 10;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | user  | NULL       | ALL  | index_name    | NULL | NULL    | NULL |    1 |   100.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 3 warnings (0.01 sec)

mysql> explain SELECT name,age,address FROM user where name = '10';
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | user  | NULL       | ref  | index_name    | index_name | 1023    | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

mysql>
```

### 左连接查询或者右连接查询查询关联的字段编码格式不一样

设置varchar不同编码

```sql
CREATE TABLE `job2` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `userId` int(11) DEFAULT NULL,
  `job` varchar(255) DEFAULT NULL,
  `name` varchar(255) CHARACTER SET gbk DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=22 DEFAULT CHARSET=utf8mb4;
```

* explain对比

```sql
mysql> EXPLAIN select a.name,b.name,b.job from user a left JOIN job b ON a.name =b.name;
+----+-------------+-------+------------+-------+---------------+------------+---------+-------------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key        | key_len | ref         | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+------------+---------+-------------+------+----------+-------------+
|  1 | SIMPLE      | a     | NULL       | index | NULL          | index_name | 1023    | NULL        |    1 |   100.00 | Using index |
|  1 | SIMPLE      | b     | NULL       | ref   | name_index    | name_index | 103     | test.a.name |    1 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+------------+---------+-------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)

mysql> EXPLAIN select a.name,b.name,b.job from user a left JOIN job2 b ON a.name =b.name;
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key        | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | a     | NULL       | index | NULL          | index_name | 1023    | NULL |    1 |   100.00 | Using index                                        |
|  1 | SIMPLE      | b     | NULL       | ALL   | NULL          | NULL       | NULL    | NULL |    1 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.01 sec)

mysql>
```

注：如果是用覆盖索引的话，那么b表就会走索引了

### 如果mysql估计使用全表扫描要比使用索引快,则不使用索引

```sql
CREATE TABLE `user3` (
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`id`),
  KEY `index_name` (`name`),
  KEY `index_age` (`age`),
  KEY `index_address` (`address`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4;
INSERT INTO `test`.`user3`(`name`, `age`, `address`, `id`) VALUES ('光头强', 12, '狗熊岭', 1);
INSERT INTO `test`.`user3`(`name`, `age`, `address`, `id`) VALUES ('熊大', 9, '狗熊岭2', 2);
```

```sql
CREATE TABLE `job3` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `userId` int(11) DEFAULT NULL,
  `job` varchar(255) DEFAULT NULL,
  `name` varchar(255) CHARACTER SET utf8mb4 DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=22 DEFAULT CHARSET=utf8mb4;
INSERT INTO `test`.`job3`(`id`, `userId`, `job`, `name`) VALUES (1, 1, 'java', '光头强');
INSERT INTO `test`.`job3`(`id`, `userId`, `job`, `name`) VALUES (2, 2, 'php', '熊大');
```

* 由于要查询`b.name`，mysql需要回表，mysql认为走全表扫描会快一些，所以即使b表的name有索引，也不会走这个索引

```sql
mysql> EXPLAIN select a.name,b.name,b.job from user3 a left JOIN job3 b ON a.name =b.name;
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+----------------------------------------------------+
| id | select_type | table | partitions | type  | possible_keys | key        | key_len | ref  | rows | filtered | Extra                                              |
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+----------------------------------------------------+
|  1 | SIMPLE      | a     | NULL       | index | NULL          | index_name | 1023    | NULL |    2 |   100.00 | Using index                                        |
|  1 | SIMPLE      | b     | NULL       | ALL   | index_name    | NULL       | NULL    | NULL |    2 |   100.00 | Using where; Using join buffer (Block Nested Loop) |
+----+-------------+-------+------------+-------+---------------+------------+---------+------+------+----------+----------------------------------------------------+
2 rows in set, 1 warning (0.00 sec)

mysql>
```

### 如果查询中没有用到联合索引的第一个字段，则不会走索引

### IS NUll ，IS NOT NUll ，！= 是否走索引

MySQL中决定使不使用某个索引执行查询的依据就是成本够不够小，如果null值很多，还是会用到索引的。

一个大概3万数据的表，如果只有10多个记录是null值，is null走索引，not null和!=没走索引，如果大部分都是null值，只有部分几条数据有值，is null，not null和!=都走索引。

## MySql 基础语句练习（*）

### 建立表

```sql
-- 建学生信息表student
CREATE TABLE `student` (
  `sno` varchar(20) NOT NULL,
  `sname` varchar(20) NOT NULL,
  `ssex` varchar(20) NOT NULL,
  `sbirthday` datetime DEFAULT NULL,
  `class` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`sno`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 创建课程表course
CREATE TABLE `course` (
  `Cno` char(5) NOT NULL COMMENT '课程号（主码）',
  `Cname` varchar(10) NOT NULL COMMENT '课程名称',
  `Tno` char(3) NOT NULL COMMENT '教工编号（外码）',
  PRIMARY KEY (`Cno`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 创建分数表score
CREATE TABLE `score` (
  `Sno` char(3) DEFAULT NULL,
  `Cno` char(5) DEFAULT NULL,
  `Degree` decimal(4,1) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 创建教师表teacher
CREATE TABLE `teacher` (
  `tno` varchar(20) NOT NULL,
  `tname` varchar(20) NOT NULL,
  `tsex` varchar(20) NOT NULL,
  `tbirthday` datetime DEFAULT NULL,
  `prof` varchar(20) DEFAULT NULL,
  `depart` varchar(20) NOT NULL,
  PRIMARY KEY (`tno`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 插入数据

```sql
-- 插入student数据
insert into student values('108','曾华','男','1977-09-01','95033');
insert into student values('105','匡明','男','1975-10-02','95031');
insert into student values('107','王丽','女','1976-01-23','95033');
insert into student values('101','李军','男','1976-02-20','95033');
insert into student values('109','王芳','女','1975-02-10','95031');
insert into student values('103','陆君','男','1974-06-03','95031');

-- 插入course数据
insert into course values('3-105','计算机导论','825');
insert into course values('3-245','操作系统','804');
insert into course values('6-166','数字电路','856');
insert into course values('9-888','高等数学','831');

-- 插入score数据
insert into score values('103','3-245','86');
insert into score values('105','3-245','75');
insert into score values('109','3-245','68');
insert into score values('103','3-105','92');
insert into score values('105','3-105','88');
insert into score values('109','3-105','76');
insert into score values('101','3-105','64');
insert into score values('107','3-105','91');
insert into score values('108','3-105','78');
insert into score values('101','6-166','85');
insert into score values('107','6-166','79');
insert into score values('108','6-166','81');

-- 插入teacher数据
insert into teacher values('804','李诚','男','1958-12-02','副教授','计算机系');
insert into teacher values('856','张旭','男','1969-03-12','讲师','电子工程系');
insert into teacher values('825','王萍','女','1972-05-05','助教','计算机系');
insert into teacher values('831','刘冰','女','1977-08-14','助教','电子工程系');
```

### 常见sql语句练习

#### 查询Score表中的最高分的学生学号和课程号。（子查询或者排序）

* 使用子查询

```sql
mysql> select Sno,Cno,Degree from score where Degree >= (select max(Degree) from score);
+------+-------+--------+
| Sno  | Cno   | Degree |
+------+-------+--------+
| 103  | 3-105 |   92.0 |
+------+-------+--------+
1 row in set (0.00 sec)

mysql>
```

* score降序取limit 1

```sql
mysql> SELECT sno 学生学号,cno 课程号,degree 分数
    -> FROM score
    -> ORDER BY degree desc
    -> LIMIT 1;
+--------------+-----------+--------+
| 学生学号     | 课程号    | 分数   |
+--------------+-----------+--------+
| 103          | 3-105     |   92.0 |
+--------------+-----------+--------+
1 row in set (0.00 sec)

mysql>
```

#### 查询每门课的平均成绩

需要`group by cno`，可以直接使用avg函数

```sql
mysql> SELECT cno 课程名称,round(AVG(IFNULL(degree,0)),2) 平均成绩
    -> FROM score
    -> GROUP BY cno;
+--------------+--------------+
| 课程名称     | 平均成绩     |
+--------------+--------------+
| 3-105        |        81.50 |
| 3-245        |        76.33 |
| 6-166        |        85.00 |
+--------------+--------------+
3 rows in set (0.02 sec)

mysql>
```

或者使用除法即可：`sum(degree)/count(degree)`

```sql
mysql> SELECT cno, sum(degree) / count(degree) from score group by cno;
+-------+-----------------------------+
| cno   | sum(degree) / count(degree) |
+-------+-----------------------------+
| 3-105 |                    81.50000 |
| 3-245 |                    76.33333 |
| 6-166 |                    85.00000 |
+-------+-----------------------------+
3 rows in set (0.00 sec)

mysql>
```

#### 查询Score表中至少有2名学生选修的并以3开头的课程的平均分数

* group by & having的使用

```sql
mysql> select cno,avg(degree) from score group by cno having cno like '3-%' and count(cno)>2;
+-------+-------------+
| cno   | avg(degree) |
+-------+-------------+
| 3-105 |    81.50000 |
| 3-245 |    76.33333 |
+-------+-------------+
2 rows in set (0.00 sec)

mysql>
```

#### 查询选修“3-105”课程的且成绩高于“109”号同学成绩的所有同学的记录

子查询

```sql
mysql> select * from student where sno in (select sno from score where degree > (select degree from score where sno='109' and cno = '3-105') and cno = '3-105');
+-----+--------+------+---------------------+-------+
| sno | sname  | ssex | sbirthday           | class |
+-----+--------+------+---------------------+-------+
| 103 | 陆君   | 男   | 1974-06-03 00:00:00 | 95031 |
| 105 | 匡明   | 男   | 1975-10-02 00:00:00 | 95031 |
| 107 | 王丽   | 女   | 1976-01-23 00:00:00 | 95033 |
| 108 | 曾华   | 男   | 1977-09-01 00:00:00 | 95033 |
+-----+--------+------+---------------------+-------+
4 rows in set (0.01 sec)

mysql>
```
