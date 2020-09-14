---
title: "Java http & tcp面试题"
layout: page
date: 2020-03-08 18:00
---

[TOC]

# 网络分层

4层，5层，7层

![](../../content/java_io_net/imgs/net.png)

## OSI 7层网络模型

1. 物理层：主要定义物理设备标准，如网线的接口类型、光纤的接口类型、各种传输介质的传输速率等。它的主要作用是传输比特流（就是由1、0转化为电流强弱来进行传输，到达目的地后再转化为1、0，也就是我们常说的数模转换与模数转换）。这一层的数据叫做比特。
2. 数据链路层：定义了如何让格式化数据以帧为单位进行传输，以及如何让控制对物理介质的访问。该层对连接相邻的通路进行差错控制、数据成帧、同步等控制。检测差错一般采用循环冗余校验(CRC)，纠正差错采用计时器恢复和自动请求重发(ARQ)等技术。
3. 网络层：网络层规定了网络连接的建立、维持和拆除的协议。它的主要功能是利用数据链路层所提供的相邻节点间的无差错数据传输功能，通过路由选择和中继功能，实现两个系统之间的连接。在计算机网络系统中，网络层还具有多路复用的功能。
4. 传输层：定义了一些传输数据的协议和端口号（WWW端口80等），如：TCP（传输控制协议，传输效率低，可靠性强，用于传输可靠性要求高，数据量大的数据）；UDP（用户数据报协议，与TCP特性恰恰相反，用于传输可靠性要求不高，数据量小的数据，如QQ聊天数据就是通过这种方式传输的）。主要是将从下层接收的数据进行分段和传输，到达目的地址后再进行重组。常常把这一层数据叫做段。
5. 会话层：通过传输层(端口号：传输端口接收端口)建立数据传输的通路。主要在你的系统之间发起会话或者接受会话请求（设备之间需要互相认识可以是IP也可以是MAC或者是主机名）。
6. 表示层：可确保一个系统的应用层所发送的信息可以被另一个系统的应用层读取。例如，PC程序与另一台计算机进行通信，其中一台计算机使用扩展二一十进制交换码(EBCDIC)，而另一台则使用美国信息交换标准码（ASCII）来表示相同的字符。如有必要，表示层会通过使用一种通格式来实现多种数据格式之间的转换。
7. 应用层：应用层是OSI参考模型的最高层。其功能是实现应用进程(如用户程序、终端操作员等)之间的信息交换。同时，还具有一系列业务处理所需要的服务功能。

## TCP/IP 4层

1. 应用层：应用程序间沟通的层，如简单电子邮件传输（SMTP）、文件传输协议（FTP）、网络远程访问协议（Telnet）等。
2. 传输层：在此层中，它提供了节点间的数据传送服务，如传输控制协议（TCP）、用户数据报协议（UDP）等，TCP和UDP给数据包加入传输数据并把它传输到下一层中，这一层负责传送数据，并且确定数据已被送达并接收。
3. 网络层：负责提供基本的数据封包传送功能，让每一块数据包都能够到达目的主机（但不检查是否被正确接收），如网际协议（IP）。
4. 网络接口层：对实际的网络媒体的管理，定义如何使用实际网络（如Ethernet、Serial Line等）来传送数据。

![](../../content/java_io_net/imgs/net2.webp)

# url到页面的过程?

## 1 浏览器输入访问地址

当我们开始在浏览器中输入网址的时候，浏览器其实就已经在智能的匹配可能得 url 了，他会从历史记录，书签等地方，找到已经输入的字符串可能对应的 url，然后给出智能提示，让你可以补全url地址。对于 google的chrome 的浏览器，他甚至会直接从缓存中把网页展示出来，就是说，你还没有按下 enter，页面就出来了。

## 2 浏览器查找域名的 IP 地址

1. 请求一旦发起，浏览器首先要做的事情就是解析这个域名，一般来说，浏览器会首先查看本地硬盘的 hosts 文件，看看其中有没有和这个域名对应的规则，如果有的话就直接使用 hosts 文件里面的 ip 地址。

2. 如果在本地的 hosts 文件没有能够找到对应的 ip 地址，浏览器会发出一个 DNS请求到本地DNS服务器 。本地DNS服务器一般都是你的网络接入服务器商提供，比如中国电信，中国移动。

