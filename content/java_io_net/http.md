---
title: "http"
layout: page
date: 2019-06-03 00:00
---

[TOC]

# HTTP的一些重要知识点
】
## http 1.0 / 1.1

## http 连接池

## http chunked

服务端边产生数据边传输(`Transfer-Encoding: chunked`)

1. 当选择分块传输时，响应头中可以不包含`Content-Length`
2. 服务器会先回复一个不带数据的报文（只有响应行和响应头和\r\n），然后开始传输若干个数据
3. 当传输完若干个数据块后，需要再传输一个空的数据块
4. 当客户端收到空的数据块时，则客户端知道数据接收完毕

## 