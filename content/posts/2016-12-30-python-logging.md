---
title: Python logging 基础
slug: python-logging
date: 2016-12-30 12:23:23
tags: [Python, logging]
categories: [Coding]
---
一个例子：
```python
import logging

logger=logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# 添加handler
handler=logging.FileHandler('hello.log')
handler.setLevel(logging.INFO)
# handler可以有多个，所以用add
logger.addHandler(handler)

# 设置格式
formatter=logging.Formatter('[%(asctime)s] [%(levelname)s] %(message)s', '%H:%M:%S')
# formatter只能有一个
logger.setFormatter(formatter)
```
对于库来说，通常使用 *\_\_name\_\_* 来获取 Logger对象，然后只添加一个`NullHandler`，表示默认不会输出任何log: <!--more-->

```python
# 设置默认日志处理方式，避免“未找到处理方法”的警告。
import logging
try:  # Python 2.7+
    from logging import NullHandler
except ImportError:
    class NullHandler(logging.Handler):
        def emit(self, record):
            pass

logging.getLogger(__name__).addHandler(NullHandler())
```

什么情况下会抛出‘未找到处理方法’的异常呢？

如果没有添加NullHandler()，即没有配置任何Handler，日志会被发给root，然后默认下root会怎么处理呢？试验的结果是：向终端输出了异常的信息。

如果添加了NullHandler()，则没有任何输出。这是因为添加了NullHandler()，found不为0，不会去使用保底的手段，又因为root本身的handlers也为空，所以就只调用了NullHandler的handle，于是就什么也输出了。

如果没添加NullHandler()，则会使用保底的lastResort手段，所以会向stderr写入log。

## 基本概念

1. Handler send the log records (created by loggers) to the appropriate destination
2. Fliters provide a finer grained facility for determining which log recoreds to output
3. Formatters specify the layout of log records in the final output

## Logger 对象

Logger不会直接初始化，只能通过`logging.getLogger(name)`获取，多次以同样的`name`调用返回的是同一个对象。

`name`以`.`分隔，表示级别。如`foo.bar.baz`，`foo.bar`,`foo.bar.baz`,`foo.bam`都是`foo`的下一级。logger的分级与Python的模块分级类型，所以推荐的`name`值通常是`_name__`.

1. `Logger.propagate` 
   如果为True，events logged to this logger除了发给handlers attatched to this logger外，还会传递给这个logger的父辈。默认为True。
   如果只对root logger设置handler，这样所有后代logger的事件都会发送给这个handler处理，这样可以避免同一个日志被输出多次。

2. `Logger.setLevel(lvl)`
   Set the threshold for this logger to _lvl_. 小于 *lvl* 的消息会被忽略。
   root logger的默认level为WARNNING，其他logger默认为NOTSET.

3. `Logger.isEnabledFor(lvl)`
   测试 *lvl* 级别的消息是否能被此logger处理

4. `Logger.getEffectiveLevel`
   如果当前logger的level不为`NOTSET`, 则返回此level。 否则向上遍历，直到找到一个level不为`NOTSET`的logger.

5. `Logger.getChild(suffix)`

6. `Logger.debug(msg, *args, **kwargs)`
   msg是一个消息格式化字符串，args是将要格式化的参数，即`msg % args`
   有三个关键字参数: `exc_info`, `stack_info`, `extra`
   extra用来填充`LogRecord.__dict__`，比如：

   ```python
   FORMAT = '%(asctime)-15s %(clientip)s %(user)-8s %(message)s'
   logging.basicConfig(format=FORMAT)
   d = {'clientip': '192.168.0.1', 'user': 'fbloggs'}
   logger = logging.getLogger('tcpserver')
   logger.warning('Protocol problem: %s', 'connection reset', extra=d)
   //输出
   //2006-02-08 22:20:02,165 192.168.0.1 fbloggs  Protocol problem: connection reset
   ```

7. `Logger.hasHandlers`
   检查当前logger是否配置了任何handler，通过循环遍历其所有parent的handlers得到结果，如果有任何一个parent的handlers数组不为空，则返回True.

8. ​

##  LogRecord

Logger对象产生 LogRecord对象，传递给 *Handler*.   Logger和Handler都可以使用 *logging level* 和 *Filter*来决定他们感兴趣的 *LogRecord*对象。当需要output LogRecord时，一个 Handler使用一个 *Formatter*来格式化字符串并把它发送到 I/O流中。

每个Logger记录一组 output Handlers，默认情况下，所有的 Logger也会将他们的输出发给他们的 ancestor Loggers。

## Logging 级别

| Level    | 数值   |
| -------- | ---- |
| CRITICAL | 50   |
| ERROR    | 40   |
| WARNNING | 30   |
| INFO     | 20   |
| DEBUG    | 10   |
| NOTSET   | 0    |

## 模块级函数

1. `logging.getLogger(name=None)`
   如果name=None，则返回root logger。
   因为相同名字返回的是同一个对象，所以logger不应该在程序的不同部分之间传递。

2. `logging.debug(msg, *args, **kwargs)`
   Logs a message with level `DEBUG` on the root logger.

3. `logging.disable(lvl)`

4. `logging.basicConfig(**kwargs)`
   创建一个`StreamHandler`,使用默认的`Formatter`，并把其加入到root logger中。模块级的`debug()`等函数都会自动调用`basicConfig()`，如果没有为root logger设置任何handler的话。


## 最简单的使用：

1. 在主线程的模块中，在其他线程启动之前调用`logging.basicConfig`为root logger设置基本的level和格式

2. 其他模块中直接`import logging`，然后使用模块级的`logging.debug`即可。

## 阅读源码

1. Logger类有一个root属性，指向的是RootLogger(WARNING). 
   RootLogger是Logger的一个子类，且Logger.name默认初始化为‘root'，此外没有区别。所以只要import了logging库，root就已经创建了，且作为所有Logger的一个属性。
2. 每个Logger都有一个parent属性，root.parent=None.
3. 每个Logger都有一个handlers数据和filters数组。
4. Logger的callHandlers方法就是调用当前logger.handlers所有handler的handle方法，将record传给它，然后c=c.parent，继续调用父亲的所有handler，除非当前logger.propagate设为False.
5. 如果不使用basicConfig或者对root添加过任何handler，或者没有调用模块级的logging.debug等函数(它们会自动调用basicConfig)，那么root.handlers也为空。
   此时还有一个保底的lastResort，默认为_StderrHandler。如果这个lastResort也被设为None，那么就会向stderr写入 'No handlers could be found for logger xx'。
   那么什么这个lastResort会被设为None呢？这是新版本的行为，新版本中如果没有找到任何handler就会使用这个保底的handler，而不是是之前的显示’No handler xx‘警告。
6. `logging.basicConfig`
   只有当root.handlers为空时才会起作用。