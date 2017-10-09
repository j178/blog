---
title: php-mini-app
tags:
---

1. 需要启用 rewrite模块：
   `sudo a2enmod rewirte`
   这条命令会在 `/etc/apache2/mod_enabled`中创建一个软连接，指向 `/etc/apache2/mod_available/rewrite`

2. 允许使用.htaccess

   `/var/www/html/mini/.htaccess` 中使用了rewrite，所以的访问都会rewrite到 `/var/www/html/mini/public`目录中，为了使 `.htaccess`生效，需要修改一点设置：

   编辑 `/etc/apache2/sites-available/000-default.conf`， 在 `DocumentRoot` 后添加：

   ```
   <Directory "/var/www/html">
   	AllowOverride All
   </Direcotry>
   ```
   因为这个`sites-enabled/000-default.conf`也指向这个，所以更改`avaiable`目录下的文件也可以使配置生效。

   `AllowOverride`选项在 apache 2.3.9 之后默认是 `None`, 服务器甚至不会尝试读取 `.htaccess`文件。`AllowOverride`只能用在`Directory` section中。

3. 重启apache

   `sudo systemctl restart apache2`