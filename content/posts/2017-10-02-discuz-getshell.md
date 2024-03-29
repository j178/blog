---
title: Discuz X 3.4 由任意文件删除到 Getshell
slug: discuz-getshell
date: 2017-10-02 00:14:34
tags: [Hack]
---

2017年9月29日，Discuz! 修复了一个安全问题用于加强安全性，这个漏洞会导致前台用户可以导致任意删除文件漏洞。并且，配合其他漏洞可以实现 GetShell。

## 初步分析

先下载官方托管在码云上的 Discuz X 源码：

```sh
git clone https://gitee.com/ComsenzDiscuz/DiscuzX.git
```

查看其提交历史，发现有一个『优化 加强安全性』的 commit 比较可疑：
![优化加强安全性](/img/discuz-getshell-1.jpg)

查看其具体内容：
![修复](/img/discuz-getshell-2.jpg)

它删除了5个 unlink 语句……

这种修复方式真是简单粗暴啊… 看来我们分析的入口就在这里了。

## 任意文件删除

这部分的分析请参考这里：<https://paper.seebug.org/411/>

(别吐槽——这篇文章分析得很好，省了我很多字 :haha:)

## 文件删除能做什么

通过上面那篇文章，想必大家都能复现文件删除的漏洞了。那么这个漏洞能做什么呢？难道只是单纯的破坏吗？

我们知道， Discuz X 这类 CMS 软件，都会一个 install 目录，用来处理用户初次安装时的配置和数据库初始化。在安装完成后，会在 data 目录下生成一个 install.lock 文件。如果尝试再次访问 /install 来安装，检测到存在 install.lock，于是提示错误：
![](/img/discuz-getshell-3.jpg)

如果用户没有删除 install 目录，而且我们也可以利用文件删除漏洞的话，我们就可以删掉这个 instal.lock，然后重新安装 Discuz。而重装过程中一般都有写配置文件的步骤，可能会给我们写入一句话的机会。

## 分析重装过程

`install/index.php` 文件控制安装的过程，它的逻辑其实很简单。安装主要分为五个步骤：
![](/img/discuz-getshell-4.jpg)

1. show_license 显示是否接受协议，同意后跳到 env_check
2. env_check 检查目录是否可写和 PHP的相关检测，然后跳到 app_reg
3. app_reg 提示是否安装 UCenter Server，POST 到  app_reg，如果 install_ucenter=yes，直接重定向至 db_init，否则显示配置 ucenter 的表单
4. db_init 要求填写数据库和网站信息，POST 到 db_init，这时会进行数据库连接和数据表的检查。所以填写的数据库地址等信息要求必须是目标站能够连接的真实的服务器。
5. ….

在 db_init 的处理逻辑里，我们看到了想要的：
![](/img/discuz-getshell-7.jpg)

跟进 save_config_file 函数：
![](/img/discuz-getshell-10.jpg)

继续跟进 getvars 函数：
![](/img/discuz-getshell-11.jpg)

结果却让人失望，这里的 addcslashes 会转义我们的引号，导致一句话无法逃逸出引号，无法执行。

但是紧跟着的 install_uc_server 让我很开心：
![](/img/discuz-getshell-12.jpg)

跟进 install_uc_server 发现它没有对传入的参数做任何过滤，传入了 save_uc_config 中:
![](/img/discuz-getshell-13.jpg)
同样 save_uc_config 也没有做任何处理，直接拼接到配置文件字符串中，然后写入了文件。

因为 dbhost, dbuser 等参数需要用来连接数据库，所以我们能利用的就只有 tablepre 了，也就是我们指定的数据库前缀。我们构造这样的 tablepre:

```php
x');@eval($_POST[pwd]);('
```

实验一下：
![](/img/discuz-getshell-14.jpg)

然后查看 config/uc_config.php 文件，发现我们的一句话已经写入了：
![](/img/discuz-getshell-15.jpg)

使用菜刀连接：
![](/img/discuz-getshell-17.jpg)

## EXP

最后，附上整个利用过程的 EXP：
<https://gist.github.com/j178/67f4dbd8e87cd012a7caa8752ea06e7b>

（**此漏洞危害巨大，请勿在网络上测试**）
