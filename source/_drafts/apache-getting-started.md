---
title: apache-getting-started
tags:
---

# Apache 初步认识

- 配置文件在 `/etc/apache2/`中，主要分为三部分的配置：`conf`, `site`, `mod`
- 三部分的配置均分为`available`和`enabled`，其中`enabled`是指向`available`的软连接
- 网站文件不一定都要放在 `/var/www/html`中，可以在一个配置文件中使用`Alias`指令。`phpmyadmin`就是这么做的，它的实际文件在`/usr/share/phpmyadmin`中。
- ​

php的配置结构？

phpmyadmin的配置结构？

- 对于 `DocumentRoot`之外的目录：
  1. 使用软连接，但是出于安全考虑，只有在`Options`设置了`FollowSymLinks`时才有效
  2. 使用`Alais`指令，它会将文件系统中的任何部分映射到web space中。
     比如 `Alias "/docs" “/var/web"`,那么 `http://www.a.com/docs/dir/file.html`实际请求的是`/var/web/dir/file.html`文件