3. 查询你输入的网址的DNS请求到达本地DNS服务器之后，本地DNS服务器会首先查询它的缓存记录，如果缓存中有此条记录，就可以直接返回结果，此过程是递归的方式进行查询。如果没有，本地DNS服务器还要向DNS根服务器进行查询。

4. 根DNS服务器没有记录具体的域名和IP地址的对应关系，而是告诉本地DNS服务器，你可以到域服务器上去继续查询，并给出域服务器的地址。这种过程是迭代的过程。

5. 本地DNS服务器继续向域服务器发出请求，在这个例子中，请求的对象是.com域服务器。.com域服务器收到请求之后，也不会直接返回域名和IP地址的对应关系，而是告诉本地DNS服务器，你的域名的解析服务器的地址。

6. 最后，本地DNS服务器向域名的解析服务器发出请求，这时就能收到一个域名和IP地址对应关系，本地DNS服务器不仅要把IP地址返回给用户电脑，还要把这个对应关系保存在缓存中，以备下次别的用户查询时，可以直接返回结果，加快网络访问。

## 3 浏览器向 web 服务器发送一个 HTTP 请求

拿到域名对应的IP地址之后，浏览器会以一个随机端口（1024<端口<65535）向服务器的WEB程序（常用的有httpd,nginx等）80端口发起TCP的连接请求。这个连接请求到达服务器端后（这中间通过各种路由设备，局域网内除外），进入到网卡，然后是进入到内核的TCP/IP协议栈（用于识别该连接请求，解封包，一层一层的剥开），还有可能要经过Netfilter防火墙（属于内核的模块）的过滤，最终到达WEB程序，最终建立了TCP/IP的连接。

建立了TCP连接之后，发起一个http请求。一个典型的 http request header 一般需要包括请求的方法，例如 GET 或者 POST 等，不常用的还有 PUT 和 DELETE 、HEAD、OPTION以及 TRACE 方法，一般的浏览器只能发起 GET 或者 POST 请求。

## 4 服务器的永久重定向响应

服务器给浏览器响应一个301永久重定向响应，这样浏览器就会访问“http://www.google.com/” 而非“http://google.com/”。

为什么服务器一定要重定向而不是直接发送用户想看的网页内容呢？其中一个原因跟搜索引擎排名有关。如果一个页面有两个地址，就像http://www.yy.com/和http://yy.com/，搜索引擎会认为它们是两个网站，结果造成每个搜索链接都减少从而降低排名。而搜索引擎知道301永久重定向是什么意思，这样就会把访问带www的和不带www的地址归到同一个网站排名下。还有就是用不同的地址会造成缓存友好性变差，当一个页面有好几个名字时，它可能会在缓存里出现好几次。

## 5 浏览器跟踪重定向地址

现在浏览器知道了 "http://www.google.com/"才是要访问的正确地址，所以它会发送另一个http请求。

## 6 服务器处理请求

## 7 服务器返回一个 HTTP 响应

## 8 浏览器显示 HTML

## 9 浏览器发送请求获取嵌入在 HTML 中的资源（如图片、音频、视频、CSS、JS等等）

# HTTP 1.0 1.1 2.0

## HTTP协议

## HTTP 1.0

HTTP 1.0规定浏览器与服务器只保持**短暂连接**，浏览器的每次请求都需要与服务器建立一个TCP连接，服务器完成请求处理后立即断开TCP连接，服务器不跟踪每个客户也不记录过去的请求。HTTP 是无状态的，服务端无需保存状态数据，能较少服务器的CPU及内存消耗。

## HTTP 1.1

HTTP 1.1的**持久连接**，也需要增加新的请求头来帮助实现，例如，Connection请求头的值为Keep-Alive时，客户端通知服务器返回本次请求结果后保持连接；Connection请求头的值为close时，客户端通知服务器返回本次请求结果后关闭连接。HTTP 1.1还提供了与身份认证、状态管理和Cache缓存等机制相关的请求头和响应头。

请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟。例如：一个包含有许多图像的网页文件的多个请求和应答可以在一个连接中传输，但每个单独的网页文件的请求和应答仍然需要使用各自的连接。HTTP 1.1还允许客户端不用等待上一次请求结果返回，就可以发出下一次请求，但服务器端必须按照接收到客户端请求的先后顺序依次回送响应结果，以保证客户端能够区分出每次请求的响应内容。

