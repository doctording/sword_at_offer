---
title: "Tomcat"
layout: page
date: 2020-07-12 00:00
---

[TOC]

# Tomcat

直接认识：一个URL访问，经过`Tomcat`处理请求，将请求传递给SpringBoot/SpringMvc的Controller

## 几个问题

1. 为什么说Tomcat是一个Servlet容器？
2. Tomcat部署应用有几种方式？
3. War包和Jar包的区别？
4. Tomcat是怎么处理请求的，处理请求的流程是怎么样的？
5. Tcp,Http,Socket,Tomcat之间的关系是什么？

## Tomcat是一个Servlet容器？

```java
class Tomcat{
    List<Servlet> servlets;
}
```

### Servlet

* Servlet在应用中的架构位置

![](../../content/distributed_design/imgs/servlet-arch.jpg)

* Servlet架构图

![](../../content/distributed_design/imgs/Servlet-LifeCycle.jpg)

1. 第一个到达服务器的 HTTP 请求被委派到 Servlet 容器。
2. Servlet 容器在调用 service() 方法之前加载 Servlet。
3. 然后 Servlet 容器处理由多个线程产生的多个请求，每个线程执行一个单一的 Servlet 实例的 service() 方法。

* Servlet生命周期

Servlet 生命周期可被定义为从创建直到毁灭的整个过程。以下是 Servlet 遵循的过程：

1. Servlet 通过调用 init () 方法进行初始化。
2. Servlet 调用 service() 方法来处理客户端的请求。
3. Servlet 通过调用 destroy() 方法终止（结束）。
4. 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。

`service()`方法是执行实际任务的主要方法。Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。

每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 `doGet`、`doPost`、`doPut`，`doDelete`等方法。

* doGet方法

```java
public void doGet(HttpServletRequest request,
                  HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```

* `doPost()`方法

```java
public void doPost(HttpServletRequest request,
                   HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```
