---
title: "python协程gevent"
layout: page
date: 2019-01-19 00:00
---

[TOC]

# gevent协程使用和效率对比

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
