---
title: Linux 中的 runlevel
slug: linux-runlevel
date: 2016-12-02 04:27:13
tags: [linux]
categories: [Linux] 
---

6个典型的运行级别：

- 0 停机，关机
- 1 单用户，无网络连接，不运行守护进程，不允许非超级用户登录
- 2 多用户，无网络连接，不运行守护进程
- 3 多用户，正常启动系统
- 4 用户自定义
- 5 多用户，带图形界面
- 6 重启<!--more-->



现在的Linux好像都用 systemd 中target来代替runlevel了，但仍然兼容runlevel，并且会将runlevel映射到相应的target中，比如：0        

| Runlevel |      Target      |
| :------: | :--------------: |
|    0     | poweroff.target  |
|    1     |  rescue.target   |
| 2, 3, 4  | multi-usr.target |
|    5     | graphical.target |
|    6     |  reboot.target   |


/etc/rc0.d/ /etc/rc1.d 这些目录对应不同的运行级别，系统启动时运行相应的目录中的脚本来启动服务。这些目录中都是指向/etc/init.d/目录中脚本的软连接。

/etc/init.d 中的脚本，一般称为服务(service)?

/etc/rcX.d/中的链接，系统会杀掉K开头的服务，按照数字从小到大启动S开头的服务。

/etc/rc.local存在于多个runlevel中，所以基本上只要机器开机这个脚本就会自动运行。一般情况下这个脚本为空，如有需要将一些程序加入开机自启，可以讲程序命令添加到这个脚本中。

查看运行级别： runlevel

运行级别是互斥的，无法同时运行多个运行级别。

本来应该由init进程读取/etc/inittab确定运行级别，但现在还有都没有这个文件了。

### upstart机制

/etc/init/是upstart寻找作业配置文件的地方

### systemd机制

service \<script\> COMMAND [options] 实际上是运行一个sysvinit程序或者upstart作业，比如service nginx start实际执行的命令是 /etc/init.d/nginx start，start命令实际是由nginx这个脚本来做的。



### 参考链接

1. http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html
2. http://www.ruanyifeng.com/blog/2013/08/linux_boot_process.html
