---
title: Ubuntu的一些奇技淫巧
date: 2017-09-12 20:14:40
tags: linux
---

接触linux越久，掉进莫名其妙的坑里就越多，于是我决定每遇到一个坑就记录下来，这样以后再踩的时候不至于爬不起来。

Ubuntu的一些奇技淫巧

目前 get 的有：
1. 调整鼠标速度
2. 解决双系统时间不同步的问题
3. 系统更新提示 /boot 空间不足的解决办法
4. 更改国内源，提高下载速度
5. Xshell5 Ubuntu系统的VPS服务器跟本地电脑互传文件

<!-- more -->

# 一. 调整鼠标速度

```
xset m N
```

* 其中 N 是速度速度值，0（最慢）- 10（最快）

---
# 二. 解决 win10 + Ubuntu 双系统 时间不同步的问题

## 1. 先在 Ubuntu 下更新时间，确保时间无误

```
sudo apt-get install ntpdate
sudo ntpdate time.windows.com
```

## 2. 然后将时间更新到硬件上

```
sudo hwclock --localtime --systohc
```

## 3. Enjoy！

<!-- more -->

---

# 三. Ubuntu 提示 /boot 空间不足的解决办法

Ubuntu 系统更新的时候，有时候会提示 /boot 空间不足，原因是 Linux 更新后，内核的旧版本不再使用，但还存放在 /boot 目录下。所以，手动将这些旧版本内核删除即可。

## 1. 查看旧版本内核

```
dpkg --get-selections|grep linux
```

看到带有 linux-image-x.x.x的就是旧版本。

## 2. 删除

```
sudo apt-get remove linux-image-(版本号)
```

## 3. 删除不干净的可以使用以下命令

```
sudo apt-get autoremove
```

## 4. Done！

---

# 四. Ubuntu 国内更新源

为了提高更新下载速度，可以把 Ubuntu 的更新源改为国内镜像。

## 1. 备份

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.old
```

## 2. 打开source.list

```
sudo gedit /etc/apt/source.list
```

## 3. 添加以下其中一个国内源并覆盖原内容

东北大学
```
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial main restricted #Added by software-properties
deb http://mirror.neu.edu.cn/ubuntu/ xenial main restricted
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial restricted multiverse universe #Added by software-properties
deb http://mirror.neu.edu.cn/ubuntu/ xenial-updates main restricted
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirror.neu.edu.cn/ubuntu/ xenial universe
deb http://mirror.neu.edu.cn/ubuntu/ xenial-updates universe
deb http://mirror.neu.edu.cn/ubuntu/ xenial multiverse
deb http://mirror.neu.edu.cn/ubuntu/ xenial-updates multiverse
deb http://mirror.neu.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirror.neu.edu.cn/ubuntu/ xenial-security main restricted
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirror.neu.edu.cn/ubuntu/ xenial-security universe
deb http://mirror.neu.edu.cn/ubuntu/ xenial-security multiverse
```

清华大学
```
# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse
```

阿里云
```
# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
deb-src http://archive.ubuntu.com/ubuntu xenial main restricted #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted
deb-src http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirrors.aliyun.com/ubuntu/ xenial-security universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-security multiverse
```

## 4. 更新

```
sudo apt-get update
```

---

# 五. 利用Xshell5 本地电脑和远程VPS服务器互传文件

购买了一台VPS，传文件还需要架设ftp服务器，实在是懒，找到一个用 Xshell5 传文件的方法，基本满足日常试用啦


## 1. 本地电脑端

打开 Xshell5 -- 文件 -- 属性 -- 文件传输 -- 使用下列下载路径

下载路径选择一个文件夹，存放从 VPS 下载到本地电脑的文件

加载路径选择一个文件夹，这是从本地电脑上传文件到VPS默认打开的路径

## 2. VPS端

```
sudo apt-get lrzsz
```

上传使用命令：
`rz`

输入`rz`后会从 windows 中选取文件，自动传输到VPS的当前目录下

下载使用命令：
`sz`

比如，要把VPS当前目录下的 gf.jpg 文件下载到本地电脑，直接`rz gf.jpg`
