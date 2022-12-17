---
title: 'Python 中的时间模块'
slug: python-time-datetime
tags:
  - Python
categories:
  - Coding
date: 2016-10-02 14:38:06
---


### time

time 模块比较简单, 常用的函数只有几个, 但是这个模块中的函数名都很奇怪, 估计是模仿 *inx 的时间函数风格, 也查不到到底是什么的缩写, 那就不管它好了.
<!--more-->
##### time.sleep(seconds)

使程序的执行推迟一定时间, seconds 可以为小数.

##### time.gmtime(seconds=None)

把 从`Epoch`之后经过的秒数 转换为time tuple (`time.struct_time`类型), 未提供`seconds`则使用当前的时间戳. (`time.time()`的返回值?)

{% blockquote Wikipedia https://en.wikipedia.org/wiki/Unix_time %}

**Unix time** (also known as **POSIX time** or **Epoch time**) is a system for describing instants in time, defined as the number of seconds that have elapsed since 00:00:00 [Coordinated Universal Time](https://en.wikipedia.org/wiki/Coordinated_Universal_Time) (UTC), Thursday, 1 January 1970

{% endblockquote %}



所以常说的时间戳或者 `Epoch` , 都是指的 UTC 时间, 如果要转换为当地时间, 则需要根据时区的偏移量进行运算.

##### time.localtime(seconds=None)

与`gmtime()`功能基本相同, 但是会先将当前时间戳转换为当前的 UTC 时间, 然后根据本地时区对UTC时间的偏移量, 计算得到本地的当前时间. (先使用`gmtime()`, 然后再加上偏移量?)

##### time.mktime()

将 本地时间的 time tuple 转为 seconds since Epoch, 是 `time.localtime()`的逆操作, 估计是现根据本地时间得到 UTC 的 time tuple, 然后计算 seconds.

##### time.ctime() 与 time.asctime()

这两个函数将 秒数/time tuple 转换为字符串, 基本没用, 因为不可以自己定义格式.

##### time.strftime() 与 time.strptime()

一个根据格式输出time tuple 的字符串表示, 一个根据格式把字符串解析为 time tuple.

### datetime



### 参考链接

- [python中datetime模块详解](http://www.jianshu.com/p/113bd56f7b56)