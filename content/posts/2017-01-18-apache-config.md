---
 title: Apache 配置学习
 date: 2017-01-18 23:55:43
 tags:  [Apache]
 categories: [Backend]
---

## 基本

- 配置文件`/etc/apach2/apache2.conf`以及`/etc/apache2/conf-enabled`
- `DirectoryIndex index.html index.php` 优先级从左到右降低
- 包含外部配置文件 `Include xxx`

## 启动

`apachectl`脚本，从`/etc/apache2/envavars`中读取并设置一些环境变量



## Rewrite

基于正则表达式的规则，动态修改 incoming URl requests。

## Virtual Host

用同一个 Apache 服务器上搭建多个服务，根据用户请求中不同的 Host首部（或其他条件） 将请求转给不同的`VirtualHost`来处理。

### Name-based vs IP-based Virtual Hosts

基于IP的虚拟主机，要求服务器有多个IP，Apache根据用户请求的IP来转发到相应的主机中
基于名字的虚拟主机，要求客户端在HTTP首部中包含主机的名字，这样多个主机就可以共享同一个IP。
基于名字(hostname)的虚拟主机，通常只需要将多个域名解析到同一个IP即可。
比如说，login.a.com和logout.a.com只想相同的IP，也使用相同的端口（即后台的web服务器是同一个），但是后台的web服务器却可以根据请求中的Host首部不同，为他们提供不同的服务。

### 服务器如何选择合适的 name-based virtual host?

先进行 IP-based的选择，也就是先根据请求的IP选择出为此IP服务的Virtual Host，然后再从中找出最符合的name-based virtual host。

先根据IP和端口选出一组最符合的hosts，然后根据`ServerName`和`ServerAlias`作比较。如果没有符合的ServerName和ServerAlias，则选择列出的第一个。

省略ServerName则默认使用fully qualified domain name。





# 参考链接

[](https://httpd.apache.org/docs/2.4/vhosts/name-based.html)