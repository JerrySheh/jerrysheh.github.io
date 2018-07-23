---
title: Debian 9 安装配置 Apache2
comments: true
categories: 技术&技巧
abbrlink: d3952d64
date: 2018-07-11 15:15:47
tags:
---

![apache2](https://www.apache.org/img/asf_logo.png)

# 安装

```shell
sudo apt-get install apache2
```

然后访问 127.0.0.1 或者该服务器的公网ip ，即可看到 apache2 的主页面

![welcome](../../../../images/Linux/Apache2Welcome.png)


<!--more-->

---

# 修改主配置文件

```shell
vim /etc/apache2/apache2.conf
```

在主配置文件里，修改以下内容为文件夹赋予打开的权限

```xml
<Directory />
 Options FollowSymLinks
 AllowOverride None
 Require all denied
</Directory>
<Directory /usr/share>
 AllowOverride None
 Require all granted
</Directory>
<Directory /var/www/>
 Options Indexes FollowSymLinks
 AllowOverride None
 Require all granted
</Directory>
```

---

# 修改其他配置文件

```
vim /etc/apache2/sites-available/000-default.conf
```

DocumentRoot 对应的值就是默认的目录了，可以任由我们修改。

修改完配置文件后记得重启服务

```
sudo service apache2 restart
```

---

# 启动和关闭 apache2 服务

启动和关闭apache2服务可以通过执行命令

```
sudo /etc/init.d/apache2 start（stop / restart）
```

或者

```
sudo service apache2 start (stop / restart)
```

这里一定要注意记得！记得！加root权限！这里不加root权限并没有明显的提示（好坑），当遇到问题的时候很难让人想到是这里出的错，所以一定要记得！记得！加root权限！

---

# 其他

- 当访问本机的时候，默认进入的页面是/var/www/html/index.html。
- 配置系统的说明在/usr/share/doc/apache2/README.Debian.gz中。
- 完整使用手册可以通过安装apache2-doc 进行下载。
- 主配置文件为/etc/apache2/apache2.conf。
- 默认情况下apache2拒绝访问除/var/www 和/usr/share文件夹外的其他文件，这种权限是通过apache2.conf文件来控制的.
