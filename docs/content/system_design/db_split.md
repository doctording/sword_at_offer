---
title: "10亿级订单系统分库分表"
layout: page
date: 2020-08-24 00:00
---

[TOC]

# 10亿级订单系统分库分表

## 数据存储划分

* 热数据：3个月内的订单数据，查询实时性较高;
* 冷数据A：3个月 ~ 12个月前的订单数据，查询频率不高;
* 冷数据B：1年前的订单数据，几乎不会查询，只有偶尔的查询需求;

可以规划如下：

* 热数据： 使用mysql进行存储，当然需要分库分表；
* 冷数据A: 对于这类数据可以存储在ES中，利用搜索引擎的特性基本上也可以做到比较快的查询；
* 冷数据B: 对于这类不经常查询的数据，可以存放到Hive中；

## MySql 如何分库分表

将不同的业务放到不同的库中，将原来所有压力由同一个库中分散到不同的库中，提升了系统的吞吐量。
* 用户库
* 商品库
* 订单库
