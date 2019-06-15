---
title: "http"
layout: page
date: 2019-06-10 00:00
---

[TOC]

# HTTP的一些重要知识点

## http 1.0 / 1.1

HTTP的长连接和短连接本质上是TCP长连接和短连接。HTTP属于应用层协议，在网络层使用IP协议，主要解决网络路由和寻址问题；在传输层使用TCP协议，主要解决如何在IP层之上可靠的传递数据包，使在网络上的另一端收到发端发出的所有包，并且顺序与发出顺序一致，具有**可靠**、**面向连接**的特点。

### 长连接、短连接

在HTTP/1.0中，默认使用的是**短连接**。也就是说，浏览器和服务器每进行一次HTTP操作，就建立一次连接，任务结束就中断连接。如果客户端浏览器访问的某个HTML或其他类型的 Web页中包含有其他的Web资源，如JavaScript文件、图像文件、CSS文件等，每遇到这样一个Web资源，就会建立一个HTTP会话。

从HTTP/1.1起，默认使用的是长连接，用以保持连接特性。使用长连接的HTTP协议，会在响应头有加入：`Connection:keep-alive`。在使用长连接的情况下，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的 TCP连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的连接。Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。HTTP协议的长连接和短连接，实质上是TCP协议的长连接和短连接。实现HTTP长连接必须要客户端和服务端都支持。

## https

## http chunked

服务端边产生数据边传输(`Transfer-Encoding: chunked`)

1. 当选择分块传输时，响应头中可以不包含`Content-Length`
2. 服务器会先回复一个不带数据的报文（只有响应行和响应头和\r\n），然后开始传输若干个数据
3. 当传输完若干个数据块后，需要再传输一个空的数据块
4. 当客户端收到空的数据块时，则客户端知道数据接收完毕

### Wireshark 抓包实例

http://www.httpwatch.com/httpgallery/chunked/chunkedimage.aspx

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_io_net/imgs/chunk.png)

参考： https://www.httpwatch.com/httpgallery/

## http 连接池

## 打开一个网页的全过程

## ping
