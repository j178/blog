---
title: PHP 使用 xdebug 调试之路
tags:
- PHP
- xdebug
categories: [Coding]
date: 2017-01-21 00:15:18
---


- PHPStorm中调试利用的是xdebug的remote特性，使用的是 `DBGp`协议，PHPStorm是一个实现了`DBGp`协议的客户端
- 在`php.ini`中启用`xdebug.remote_enable`, 配置`xdebug.remote_host`和`xdebug.remote_port`, port默认是9000.
  通过`phpinfo()`函数的输出，可以在`Loaded Configuration File`中看到实际加载的`php.ini`文件是哪一个。
- `sudo apt install php-xdebug`之后，会自动在`/etc/php/7.0/apache2/conf.d`中创建一个`xdebug`的配置文件，可以在这个文件中修改关于xdebug的配置，而不用去改通用的`php.ini`.
- client启动之后会监听 port，等待xdebug发送的请求。
- 要使php解释器在运行的时候activate xdebug，有三种方式：
  1. 从命令行运行php脚本时，设置环境变量`export XDEBUG_CONFIG="idekey=session_name"`, 同样也可以在`XDEBUG_CONFIG`环境变量中设置`remote_host`,`remote_port`等属性。
  2. 从一个浏览器中启动debugger，需要传递一个`XDEBUG_SESSION_START=session_name`参数，这个参数可以是`GET`，`POST`或者`Cookie`。可以使用一些浏览器的插件自动传递这个参数。
- 只要`xdebug.remtoe_enable`开启了，PHPStorm打开了 Listening for PHP Debug，也就是服务器和客户端都准备好了，然后浏览器发送请求的时候包含一个`XDEBUG_SESSION_START`参数（？文档中写的是XDEBUG_SESSION_START,而实际上`bookmark`和插件中设置的cookie都是`XDEBUG_SESSION`），就可以愉快地调试了。