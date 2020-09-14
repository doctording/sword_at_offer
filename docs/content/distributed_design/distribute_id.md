---
title: "分布式Id"
layout: page
date: 2020-06-07 00:00
---

[TOC]

# 分布式Id

## 数据库自增ID

The `AUTO_INCREMENT` attribute can be used to generate a unique identity for new rows:

```sql
CREATE TABLE animals (
 id MEDIUMINT NOT NULL AUTO_INCREMENT,
 name CHAR(30) NOT NULL,
 PRIMARY KEY (id)
);

INSERT INTO animals (name) VALUES
 ('dog'),('cat'),('penguin'),
 ('lax'),('whale'),('ostrich');

SELECT * FROM animals;
```

When you insert any other value into an AUTO_INCREMENT column, the column is set to that value
and the sequence is reset so that the next automatically generated value follows sequentially from the
largest column value. For example:

```sql
INSERT INTO animals (id,name) VALUES(100,'rabbit');
INSERT INTO animals (id,name) VALUES(NULL,'mouse');
SELECT * FROM animals;
+-----+-----------+
| id | name |
+-----+-----------+
| 1 | dog |
| 2 | cat |
| 3 | penguin |
| 4 | lax |
| 5 | whale |
| 6 | ostrich |
| 7 | groundhog |
| 8 | squirrel |
| 100 | rabbit |
| 101 | mouse |
+-----+-----------+
```

### AUTO_INCREMENT Handling in InnoDB

#### 相关的MySql系统变量

```sql
mysql> show variables like '%auto_increment%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| auto_increment_increment | 1     |
| auto_increment_offset    | 1     |
+--------------------------+-------+
2 rows in set (0.01 sec)

mysql>
```

* auto_increment_increment：自增量
* auto_increment_offset：自增开始值

* 自增锁的`innodb_autoinc_lock_mode`是1

```sql
mysql> show variables like '%lock%';
+-----------------------------------------+----------------------+
| Variable_name                           | Value                |
+-----------------------------------------+----------------------+
| block_encryption_mode                   | aes-128-ecb          |
| innodb_api_disable_rowlock              | OFF                  |
| innodb_autoinc_lock_mode                | 1                    |
| innodb_lock_wait_timeout                | 50                   |
| innodb_locks_unsafe_for_binlog          | OFF                  |
| innodb_old_blocks_pct                   | 37                   |
| innodb_old_blocks_time                  | 1000                 |
| innodb_print_all_deadlocks              | OFF                  |
| innodb_status_output_locks              | OFF                  |
| innodb_table_locks                      | ON                   |
| key_cache_block_size                    | 1024                 |
| lock_wait_timeout                       | 31536000             |
| locked_in_memory                        | OFF                  |
| max_write_lock_count                    | 18446744073709551615 |
| metadata_locks_cache_size               | 1024                 |
| metadata_locks_hash_instances           | 8                    |
| performance_schema_max_rwlock_classes   | 40                   |
| performance_schema_max_rwlock_instances | 9102                 |
| query_alloc_block_size                  | 8192                 |
| query_cache_wlock_invalidate            | OFF                  |
| range_alloc_block_size                  | 4096                 |
| skip_external_locking                   | ON                   |
| transaction_alloc_block_size            | 8192                 |
+-----------------------------------------+----------------------+
23 rows in set (0.00 sec)

mysql>
```

#### innodb_autoinc_lock_mode(refman-5.7)

The lock mode to use for generating auto-increment values. Permissible values are 0, 1, or 2, for
traditional, consecutive(adj.连续不断的), or interleaved, respectively. The default setting is 1 (consecutive).

几个语句概念

*  “INSERT-like” statements

All statements that generate new rows in a table, including INSERT, INSERT ... SELECT,
REPLACE, REPLACE ... SELECT, and LOAD DATA. Includes “simple-inserts”, “bulk-inserts”, and
“mixed-mode” inserts.

* “simple-inserts”

Statements for which the number of rows to be inserted can be determined in advance (when the
statement is initially processed). This includes single-row and multiple-row INSERT and REPLACE
statements that do not have a nested subquery, but not INSERT ... ON DUPLICATE KEY
UPDATE.

* “bulk-inserts”

Statements for which the number of rows to be inserted (and the number of required autoincrement values) is not known in advance. This includes INSERT ... SELECT, REPLACE ...
SELECT, and LOAD DATA statements, but not plain INSERT. InnoDB assigns new values for the
AUTO_INCREMENT column one at a time as each row is processed.

* “Mixed-mode inserts”

