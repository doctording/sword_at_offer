---
title: "python协程gevent"
layout: page
date: 2019-01-19 00:00
---

[TOC]

# gevent协程使用和效率对比

## 协程的概念

协程，又称微线程，纤程。英文名Coroutine。一句话说明什么是协程：<font color='red'>协程是一种用户态的轻量级线程，是一种用户态内的上下文切换技术</font>

协程存在的意义：对于多线程应用，CPU通过切片的方式来切换线程间的执行，线程切换时需要耗时（内核保存状态，下次继续）。协程，则只使用一个线程，在一个线程中规定某个代码块执行顺序。

协程拥有自己的寄存器上下文和栈。协程调度切换时，将寄存器上下文和栈保存到其它地方，在切回来的时候，恢复先前保存的寄存器上下文和栈。因此：协程能保留上一次调用时的状态（即所有局部状态的一个特定组合），每次过程重入时，就相当于进入上一次调用的状态，换种说法：进入上一次离开时所处逻辑流的位置。

适用场景：在python有GIL（Global Interpreter Lock 全局解释器锁）在同一时间只有一个线程在工作，所以：如果一个线程里面I/O操作特别多，协程就比较适用;

### 协程实现生产者，消费者

```python
import time

def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        time.sleep(1)
        r = '200 OK'

def produce(c):
    c.next()
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

if __name__=='__main__':
    c = consumer()
    produce(c)
```

* 输出

```python
[PRODUCER] Producing 1...
[CONSUMER] Consuming 1...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 2...
[CONSUMER] Consuming 2...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 3...
[CONSUMER] Consuming 3...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 4...
[CONSUMER] Consuming 4...
[PRODUCER] Consumer return: 200 OK
[PRODUCER] Producing 5...
[CONSUMER] Consuming 5...
[PRODUCER] Consumer return: 200 OK
```

* 程序说明

注意到consumer函数是一个generator（生成器），把一个consumer传入produce后：

* 首先调用c.next()启动生成器；
* 然后，一旦生产了东西，通过c.send(n)切换到consumer执行；
* consumer通过yield拿到消息，处理，又通过yield把结果传回；
* produce拿到consumer处理的结果，继续生产下一条消息；
* produce决定不生产了，通过c.close()关闭consumer，整个过程结束。

整个流程无锁，由一个线程执行，produce和consumer协作完成任务，所以称为“协程”，而非线程的抢占式多任务。

### 协程优点

* 无需线程上下文切换的开销
* 无需原子操作锁定及同步的开销
* 方便切换控制流，简化编程模型
* 高并发+高扩展性+低成本：一个CPU支持上万的协程都不是问题。所以很适合用于高并发处理。

### 协程缺点

* 无法利用多核资源：协程的本质是个单线程,它不能利用多核,协程需要和进程配合才能运行在多CPU上.当然我们日常所编写的绝大部分应用都没有这个必要，除非是cpu密集型应用。

* 进行阻塞（Blocking）操作（如IO时）会阻塞掉整个程序

## 网络IO密集型

```python
#coding=utf-8
import time
import json
import os
import requests
import threading
from threading import Thread
from multiprocessing import Process, Pool
import logging
import gevent
import gevent.monkey
import functools
import threadpool
import time, random
import Queue

HTTP_SUCCESS_CODE = 200

gevent.monkey.patch_all()


class Log:

    @classmethod
    def debug_time(cls, func):

        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print "[DEBUG_TIME]: enter {}()".format(func.__name__)
            # 毫秒级的时间戳
            start_millis_unixtime = int(round(time.time() * 1000))
            # 原函数调用
            func(*args, **kwargs)
            end_millis_unixtime = int(round(time.time() * 1000))
            span = end_millis_unixtime - start_millis_unixtime
            print "[DEBUG_TIME]: {} cost {}ms".format(func.__name__, span)

        # 返回一个包装函数
        return wrapper

def http_request():
    url = "http://www.baidu.com"
    try:
        rsp = requests.get(url, timeout=1.0)
        if rsp.status_code == HTTP_SUCCESS_CODE:
            return 1
        return 0
    except Exception as e:
        logging.error(e.message)
        return 0

@Log.debug_time
def line_net_io(n):
    success_cnt  = 0
    for i in range(n):
        success_cnt = success_cnt + http_request()

    print("url successful cnt: {}".format(success_cnt))

@Log.debug_time
def coroutine_net_io(n):
    def func():
        return http_request()

    jobs = [gevent.spawn(func) for i in range(n)]
    gevent.joinall(jobs)
    success_cnt = sum([job.value for job in jobs])

    print("url successful cnt: {}".format(success_cnt))

class MyThread(threading.Thread):
    def __init__(self, func, args=None, name=None):
        threading.Thread.__init__(self)
        self.name = name
        self.func = func
        self.args = args
        if self.args:
            self.result = self.func(*self.args)
        else:
            self.result = self.func()

    def get_result(self):
        try:
            return self.result
        except Exception:
            return None

@Log.debug_time
def thread_net_io(n):
    def func():
        return http_request()

    l = []
    for i in range(n):
        p = MyThread(func)
        l.append(p)
        p.start()
    for p in l:
        p.join()

    success_cnt = sum([p.get_result() for p in l])
    print("url successful cnt: {}".format(success_cnt))

@Log.debug_time
def thread_pool_net_io(n):
    def func(i):
        return http_request()

    def print_result(request, result):
        print result

    pool_cnt = 10
    task_pool = threadpool.ThreadPool(pool_cnt)
    requests = threadpool.makeRequests(func, range(n), print_result)
    for req in requests:
        task_pool.putRequest(req)
    task_pool.wait()


if __name__ == '__main__':
    n = 10
    line_net_io(n)
    coroutine_net_io(n)
    thread_net_io(n)
    thread_pool_net_io(n)

```

对网络密集型：协程、多线程，都远优于线性执行

1. 等待IO而阻塞

2. 多线程进程可以利用IO阻塞等待时的空闲时间执行其它线程，提升效率

3. 每个协程遇到IO阻塞的时候就把控制权交给最顶层的协程，它会看哪个个协程的IO event已经完成，就将控制权给它；所有的IO都阻塞，才阻塞
