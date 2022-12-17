---
title: 'HTTP for Human: 深入研究 Requests'
slug: requests-http-for-human
tags: [Python, requests]
categories: [Coding]
---

这是一篇学习 Requests 的随笔.

### Python 回顾

- `bytearray`是一个字节数组, 数组的内容是可变的, 数组中的每一个元素都是 `[0, 256)`之间的整数
  定义为: `bytearray([source[, encoding[, errors]]])` <!-- more -->
  其中`source`参数有几种情况:
  - If it is a string, you must also give the encoding (and optionally, errors) parameters; `bytearray()` then converts the string to bytes using `str.encode()`.
  - If it is an integer, the array will have that size and will be initialized with `null` bytes.
  - If it is an object conforming to the *buffer* interface, a read-only buffer of the object will be used to initialize the bytes array.
  - If it is an iterable, it must be an iterable of integers in the range 0 <= x < 256, which are used as the initial contents of the array.

  `bytearray`是一个`Mutable Sequence`, 所以支持大多数`list`的操作:
  ![list-operation](/images/list-operation.png)

  (**注意**: 图片放在`source/images`中, 会被拷贝到`public/images`中, 在Markdown中绝对引用 `/images/xxx.jpg`才能显示图片)

- `bytes`是一个不可变版本的`bytearray`
  `bytes`对象可以通过 字面量(literal) 直接创建: `b''`或者`B''`
### 开始

我们最常使用的`requests.get()`, `requests.post()` 等方法都是简便的 API, 传递给这些 API 的参数被用来构造`Request`对象.

`Request` 对象有两种形态, 一种是普通的对象形态, 一种是经过 `prepare` 之后转换为 `PreparedRequest`, 才是真正可以被发送的对象, 其中的`body`, `header`, `cookie` 等都已经转换成符合HTTP规范的字符串了, 准备好在网络上传输了.

`PreparedRequest`对象只是将要被发送的内容, 而自身没有发送的功能. 发送的功能是由`session`对象提供的, `session.send()`方法会调用`adapter.send()`, 将请求真正地发送出去.

请求被发送之后收到服务器的响应, 响应被`adapter`解析为`Response`对象, 从`requests.get()`, `requests.post()`等方法中返回. 调用者可以用`Response`中获取需要的信息.

### 常见用法

1. 普通青年的用法

   ```python
   resp = requests.get('http://www.baidu.com')
   print(resp.content)
   ```

2. 作死青年的用法

   ```python
   s = requests.Session()
   s.headers.update({'laozizuidiao': 'laozizuidiao'})
   s.get('http://www.gov.cn')
   s.get('http://www.beijing.gov.cn')
   ```
3. 文艺青年的用法
   ```python
     s = Session()
     req = Request('GET', url,
        data=data,
        headers=header
     )
     prepped = req.prepare()
     //prepped = s.prepare_request(req) 这样可以把 session 中的已有的状态添加到这个 req 中
    resp = s.send(prepped,
        stream=stream,
        verify=verify,
        proxies=proxies,
        cert=cert,
        timeout=timeout
    )

    print(resp.content)
   ```

下面我们逐个分析 Requests 的组成部分.

### Request

`Request`其实是一个没什么用的类, 我还不怎么明白为什么要设计这个类.

`Request`继承自`RequestHookMixin`, 自身只有一个方法`prepare()`, 作用就是新建一个`PreparedRequest`对象, 然后将自己的那些参数传给`PreparedReques.prepare()`, 准备好一个可以发送的对象

我们直接使用的API, 都是先用我们传递的参数构造这个对象.

下面我们来分析一下常用的几个参数: TODO

`params`

`data`

`headers`

`cookies`

`allow_redirects`

`auth`

`json`

`stream`



### PreparedRequest

`PreparedRequest`是请求过程中最重要的一个类, 而这个类中最重要的就是它的各个`prepare_xxx`方法.

TODO

### Response

❓`Response` 实际是由`adapter`生成的, `adapter`是什么?

下面分析一下`Response`常用的属性:

`.status_code` 服务器响应的状态码

`.reason` 原因短语

`.url` TODO

`.encoding` 

`.headers` 是一个`CaseInsensitiveDict`

`.cookies`初始化为`cookiejar_from_dict({})`, 这个函数的定义为:

```py
def cookiejar_from_dict(cookie_dict, cookiejar=None, overwrite=True):
    """Returns a CookieJar from a key/value dictionary.

    :param cookie_dict: Dict of key/values to insert into CookieJar.
    :param cookiejar: (optional) A cookiejar to add the cookies to.
    :param overwrite: (optional) If False, will not replace cookies
        already in the jar with new ones.
    """
```

也就是说将一个`dict`添加到一个`cookiejar`中, 如果`cookiejar`为`None`, 则新建一个空的`RequestCookieJar`

`.request`是请求时的`PreparedRequest`对象

`.elapsed` TODO

`.content` 是没有 `decode`过的**字节串**, 而`.text` 会自动根据`.encoding` 中的编码自动将`content`解码为`str`. 如果手动改变`.encoding`, 那么重新读取`.text`时也会重新解码

默认情况, 请求之后响应体会立即被下载. 如果设置 `stream=True` , 则直到访问 `Response.content`时才会下载
但是设置`stream=True`, Requests 无法自动将连接释放到连接池中, 除非消耗了所有数据或者显式地调用`Response.close()`

