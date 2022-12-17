---
title: Composer 初试
slug: php-composer
tags:
  - PHP
  - Composer
categories: [Coding]
date: 2017-01-21 00:15:22
---


- Composer 不是一个包管理器，默认下它不会全局安装任何包，只是在一个项目的某个目录中进行安装，它只是一个**依赖管理**工具。
- Composer 解决的问题是：
  1. 你有一个项目依赖于若干个库
  2. 其中一些库依赖于其他库
  3. 你声明你依赖的东西
  4. Composer 会找出哪个版本的包需要安装，并安装他们（将他们下载到你的项目中）
### 安装

1. curl -sS  https://getcomposer.org/installer | php 

   installer是一个PHP脚本，用来下载真正的composer.phar，composer.phar会保存到当前目录中。可以使用 `--install-dir`指定保存目录。

2. 将composer.phar移动到`PATH`中
   `mv composer.phar /usr/local/bin/composer`
   这样就可以直接使用`composer`命令

### 使用

- composer help <command> 非常有用
- `composer install` 读取当前目录下的`composer.lock`文件，下载和安装其中提到的库和依赖。如果`composer.lock`不存在，则查看`composer.json`文件。
- `composer update` 读取当前目录中的`composer.json`文件，更新、删除或者安装所有的依赖。
- `composer init`
- `composer require` xx 添加xx依赖到`composer.json`中，并安装他们
- 修改composer 全局配置`composer config -g repo.packagist composer https://packagist.phpcomposer.com`

### 链接

[Composer中文文档](http://docs.phpcomposer.com/00-intro.html)

[Packagist/Composer中国全量镜像](http://pkg.phpcomposer.com/)