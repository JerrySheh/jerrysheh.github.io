---
title: Debian 9 安装 Apche2 和 vsftpd
comments: true
categories: Linux
abbrlink: d3952d64
date: 2018-07-11 15:15:47
tags:
---

![apache2](https://www.apache.org/img/asf_logo.png)

# 什么是 Apache HTTP Server

Apache HTTP Server Project 是致力于为现代操作系统（包括UNIX 和 Windows）提供和维护的一个开源 HTTP 服务项目。该项目发起于 1995 年，至今已有20+年的历史。

借助 Apache HTTP Server，我们可以在我们的计算机或服务器上快速部署一个高效、可用的 HTTP 服务。Apache2 是 Apache HTTP Server 的最新版本。

- Apache官网：[Link](http://httpd.apache.org/)

<!--more-->

# Getting Started

## 安装

```shell
sudo apt-get install apache2
```

访问 127.0.0.1 或者该服务器的公网ip ，即可看到 apache2 的主页面

![welcome](../../../../images/Linux/Apache2Welcome.png)




## 修改主配置文件

```shell
vim /etc/apache2/apache2.conf
```

在主配置文件里，修改以下内容为文件夹赋予打开的权限

```xml
# 拒绝访问 / 目录
<Directory />
 Options FollowSymLinks
 AllowOverride None
 Require all denied
</Directory>

# 不用改或注释掉
<Directory /usr/share>
 AllowOverride None
 Require all granted
</Directory>

# 这里可以修改成自己想更改的目录
<Directory /var/www/>
 Options Indexes FollowSymLinks
 AllowOverride None
 Require all granted
</Directory>
```

## 修改其他配置文件

### 默认目录

```
vim /etc/apache2/sites-available/000-default.conf
```

DocumentRoot 对应的值就是默认的目录了，可以任由我们修改。


### 修改端口号

```
vim /etc/apache2/sites-available/000-default.conf
```

第一行 virtualport 记得改（默认80）

```
vim /etc/apache2/ports.conf
```

listen 端口号（默认80）

修改完配置文件后重启服务

```
sudo service apache2 restart
```

## 启动和关闭 apache2 服务

启动和关闭apache2服务可以通过执行命令

```
sudo /etc/init.d/apache2 start（stop / restart）
```

或者

```
sudo service apache2 start (stop / restart)
```

- **注意**：这里一定要注意记得！不加root权限可能没有明显的提示。但实际上并没有启动成功。

## 其他配置

- 当访问本机的时候，默认进入的页面是/var/www/html/index.html。
- 配置系统的说明在/usr/share/doc/apache2/README.Debian.gz中。
- 完整使用手册可以通过安装apache2-doc 进行下载。
- 主配置文件为/etc/apache2/apache2.conf。
- 默认情况下apache2拒绝访问除/var/www 和/usr/share文件夹外的其他文件，这种权限是通过apache2.conf文件来控制的.

---

![vsftpd](../../../../images/Linux/vsftpd.jpg)

# 什么是 vsftpd

vsftpd 是 UNIX（包括linux）系统下的一个 FTP 服务。借助 vsftpd，我们可以在 UNIX 服务器上快速部署一个安全、快速、稳定的 FTP 服务。


# Getting Started

## 安装

```
sudo apt-get update
sudo apt-get install vsftpd
```


## 配置

```
sudo vim /etc/vsftpd.conf
```

在配置文件里几点注意:

```bash

# listen 和 listen_ipv6 开一个就行；两个都开，vsftpd就报错了

listen=YES
# listen_ipv6=YES


# 本地用户访问
local_enable=YES

# 可写
write_enable=YES


# 不允许所有用户访问用户主目录之外的目录
chroot_local_user=NO

```

## 启动

```
# 启动
sudo service vsftpd start

# 查看状态
sudo service vsftpd status
```

## 客户端连接

![](https://www.filezilla.cn/wp-content/uploads/2015/03/logo-300x138.png)

客户端连接推荐使用 FileZilla

- 官网下载地址：https://filezilla-project.org/

### 使用方法

在站点管理器里面简单配置连接即可：

![filezilla](../../../../images/Linux/filezilla.png)

- **提示**：用户名和密码是 linux 系统的用户名和密码