`.history`是一个`list` TODO

`Reponse`的核心就是`.raw`, `iter_content()`依次从`raw`中`read`数据, 而`iter_lines`, `.content` , `__iter__()`都直接依赖于`iter_content()`, `.text`依赖于`.content`, `.json()`依赖于`.text`

### Session

重头戏, 好好说说. TODO

在方法级使用的参数(比如`cookies`)的优先级高于会话级, 但是在方法内使用的参数不会在之后的请求中保留.

### Cookie

Requests 中默认使用的`RequestCookieJar` 继承自 `cookielib.CookieJar`, 而 Python3 中 `cookielib`已被移到`http.cookiejar`中.

为了适配`http.cookiejar.CookieJar`的接口, `requests.cookies`又使用了`MockRequest`, `MockResponse`, 而`http.cookiejar`的代码相当复杂, 实在是搞不懂.

`RequestCookieJar`为`http.cookiejar.CookieJar`提供了`dict`的访问接口. Requests 的所有代码都可以无障碍地使用`cookielib.CookieJar`的子类, 比如 `LWPCookieJar`, `FileCookieJar`.

**CookiePolicy**

该类的主要功能是收发cookie, 即确保正确的cookie发往对应的域名, 反之一样.

**DefaultCookiePolicy**

实现了`CookiePolicy`接口.

**Cookie**

可以看成一条 cookie 数据.
有好几种版本的 cookie 吗? Netscape 和 RFC2965 ? 相应的头部分别是 Set-Cookie 和 Set-Cookie2

**CookieJar**

很多条 cookie 数据的集合, 是我们主要的操作对象, 里面有一系列的方法支持更细致的操作. Requests 的 cookie 主要依赖于此类.

**FileCookieJar**

该类继承自`CookieJar`，`CookieJar`只是在内存中完成自己的生命周期，`FileCookieJar`的子类能够实现数据持久化，定义了`save`、`load`、`revert`三个接口

**LWPCookieJar**

`Response.cookies` 默认是`RequestCookieJar`, 因为一般不会手动构造`Response`, 都是由`HTTPAdapter.build_response()`方法构建的, 所以`Response.cookies`基本永远都是`RequestCookieJar`, 用户可以方便的使用. 而`Session.cookies` 可以由用户随意替换, 使用`LWPCookieJar`等已经实现了`load()`,`save()`等方法的 cookie 比较方便.

**MozillaCookieJar**



### Auth

各个API都有`auth`参数, 这个参数可以是一个`(user, pass)` 这样的 2-tuple, 也可以是一个 callable.

这个 callable 接受 `PreparedRequest`对象作为参数, 可以在发送之前修改请求(比如说添加 Authorization 头部). `requests.auth`中提供了两个常用的验证方法:`HTTPBasicAuth`和`HTTPDigestAuth`, 其他自定义的身份验证机制一般继承自`requests.auth.AuthBase` 类, 这个类虽然什么也没做, 但是语义化比较好.

### Proxy

我自己还没明白怎么用.

### Adapter

非常重要的基础类, 使用`urllib3`提供发送功能, 很多概念不太懂, 日后再说. :smile:

Python3 中 urllib 和 urllib2 合并为 urllib, urllib3是一个第三方库, 是对`httplib`的包装, 发请求的时候，拼凑http请求, 收到回复的时候,解析response,timeout 也是用httplib 库的timeout参数, requests 内部集成了 urllib3.

![urllib3实现](/images/python_lib_urllib3_implements.png)

{% plantuml %}
title hello
"requests.get()" -> "requests.request()":
"requests.request()"-> "Session.request()":
"Session.request()" -> "Session.send()":
"Session.send()" -> "HTTPAdapter.send()":
"HTTPAdapter.send()" -> "HTTPAdapter.build_response()":HTTPResponse
{% endplantuml%}

`Session.send()`调用`HTTPAdapter.send()`, `HTTPAdapter.send()`从`urllib3`中获取一个`connection`, 然后调用其`urlopen`或者`HTTPResponse.fromhttplib`得到一个`HTTPResponse`对象. 然后调用`Adapter.build_response()`, 从`HTTPResponse`中取出各种数据构造`Requests.Response`, 将`Requests.Response.raw`设为这个`HTTPResponse`对象, 从`HTTPResponse`中取出`cookie`添加到`Response.cookie`中. 然后`Session.send()`再一次从`HTTPRespnse`中获取`cookie`, 添加到自己的`Session.cookie`中.


### 编码问题

[issue #2266](https://github.com/kennethreitz/requests/issues/2266) 中说, 会将 `util.get_encodings_from_content`这样的方法移到`requests-toolbelt`中, 使得 requests 更专注于 HTTP 而不是 HTML.

### 参考链接

- [requests源码浅析](http://www.jianshu.com/p/0c0502dfecea)
- [Requests: HTTP for Humans](http://docs.python-requests.org/en/master/)
- [python中urllib, urllib2,urllib3, httplib,httplib2, request的区别](https://my.oschina.net/sukai/blog/611451)
- [urllib3](http://xwl-note.readthedocs.io/en/latest/software/python-lib/urllib3.html)
- [requests 如何自动识别编码](http://www.jianshu.com/p/b0e386361570 )
- [python中的cookielib的使用方法](https://blog.phpgao.com/python-cookielib.html)