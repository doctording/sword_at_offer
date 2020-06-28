---
title: "中断"
layout: page
date: 2020-06-27 00:00
---

[TOC]

# 中断

一台典型的个人PC中，中断结构如下图：

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_io_net/imgs/interrupt.png)

设备完成工作，产生一个中断，他是通过在分配给它的一条总线信号线上置起信号而产生中断的。该信号主板上的中断控制器芯片检测到，由中断控制器芯片决定做什么。

在总线上置起中断信号，中断信号导致CPU停止当前正在做的工作并且开始做其它的事情。地址线上的数字被用做指向一个成为**中断向量(interrupt vector)**的表格的所用，以便读取一个新的程序计数器。这个程序计数器指向相应的中断服务过程的开始。
