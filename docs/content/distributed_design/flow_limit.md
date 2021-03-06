---
title: "限流"
layout: page
date: 2020-07-06 00:00
---

[TOC]

# 限流

## 常见限流算法

### 计数器（固定窗口）算法

计数器算法是使用计数器在周期内累加访问次数，当达到设定的限流值时，触发限流策略。下一个周期开始时，进行清零，重新计数。

此算法在单机还是分布式环境下实现都非常简单，使用redis的incr原子自增性和线程安全即可轻松实现。

计算器算法的缺点举例：

假设有一个恶意用户，在0:59时，瞬间发送了99个请求，并且1:01又瞬间发送了99个请求，那么其实这个用户在这1秒里面，瞬间发送了198个请求。我们规定的是1分钟最多100个请求，也就是每秒钟最多1.7个请求；用户通过在时间窗口的重置节点处突发请求，可以瞬间超过我们的速率限制。

### 滑动窗口算法

滑动窗口算法是将时间周期分为N个小周期，分别记录每个小周期内访问次数，并且根据时间滑动删除过期的小周期。同时整个周期是有限制的，避免了固定窗口的问题。

保证任意请求的1秒间隔内的容量不超过设定的阈值；

### 漏桶算法

漏桶算法是访问请求到达时直接放入漏桶，如当前容量已达到上限（限流值），则进行丢弃（触发限流策略）。漏桶以固定的速率进行释放访问请求（即请求通过），直到漏桶为空。

### 令牌桶算法

令牌桶算法是程序以r（r=时间周期/限流值）的速度向令牌桶中增加令牌，直到令牌桶满，请求到达时向令牌桶请求令牌；如获取到令牌则通过请求，否则触发限流策略。

## 通过限制单位时间段内的调用量来限流（控制访问速率）

## 通过限制系统的并发调用程度来限流（控制并发数量）
