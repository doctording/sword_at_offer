---
title: "python上下文管理器"
layout: page
date: 2018-11-14 00:00
---

[TOC]

# Contextor

上下文管理器的常用于一些资源的操作，需要在资源的获取与释放相关的操作。

使用`with`语句当对象`__init__`后

* __enter__  进入对象范围时（一般代码块开始）被调用；
* __exit__  离开对象范围时（代码块结束）被调用；

```python
# -*- coding:utf-8 -*-
import pymysql
import time


class Database(object):

    def __init__(self):
        print("__init__")
        self.dbhost = '127.0.0.1'
        self.port = 3306
        self.dbuser = 'root'
        self.dbpass = ''
        self.dbname = 'test'
        # DB_URI = 'mysql://' + dbuser + ':' + dbpass + '@' + dbhost + '/' + dbname
        self.conn = None

    def connect(self):
        print("connect")
        self.conn = pymysql.connect(host=self.dbhost,
                                port=self.port,
                                user=self.dbuser,
                                password=self.dbpass,
                                database=self.dbname)

    def close(self):
        try:
            print("close")
            self.conn.close()
        except:
            raise ValueError('conn close exception')

    def __enter__(self):
        print("__enter__")
        self.connect()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("__exit__")
        self.close()


if __name__ == '__main__':
    with Database() as db:
        db.conn
        sql = "select id, sno, name from t_user"
        cur = db.conn.cursor()
        cur.execute(sql)
        rows = cur.fetchall()
        print rows
        cur.close()

```

结果如下,且观察MySQL的`Aborted_clients`没有增1

```bash
__init__
__enter__
connect
((1, '21651019', 'mubi'),)
__exit__
close
```