##### innodb_autoinc_lock_mode = 1 (“consecutive” lock mode)

This is the default lock mode. In this mode, “bulk inserts” use the special AUTO-INC table-level lock
and hold it until the end of the statement. This applies to all INSERT ... SELECT, REPLACE ...
SELECT, and LOAD DATA statements. Only one statement holding the AUTO-INC lock can execute
at a time.

“Simple inserts” (for which the number of rows to be inserted is known in advance) avoid table-level
AUTO-INC locks by obtaining the required number of auto-increment values under the control of a
mutex (a light-weight lock) that is only held for the duration of the allocation process, not until the statement completes.

The exception is for “mixed-mode inserts”, where the user provides explicit values for an
AUTO_INCREMENT column for some, but not all, rows in a multiple-row “simple insert”. For such
inserts, InnoDB allocates more auto-increment values than the number of rows to be inserted.
However, all values automatically assigned are consecutively generated (and thus higher than)
the auto-increment value generated by the most recently executed previous statement. “Excess”
numbers are lost.

“bulk inserts”仍然使用AUTO-INC表级锁,并保持到语句结束.这适用于所有INSERT ... SELECT，REPLACE ... SELECT和LOAD DATA语句。**同一时刻只有一个语句可以持有AUTO-INC锁**。

而“Simple inserts”（要插入的行数事先已知）通过在**mutex（轻量锁）**的控制下获得所需数量的自动递增值来**避免表级AUTO-INC锁**，它只在分配过程的持续时间内保持，而不是直到语句完成。 不使用表级AUTO-INC锁，**除非AUTO-INC锁由另一个事务保持**。如果另一个事务保持AUTO-INC锁，则“简单插入”等待AUTO-INC锁，如同它是一个“批量插入”。

#### 自增id的大小

一个正常大小整数有符号的范围是`-2147483648`到`2147483647`，无符号的范围是`0`到`4294967295`

```sql
insert into tb_user(userID, password, name, phone, address) values('0004','123456','tom', '110', 'beijing');

insert into tb_user values(10, '0004','123456','tom', '110', 'beijing');
```

* 插入超过大值会默认最大，并报错

```java
mysql> insert into tb_user values(2147483648, '0004','123456','tom', '110', 'beijing');
Query OK, 1 row affected, 1 warning (0.00 sec)

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

mysql> insert into tb_user values(2147483648, '0004','123456','tom', '110', 'beijing');
ERROR 1062 (23000): Duplicate entry '2147483647' for key 'PRIMARY'
mysql>
mysql>
mysql>
mysql> insert into tb_user(userID, password, name, phone, address) values('0004','123456','tom', '110', 'beijing');
ERROR 1062 (23000): Duplicate entry '2147483647' for key 'PRIMARY'
mysql>
```

## 分布式ID解决方案

* 完全依赖数据源方式

ID的生成规则，读取控制完全由数据源控制，常见的如数据库的自增长ID，序列号等，或Redis的INCR/INCRBY原子操作产生顺序号等。

* 半依赖数据源方式

ID的生成规则，有部分生成因子需要由数据源（或配置信息）控制，如**snowflake算法**。

* 不依赖数据源方式

ID的生成规则完全由机器信息独立计算，不依赖任何配置信息和数据记录，如常见的UUID，GUID等
实

作者：1黄鹰
链接：https://juejin.im/post/6844903977612476430
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### snowflake算法

snowflake算法的特性是有序、唯一，并且要求高性能，低延迟（每台机器每秒至少生成10k条数据，并且响应时间在2ms以内），要在分布式环境（多集群，跨机房）下使用，因此snowflake算法得到的ID是分段组成的：

* 与指定日期的时间差（毫秒级），41位，够用69年
* 集群ID + 机器ID， 10位，最多支持1024台机器
* 序列，12位，每台机器每毫秒内最多产生4096个序列号

![](../../content/distributed_design/imgs/snow.png)

Java简单实现：<a href='https://github.com/beyondfengyu/SnowFlake/blob/master/SnowFlake.java' target='_blank'>snowflake算法</a>

优点：
* 毫秒数在高位，自增序列在低位，整个ID都是趋势递增的。
* 不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也是非常高的。
* 可以根据自身业务特性分配bit位，非常灵活。

缺点：
* 强依赖机器时钟，如果机器上时钟回拨，会导致发号重复或者服务会处于不可用状态。

## Leaf(美团点评分布式ID生成系统)

<a href="https://tech.meituan.com/2017/04/21/mt-leaf.html" target="_blank">美团Leaf</a>
