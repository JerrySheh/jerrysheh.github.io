---
title: MySQL运维
categories: 
- 数据库
- 高性能MySQL
tags: database
abbrlink: 9faffbfd
date: 2018-03-22 21:00:00
---


# Ubuntu 18.04 安装 Mysql 5.7

注：如果要安装 Mysql 8.0 ，可以参考 [官方文档](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/)

<!-- more -->

## 卸载和安装

干净卸载后安装

```
sudo apt-get --purge remove mysql-server mysql-common mysql-client
sudo apt-get install mysql-server
```

设置 root 密码

```
mysqladmin -u root password your-new-password
sudo /etc/init.d/mysql restart
```

或者，直接运行安全脚本配置

```
sudo mysql_secure_installation
```

## 安装完毕后发现没有登录权限

```
ERROR 1045: Access denied for user: 'root@localhost' (Using
password: YES)
```

首先无密码登录

```
sudo mysql -u root
```


然后查看当前用户，删除后重新创建 ROOT 账户，并授权

```sql
mysql> SELECT User,Host FROM mysql.user;

mysql> DROP USER 'root'@'localhost';

mysql> CREATE USER 'root'@'%' IDENTIFIED BY '123456';

mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

mysql> FLUSH PRIVILEGES;
```

## 允许远程访问

```sql
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

mysql> FLUSH PRIVILEGES;
```

打开 /etc/mysql/mysql.conf.d/mysqld.cnf 文件

```
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

把 bind-address 改为 0.0.0.0

```
#bind-address = 127.0.0.1
bind-address = 0.0.0.0
```

---

# 修改ROOT密码

## 方法1： 用SET PASSWORD命令

```
MySQL -u root
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
```

## 方法2：用mysqladmin

```
mysqladmin -u root password "newpass"
```

如果root已经设置过密码，采用如下方法

```
mysqladmin -u root password oldpass "newpass"
```

---

# 如何导入数据库文件

假设我们已经有了一个数据库脚本， projectDB.sql，如何导入到MySQL？

方法一：直接通过 mysql 命令运行

```
mysql -u root -p123456 --port 3306 < /home/jerrysheh/projectDB.sql
```

方法二：登录 mysql 后使用 source 命令

```
mysql>create database <database-name>;
mysql>use <database-name>;
mysql>source /home/jerrysheh/projectDB.sql;
```

## 解决导入慢的问题

当数据库非常大（>100M）时，导入时间可能要花费几小时，优化方式如下：

假设要讲A数据库的数据导入到B数据库

1、首先确定目标库(B)的参数值，登录数据库B，执行以下命令

```
mysql>show variables like 'max_allowed_packet';
mysql>show variables like 'net_buffer_length';
```

2、根据参数值，在A数据库中使用 mysqldump 命令，如：

```
C:\Program Files\MySQL\MySQL Server 8.0\bin>mysqldump.exe -uroot -p mall  -e --max_allowed_packet=67108864 --net_buffer_length=16384 > mall.sql
```

各个参数的意思：

- **-e**： 使用包括几个VALUES列表的多行INSERT语法;
- **--max_allowed_packet=XXX**：客户端/服务器之间通信的缓存区的最大大小;
- **--net_buffer_length=XXX**：TCP/IP和套接字通信缓冲区大小,创建长度达net_buffer_length的行。

注意：max_allowed_packet和net_buffer_length不能比目标数据库的设定数值大，否则可能出错。

3、登录数据库B，执行 source 命令导入

---

# 修改默认字符集

```SQL
ALTER DATABASE db_name CHARACTER SET utf8mb4;
ALTER TABLE db_name DEFAULT CHARACTER SET utf8mb4;
ALTER TABLE db_name CONVERT TO CHARACTER SET utf8mb4;
```
