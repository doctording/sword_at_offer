---
title: "python时间处理相关"
layout: page
date: 2018-11-14 00:00
---

[TOC]

# 时间处理

三个重要的模块：`time`,`datetime`,`calendar`, 时间处理基本上各种开发都要得到(一般在项目的工具包中)，如下是三个时间模块的说明文档

https://docs.python.org/2/library/datetime.html#module-datetime

https://docs.python.org/2/library/time.html

https://docs.python.org/2/library/calendar.html#module-calendar

## time

```python
time.time()
"""
Return the time in seconds since the epoch as a floating point number. Note that even though the time is always returned as a floating point number, not all systems provide time with a better precision than 1 second. While this function normally returns non-decreasing values, it can return a lower value than a previous call if the system clock has been set back between the two calls.
"""
```

```python
# -*- coding:utf-8 -*-
import time

if __name__ == '__main__':
    # 当前时间戳，float类型
    ct = time.time()
    print(ct, type(ct))

    # 当前时间，格式自定义，字符串类型
    local_time = time.localtime(ct)
    local_time_str = time.strftime("%Y-%m-%d %H:%M:%S", local_time)
    print(local_time_str, type(local_time_str))
```

## datetime & time

```python
# -*- coding:utf-8 -*-
import time
import datetime

if __name__ == '__main__':
    t_obj = datetime.datetime(2018, 11, 12, 9, 8, 2)
    print(t_obj, type(t_obj))

    t_str = t_obj.strftime('%Y-%m-%d %H:%M:%S')
    print(t_str, type(t_str))

    timestamp1 = datetime.datetime.now()
    print(timestamp1, type(timestamp1))
    # float 时间戳， 同 time.time()
    un_time = time.mktime(timestamp1.timetuple())
    print(un_time, type(un_time))

    # 秒级别的时间戳
    sec_unixtime = int(un_time)
    # 毫秒级的时间戳
    millis_unixtime = int(round(time.time() * 1000))
    print(sec_unixtime)
    print(millis_unixtime)

```

## calendar

```python
# -*- coding:utf-8 -*-
import calendar

if __name__ == '__main__':
    year = 2018
    month = 11
    day = 11
    cal = calendar.month(year, month)
    print(cal)

    day_code = calendar.weekday(year, month, day)
    week_arr = ['星期一', '星期二', '星期三', '星期四', '星期五', '星期六', '星期日']
    week = week_arr[day_code]
    print(week)
```

## 统计函数执行时间(装饰器)

```python
# -*- coding:utf-8 -*-
import time


def debug_time(func):
    def wrapper():
        print "[DEBUG_TIME]: enter {}()".format(func.__name__)
        # 毫秒级的时间戳
        start_millis_unixtime = int(round(time.time() * 1000))
        # 执行函数
        func()
        end_millis_unixtime = int(round(time.time() * 1000))
        span = end_millis_unixtime - start_millis_unixtime
        print "[DEBUG_TIME]: {} cost {}ms".format(func.__name__, span)
    return wrapper  # 返回包装过函数


@debug_time
def fun_sleep():
    print "func start"
    time.sleep(3)
    print "func end"

if __name__ == '__main__':
    fun_sleep()

```

## 统计函数执行时间(装饰器)2

```python
# -*- coding:utf-8 -*-
import time
import functools

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


@Log.debug_time
def fun_sleep(*args, **kwargs):
    print "func start"
    time.sleep(3)
    print "func end"


@Log.debug_time
def fun_sleep_sh(something):
    print "func start {}".format(something)
    time.sleep(2)
    print "func end {}".format(something)

if __name__ == '__main__':
    fun_sleep()
    fun_sleep_sh("hello")

```
