---
title: Python 协程基础
date: 2016-12-25 00:53:26
tags: [Python, 协程]
categories: 编程
---

## 基础

- 定义协程(coroutine)的两种方式：`async def`或使用generator实现。
  基于generator的协程应该使用`@asyncio.coroutine`修饰，但这并不是严格要求的，基于generator的协程使用`yield from`语法。<!--more-->

- `coroutine`这个词有两种意思：
  1. 定义协程的**函数**
     使用`async def`或`@asyncio.coroutine`。无歧义的称呼是`coroutine function`,`isroutinefunction()`返回`True`。
  2. 调用协程函数获得的**对象**
     这个对象表示一个可结束的**计算**或**I/O操作**。无歧义的称呼是`coroutine object`,`iscoroutine()`返回`True`。

- 协程可以做的事：

  - `result = await future`或`result = yield from future`
    暂停协程，直到future完成，然后返回`future`的结果，或者抛出一个异常。

  - `result = await coroutine` 或 `result = yield from coroutine` 
    等待另一个协程产生结果(或者抛出一个异常)。`coroutine`表达式必须为对另一个协程的`调用`。

  - `return expersssion` 产生结果，结果传递给那些使用`await`或`yield from`等待此协程的协程。

  - `raise exception` 抛出异常。

## asyncio

- `asyncio.get_event_loop`

- `asyncio.ensure_future(coro_or_future, *, loop=None)`
  将coroutine object包装在future中，返回`Task`对象。

- `BaseEventLoop.run_until_complete(future)`
  阻塞，直到future完成，返回future的结果。
  如果参数是一个`coroutine object`，则使用`ensure_future()`包装一下。

## 本质

1. 添加任务到事件循环中

2. 每一次循环，从队列中取出一个任务，对其调用send，让其继续执行，将send的结果继续添加到任务队列中。如果调用send抛出了`StopIteration`异常，则表示任务已经完成了，则不再添加到队列中。

3. `await future`，future 抛出的`StopIteration`由await捕获，并从`StopIteration`

   对象的第一参数中获取`future`的结果，作为await的返回值，此时被等待的`future`已经完成，继续执行当前coroutine之后的代码。

   如果future没有抛出异常，则future中`yield`的结果经由这个await(或者yield from)向上传递，一直到调用`send`的那个语句中(一般是事件循环中)，作为send的返回值。

   所以一直都是最深层次的任务和事件循环在打交道，与途中的await没有关系，只有当最底层的future完成之后，才开始执行其上一层的任务。

## 疑问

1. 第一个`send(None)`或者`next(coro)`的返回值是什么？

   第一次 send 使 generator 开始执行，直到遇到第一个 yield

   此时 generator 让出执行权，将 yield 表达式的值作为 send 的返回值，此时 generator 内部 yield 左边的赋值并没有进行，而是在这之间就 yield(让步)了。

   第二次调用 send(something), 则从 yield 赋值语句开始恢复，send 的参数作为 yield 的返回值，赋给左边的变量，继续执行。

2. yield 与 yield from 有什么区别？

   函数中的 yield 使函数成为 generator，当 generator 到达末尾时(自然结束或遇到 return 语句)，会抛出`StopIteration`异常。

   Python3.3 之后，return 语句的参数会成为`StopIteration`的 value 参数(`raise StopIteration(value=something)`)。

   普通的由 yield 创造的 generator 一般会忽略 StopIteraion.value，也就是忽略reurn的值，只将StopIteration作为generator exhausted的条件。而`yield from` 会捕获`StopIteraion` 异常，并获取其 value 参数，作为自己的返回值。看上去没什么大不了，但却是后来`await`的基础。

3. ？？

   如果yield from future， future 没有 return，那么 future 中 yield 产生的值都会**经过** yield from 向更高层传递，最终到达事件循环，与 yield from 一点关系也没有。当 futurn return时，yield from 很开心，终于不用做传话人了，于是获取 futurn 的结果，作为自己的返回值，也就意味着这个future resolve了，这个异步任务完成了。

   本来使用yield可以产生任意类型的值，yield from都会将其逐步传递到最顶层作为 send 的返回值。但是因为最顶层一般是事件循环，它需要将 send 的结果，继续添加到任务队列中，之后再从队列中将其取出，对其调用 send 方法(或next)让其继续执行。所以这个对象就必须有一些特殊要求，这个要求就是awaitable???? Future? Task?

4. await与yield from的区别？

   await比yield from只多了一步参数检查：await只接受**awaitable**对象，包括：

   - 一个 *native coroutine object* returned from *native coroutine function*
   - *generator-based coroutine object* returned from a functin decoreated with `asyncio.coroutine()`
   - an object with an \_\_await\_\_  method returning an iterator.

   任何 yield from 链都是以 yield 结尾的，这是 Future 实现的基础。每个await 都是被 await链底端的 yield 暂停的。

## async 与 await 语法

- async def 函数永远是协程，即使它不包含await表达式
- async 函数中使用 yield 或 yield from 是语法错误
- async def 定义的是 native coroutine，设置了 `CO_COROUTINE`标志
- *generator-based coroutine* 与 native coroutine 兼容，设置了 `CO_ITERABLE_COROUTINE` 标志
- 在async def 函数外使用 await 是语法错误
- 为 await 传递非awaitable对象会报 TypeError

## 参考链接

- [](https://snarky.ca/how-the-heck-does-async-await-work-in-python-3-5/)
- [asyncio — Asynchronous I/O, event loop, coroutines and tasks](https://docs.python.org/3/library/asyncio.htmll)
- [PEP 492 -- Coroutines with async and await syntax](https://www.python.org/dev/peps/pep-0492/#examples-of-await-expressions)
- [PEP 3156 -- Asynchronous IO Support Rebooted: the "asyncio" Module](https://www.python.org/dev/peps/pep-3156/)