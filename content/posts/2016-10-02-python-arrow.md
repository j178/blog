---
title: 'Arrow: better dates and times for Python'
date: 2016-10-02 00:22:47
tags: [Python, arrow]
categories: [Coding]
---

Python 中时间处理的库太多, 基本的有`time`, 复杂的有`datetime`, 日历处理有`Calendar`, 各种时区信息的库... 各个库中函数的命名也十分让人崩溃,比如`time`中的`ctime()`, `asctime()`, `strftime()`, `strptime()`, `localtime()`你能记得住这些函数分别接受什么参数, 起什么作用吗? 我觉得挺崩溃的. <!--more-->

Arrow 是一个第三方时间日期处理库, 提供了非常简洁优雅的接口, 我们首先来分析一下这个库的功能模块.

#### api.py

向外提供了四个接口: `get()`, `now()`, `utcnow()`, `factory()`. 前三个都是代理到 `ArrowFactory` 相应的方法上, `factory()` 返回一个 `ArrowFactory` 实例.

#### facotry.py

`ArrowFactory` 干的事很少, 还是那三个工厂方法 `get()`,`now()`, `utcnow()`, 都返回一个 `Arrow` 实例, 代表一个时间点.
`get()`由`ArrowFactory`实现, 其他方法由`Arrow`实现.

#### arrow.py

`Arrow` 实现了 `datetime` 接口, 是一个标准的 `datetime` 对象, 而且增加了许多功能.

##### 1.构造函数:

```python
def __init__(self, year, month, day, hour=0, minute=0, second=0, microsecond=0, tzinfo=None)
```

如果`tzinfo == None`, 则默认为 UTC 时间.

`tzinfo` 参数接受四种形式:
- A `tzinfo` object.
- A `str` describing a timezone, similar to 'US/Pacific', or 'Europe/Berlin'.
- A `str` in ISO-8601 style, as in '+07:00'.
- A `str`, one of the following:  'local', 'utc', 'UTC'.

##### 2. Arrow.utcnow() 与 Arrow.now(tzinfo=None)

这两个工厂方法比较简单. `utcnow()`调用`datatime.utcnow()`获取 UTC 时间, 然后构造`Arrow`对象.

`now()`接受一个`tzinfo`参数, 如果为空则默认为 `dateutil_tz.tzlocal()`, 即本地时间.

##### 3. Arrow.get()



### 参考链接

- [Arrow: better dates and times for Python](http://crsmithdev.com/arrow/)