HTTP 1.1增加host字段:在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。此外，服务器应该接受以绝对路径标记的资源请求。

### Cookie

<a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies">mozilla doc of cookie</a>

An HTTP cookie (web cookie, browser cookie) is a small piece of data that a server sends to the user's web browser. The browser may store it and send it back with the next request to the same server. Typically, it's used to tell if two requests came from the same browser — keeping a user logged-in, for example. It remembers stateful information for the **stateless** HTTP protocol.

Cookie总是保存在客户端中，按在客户端中的存储位置，可分为内存Cookie和硬盘Cookie。内存Cookie由浏览器维护，保存在内存中，浏览器关闭后就消失了，其存在时间是短暂的。硬盘Cookie保存在硬盘里，有一个过期时间，除非用户手工清理或到了过期时间，硬盘Cookie不会被删除，其存在时间是长期的。所以，按存在时间，可分为非持久Cookie和持久Cookie。Cookie由服务端生成，响应给客户端，客户端存储后下次访问可以带上，发送给服务端。

![](../../content/java_io_net/imgs/http_cookie.png)

#### Cookie的作用

* Session management(会话管理：登录信息，购物车等类似功能)

Logins, shopping carts, game scores, or anything else the server should remember

* Personalization（个性化设置，如用户偏好主题等）

User preferences, themes, and other settings

* Tracking（追踪用户行为）

Recording and analyzing user behavior

#### 如何创建Cookie

When receiving an HTTP request, a server can send a `Set-Cookie header` with the response. The cookie is usually stored by the browser, and then the cookie is sent with requests made to the same server inside a `Cookie HTTP header`. An expiration date or duration can be specified, after which the cookie is no longer sent. Additionally, restrictions to a specific domain and path can be set, limiting where the cookie is sent.

#### Cookie的缺陷

* Cookie会被附加在每个HTTP请求中，所以无形中增加了流量
* 由于在HTTP请求中的cookie是明文传递的，所以安全性成问题(除非用HTTPS)
* Cookie的大小限制在4KB左右。对于复杂的存储需求来说是不够用的

### Session

A session creates a file in a temporary directory on the server where registered session variables and their values are stored. This data will be available to all pages on the site during that visit. A session ends when the user closes the browser or after leaving the site, the server will terminate the session after a predetermined period of time, commonly 30 minutes duration.

session存在于服务端，会增加服务端存储压力；客户端浏览器访问服务器的时候，服务端把客户端信息以某种形式记录在服务器上

Session生成后，只要用户继续访问，服务器就会更新Session的最后访问时间，并维护该Session。用户每访问一次，无论是否读写Session，服务器都认为该用户的Session活跃（active）了一次。由于有越来越多的用户访问服务器，因此Session也会越来越多。为防止内存溢出，服务器会把长时间没有活跃的Session从内存中删除。这个时间就是Session的超时时间。如果超过了超时时间没有访问过服务器，Session就自动失效。

#### Session 需要借助 Cookie

虽然Session保存在服务器，对客户端是透明的，他的正常运行任然需要客户端浏览器的支持。这是因为Session需要使用Cookie作为识别标志。HTTP协议是无状态的，Session不能依据HTTP链接来判断是否为同一客户，因此服务器先客户端浏览器发送一个名为JSESSIONID的Cookie，他的值为该Session的id（也就是HTTPSession.getId()的返回值）。Session依据Cookie来识别是否为同一用户。

该Cookie为服务器自动生成的，它的maxAge属性一般为-1，表示仅当前浏览器内有效，并且个浏览器窗口间不共享，关闭浏览器就会失效。

因此同一机器的两个浏览器窗口访问服务器时，会生成两个不同的Session、但是由浏览器窗口内的链接、脚本等打开的新窗口（也就是说不是双击=桌面浏览器图标打开 的窗口）除外。这类子窗口会共享父窗口的Cookie，因此会共享一个Session。

## HTTP 2.0

<a href='https://http2.github.io/http2-spec/'>http2-spec</a>

