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


## Leaf(美团点评分布式ID生成系统)

<a href="https://tech.meituan.com/2017/04/21/mt-leaf.html" target="_blank">美团Leaf</a>
