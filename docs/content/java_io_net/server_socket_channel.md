---
title: "Java ServerSocketChannel"
layout: page
date: 2019-03-18 00:00
---

[TOC]

# ServerSocketChannel

## 阻塞方式的`ServerSocketChannel`

```java
 public static void main(String[] args) throws IOException {
    ServerSocketChannel ssc = ServerSocketChannel.open();
    ssc.socket().bind(new InetSocketAddress("127.0.0.1", 8889));
    ssc.configureBlocking(true);

    System.out.println("server started, listening on :" + ssc.getLocalAddress());

    while(true) {
        //  ssc.configureBlocking(true);则 accept会一直阻塞直到接收到连接
        SocketChannel sc = ssc.accept();
        System.out.println("accept SocketChannel:" + sc.getRemoteAddress());
        try {
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.clear();
            int len = sc.read(buffer);

            if(len != -1) {
                System.out.println("receive client data:" + new String(buffer.array(), 0, len));
            }
            ByteBuffer bufferToWrite = ByteBuffer.wrap("I am Server".getBytes());
            sc.write(bufferToWrite);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(sc != null) {
                try {
                    sc.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

* accept方法一直阻塞等待连接的到来

### 阻塞方式register selector会报错

```java
Selector selector = Selector.open();
// selector 注册感兴趣的事情：连接事件
ssc.register(selector, SelectionKey.OP_ACCEPT);
```

* 报错如下

```java
Exception in thread "main" java.nio.channels.IllegalBlockingModeException
	at java.nio.channels.spi.AbstractSelectableChannel.register(AbstractSelectableChannel.java:201)
	at java.nio.channels.SelectableChannel.register(SelectableChannel.java:280)
	at io.Server.main(Server.java:28)
```

* 源码

```java
/**
    * Registers this channel with the given selector, returning a selection key.
    *
    * <p>  This method first verifies that this channel is open and that the
    * given initial interest set is valid.
    *
    * <p> If this channel is already registered with the given selector then
    * the selection key representing that registration is returned after
    * setting its interest set to the given value.
    *
    * <p> Otherwise this channel has not yet been registered with the given
    * selector, so the {@link AbstractSelector#register register} method of
    * the selector is invoked while holding the appropriate locks.  The
    * resulting key is added to this channel's key set before being returned.
    * </p>
    *
    * @throws  ClosedSelectorException {@inheritDoc}
    *
    * @throws  IllegalBlockingModeException {@inheritDoc}
    *
    * @throws  IllegalSelectorException {@inheritDoc}
    *
    * @throws  CancelledKeyException {@inheritDoc}
    *
    * @throws  IllegalArgumentException {@inheritDoc}
    */
public final SelectionKey register(Selector sel, int ops,
                                    Object att)
    throws ClosedChannelException
{
    synchronized (regLock) {
        if (!isOpen())
            throw new ClosedChannelException();
        if ((ops & ~validOps()) != 0)
            throw new IllegalArgumentException();
        if (blocking)
            throw new IllegalBlockingModeException();
        SelectionKey k = findKey(sel);
        if (k != null) {
            k.interestOps(ops);
            k.attach(att);
        }
        if (k == null) {
            // New registration
            synchronized (keyLock) {
                if (!isOpen())
                    throw new ClosedChannelException();
                k = ((AbstractSelector)sel).register(this, ops, att);
                addKey(k);
            }
        }
        return k;
    }
}
```

### 附：linux socket accept方法

```cpp
#include <sys/types.h>

#include <sys/socket.h>

int accept(int sockfd,struct sockaddr *addr,socklen_t *addrlen);
```

`accept()`系统调用主要用在基于连接的套接字类型，比如SOCK_STREAM和SOCK_SEQPACKET。它提取出所监听套接字的等待连接队列中第一个连接请求，创建一个新的套接字，并返回指向该套接字的文件描述符。

一般`accept()`为阻塞函数，当监听socket调用accept()时，它先到自己的receive_buf中查看是否有连接数据包；若有，把数据拷贝出来，删掉接收到的数据包，创建新的socket与客户发来的地址建立连接；若没有，就阻塞等待；

## 非阻塞方式

```java
public static void main(String[] args) throws IOException {
    ServerSocketChannel ssc = ServerSocketChannel.open();
    ssc.socket().bind(new InetSocketAddress("127.0.0.1", 8889));
    ssc.configureBlocking(false);

    System.out.println("server started, listening on :" + ssc.getLocalAddress());

    while(true) {
        // 非阻塞的accept: 如果没有连接，则返回null
        SocketChannel sc = ssc.accept();
        if(sc != null) {
            System.out.println("accept SocketChannel:" + sc.getRemoteAddress());
            try {
                ByteBuffer buffer = ByteBuffer.allocate(1024);
                buffer.clear();
                int len = sc.read(buffer);

                if (len != -1) {
                    System.out.println("receive client data:" + new String(buffer.array(), 0, len));
                }
                ByteBuffer bufferToWrite = ByteBuffer.wrap("I am Server".getBytes());
                sc.write(bufferToWrite);
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (sc != null) {
                    try {
                        sc.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```

### Selector处理事件

```java
public static void main(String[] args) throws IOException {
    ServerSocketChannel ssc = ServerSocketChannel.open();
    ssc.socket().bind(new InetSocketAddress("127.0.0.1", 8889));
    ssc.configureBlocking(false);

    System.out.println("server started, listening on :" + ssc.getLocalAddress());

    // Select 轮询监听channel事件（这里是注册连接事件）
    Selector selector = Selector.open();
    ssc.register(selector, SelectionKey.OP_ACCEPT);

    while (true) {
        // selector.select(); 阻塞等待连接事件
        selector.select();
        Set<SelectionKey> keys = selector.selectedKeys();
        Iterator<SelectionKey> it = keys.iterator();
        while (it.hasNext()) {
            SelectionKey key = it.next();
            it.remove();
            // 处理连接事件
            handleConnect(key);
        }
    }
}

private static void handleConnect(SelectionKey key) {
    if(key.isAcceptable()) {
        try {
            ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
            SocketChannel sc = ssc.accept();
            System.out.println("accept a client:" + sc.getRemoteAddress());
            // 继续注册socket的读事件
            sc.configureBlocking(false);
            sc.register(key.selector(), SelectionKey.OP_READ );
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
        }
    } else if (key.isReadable()) {
        SocketChannel sc = null;
        try {
            sc = (SocketChannel)key.channel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.clear();
            int len = sc.read(buffer);
            if(len != -1) {
                System.out.println(new String(buffer.array(), 0, len));
            }
            ByteBuffer bufferToWrite = ByteBuffer.wrap("I am Server".getBytes());
            sc.write(bufferToWrite);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(sc != null) {
                try {
                    sc.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