HTTP/2 enables a more efficient use of network resources and a reduced perception of latency by introducing header field compression and allowing multiple concurrent exchanges on the same connection. It also introduces unsolicited push of representations from servers to clients.

关键词：压缩，IO多路复用

## 1.0和1.1和2.0之间的区别

## HTTPS

# 什么是TCP粘包/拆包,解决办法？

## 什么是粘包、拆包？

假设客户端向服务端连续发送了两个数据包，用packet1和packet2来表示，那么服务端收到的数据可以分为三种，现列举如下：

* 第一种情况，接收端正常收到两个数据包，即没有发生拆包和粘包的现象

![](../../content/java_io_net/imgs/tcp1.png)

* 第二种情况，接收端只收到一个数据包，由于TCP是不会出现丢包的，所以这一个数据包中包含了发送端发送的两个数据包的信息，这种现象即为粘包

![](../../content/java_io_net/imgs/tcp2.png)

* 第三种情况，这种情况有两种表现形式，如下图。接收端收到了两个数据包，但是这两个数据包要么是不完整的，要么就是多出来一块，这种情况即发生了拆包和粘包

![](../../content/java_io_net/imgs/tcp3.png)

![](../../content/java_io_net/imgs/tcp4.png)

## 为什么会发生TCP粘包、拆包？

1. 应用程序写入的数据大于套接字缓冲区大小，这将会发生拆包。
2. 应用程序写入数据小于套接字缓冲区大小，网卡将应用多次写入的数据发送到网络上，这将会发生粘包。
3. 进行MSS（最大报文长度）大小的TCP分段，当TCP报文长度-TCP头部长度>MSS的时候将发生拆包。
4. 接收方法不及时读取套接字缓冲区数据，这将发生粘包。

## 粘包、拆包解决办法

TCP本身是面向流的，作为网络服务器，如何从这源源不断涌来的数据流中拆分出或者合并出有意义的信息呢？通常会有以下一些常用的方法：

1. 发送端给每个数据包添加包首部，首部中应该至少包含数据包的长度，这样接收端在接收到数据后，通过读取包首部的长度字段，便知道每一个数据包的实际长度了。
2. 发送端将每个数据包封装为固定长度（不够的可以通过补0填充），这样接收端每次从接收缓冲区中读取固定长度的数据就自然而然的把每个数据包拆分开来。
3. 可以在数据包之间设置边界，如添加特殊符号；这样，接收端通过这个边界就可以将不同的数据包拆分开。

# TCP半连接，半关闭，半打开？

# TCP长连接/短连接

## 长连接

在一个TCP连接上可以连续发送多个数据包，在TCP连接保持期间，如果没有数据包发送，需要双方发检测包以维持此连接，一般需要自己做在线维持（不发生RST包和四次挥手）

```java
连接 → 数据传输 → 保持连接(心跳) → 数据传输→保持连接(心跳) → …… → 关闭连接（一个TCP连接通道多个读写通信）；
```

<font color='red'>长连接在没有数据通信时需要定时发送数据包(心跳)，以维持连接状态；</font>

### Keep-Alive & 心跳包

* TCP keepalive处于传输层，由操作系统负责，能够判断进程存在，网络通畅，但无法判断进程阻塞或死锁等问题
    * keepalive只能检测连接是否存活，不能检测连接是否可用。比如服务器因为负载过高导致无法响应请求但是连接仍然存在，此时keepalive无法判断连接是否可用。
    * 如果TCP连接中的另一方因为停电突然断网，我们并不知道连接断开，此时发送数据失败会进行重传，由于重传包的优先级要高于keepalive的数据包，因此keepalive的数据包无法发送出去。只有在长时间的重传失败之后我们才能判断此连接断开了。
* 客户端与服务器之间有四层代理或负载均衡，即在传输层之上的代理，只有传输层以上的数据才被转发

基于以上原因，有时候还是需要应用程序自己去设计心跳规则的。可以服务端负责周期发送心跳包，检测客户端，也可以客户端负责发送心跳包，或者服服务端和客户端同时发送心跳包。

## 短连接

短连接是指通信双方有数据交互时，就建立一个TCP连接，数据发送完成后，则断开此TCP连接（管理起来比较简单，存在的连接都是有用的连接，不需要额外的控制手段）；

```java
连接 → 数据传输 → 关闭连接；
```
