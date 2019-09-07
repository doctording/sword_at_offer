---
title: "Java 多线程socket"
layout: page
date: 2019-08-19 00:00
---

[TOC]

# 多线程(线程池)socket服务端

给每个 socket 客户端分配一个线程

* server

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Author mubi
 * @Date 2019/8/19 21:57
 */
public class Server extends Thread{
    private Socket client;

    public Server(Socket c, String name) {
        this.client = c;
        this.setName(name);
    }

    @Override
    public void run() {
        try {
            BufferedReader in = new BufferedReader(new InputStreamReader(
                    client.getInputStream()));
            PrintWriter out = new PrintWriter(client.getOutputStream());
            // Mutil User but can't parallel

            while (true) {
                String str = in.readLine();
                System.out.println(str);
                out.println("has receive....");
                out.flush();
                if (str.equals("end")) {
                    break;
                }
            }
            client.close();
        } catch (IOException ex) {
        } finally {
        }
    }

    public static void main(String[] args) throws IOException {
        ServerSocket server = new ServerSocket(8888);  //绑定端口
        while (true) {
            // transfer location change Single User ofr Multi User

            Server mc = new Server(server.accept(), "thread_" + System.currentTimeMillis());  //阻塞进程，等待客户端的连接
            mc.start();
        }
    }
}

```

* client

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Author mubi
 * @Date 2019/8/19 21:57
 */
public class Client {
    static Socket server;

    public static void main(String[] args) throws Exception {
        server = new Socket(InetAddress.getLocalHost(), 8888);
        BufferedReader in = new BufferedReader(new InputStreamReader(
                server.getInputStream()));
        PrintWriter out = new PrintWriter(server.getOutputStream());
        BufferedReader wt = new BufferedReader(new InputStreamReader(System.in));
        while (true) {
            String str = wt.readLine();
            out.println(str);
            out.flush();
            if (str.equals("end")) {
                break;
            }
            System.out.println(in.readLine());
        }
        server.close();
    }
}

```

* socket 服务端 / 客户端 交互流程

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_io/imgs/socket_server_client.png)

* 查看服务端运行的线程情况

![](https://raw.githubusercontent.com/doctording/sword_at_offer/master/content/java_io/imgs/socket.png)
