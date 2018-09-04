---
title: Ubuntu的一些奇技淫巧
categories: linux
tags:
  - Linux
abbrlink: 2656dc91
date: 2017-09-12 20:14:40
---

接触Linux越久，掉进莫名其妙的坑里就越多，于是我决定每遇到一个坑就记录下来，这样以后再踩的时候不至于爬不起来。

Ubuntu的一些使用技巧

目前 get 的有：
1. 调整鼠标速度
2. 解决双系统时间不同步的问题
3. 系统更新提示 /boot 空间不足的解决办法
4. 更改国内源，提高下载速度
5. 利用 Xshell / SSH 在远程和本地之间传文件
6. 编辑菜单
7. 终端使用SS，查公有ip
8. 管理ppa源
9. 安装 jdk 1.8
10. 更改 root 账户密码
11. apt安装不成功，每次 apt install 都报错
12. 使用openssh远程连接
13. ROOT账户没有环境变量
14. 如何正确地配置环境变量
15. Ubuntu出现了内部错误
16. Ubuntu/Debian 完全卸载Mysql

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

看到带有 Linux-image-x.x.x的就是旧版本。

## 2. 删除

```
sudo apt-get remove Linux-image-(版本号)
```

## 3. 删除不干净的可以使用以下命令

```
sudo apt-get autoremove
```

## 4. Done！

---

# 四. Ubuntu 国内更新源

为了提高更新下载速度，可以把 Ubuntu 的更新源改为国内镜像。推荐使用阿里云源。因为大学的服务器在某些特殊时期因为某些原因可能无法访问，你懂的。

## 1. 备份

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.old
```

## 2. 打开source.list

```
sudo gedit /etc/apt/source.list
```

## 3. 添加以下国内源并覆盖原内容

阿里云（推荐）
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

# 五. 利用 Xshell / SSH 在远程和本地之间传文件

~~购买了一台VPS，传文件还需要架设ftp服务器，实在是懒~~ 找到一个用 Xshell 传文件的方法，基本满足日常试用啦。

2018.4.1 更新：后来发现 SSH 居然可以传文件！ 方法写在下面


## 使用 Xshell

### 本地

打开 Xshell5 -- 文件 -- 属性 -- 文件传输 -- 使用下列下载路径

下载路径选择一个文件夹，存放从 VPS 下载到本地电脑的文件

加载路径选择一个文件夹，这是从本地电脑上传文件到VPS默认打开的路径

## 远程

```
sudo apt-get install lrzsz
```

上传使用命令：
`sudo rz`

输入`rz`后会从 windows 中选取文件，自动传输到VPS的当前目录下

下载使用命令：
`sudo sz`

如果没有使用`sudo`，可能导致卡在上传中。

比如，要把VPS当前目录下的 gf.jpg 文件下载到本地电脑，直接`rz gf.jpg`

## 使用 SSH

### SSH登录

```
ssh jerry@97.61.22.1 -p22
```

这里的 jerry 是你 VPS 的用户名， 97.61.22.1 是 VPS 公网IP地址 ， -p22 指 SSH 的22端口

> 如果报 WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED！ 错误，是因为服务器可能重装了系统，用 `ssh-keygen -R [server ip]` 来更新 ssh-key

### SSH本地传文件到远程

```
scp /path/filename jerry@servername:/path/
```

例如scp /var/www/test.php root@192.168.0.101:/var/www/ 把本机/var/www/目录下的test.php文件上传到192.168.0.101这台服务器上的/var/www/目录中

### SSH从远程下载文件到本地

```
scp username@servername:/path/filename /var/www/local_dir（本地目录）
```

例如 scp root@192.168.0.101:/var/www/test.txt 把192.168.0.101上的/var/www/test.txt 的文件下载到/var/www/local_dir（本地目录）

---

# 六、编辑开始菜单

```
sudo apt install alacarte
```

然后直接在Ubuntu终端输入命令alacarte。可以任意增、改、隐藏、显示菜单，但无法删除菜单，即使拥有root权限。

---

# 七、终端使用SS，并查公有ip

首先根据[这篇文章](https://jerrysheh.github.io/post/879f3462.html)配置好SS软件。

然后在终端中

```
export ALL_PROXY=socks5://127.0.0.1:1080
```

* 注意: 该命令仅对本终端一次性有效

查看公有ip

```
curl ipinfo.io/ip
```

---

# 八、管理ppa源

Ubuntu 软件仓库十分方便，但是有一些软件是在第三方库里的，因此我们要添加相应的ppa源，才能用 apt install

```
sudo add-apt-repository ppa:ownername/projectname
sudo apt update
sudo apt install something
```

有些库我们已经不需要了，用文本编辑器修改`/etc/apt/sources.list.d/`文件夹下对应的.list即可

```
sudo rm /etc/apt/sources.list.d/xxxxxx.list
```

---

# 九、如何正确地在Ubuntu 16.04 安装 JDK1.8

其实很简单

添加ppa源
```
sudo apt install software-properties-common
sudo add-apt-repository ppa:webupd8team/java
sudo apt update
```

如果第二行提示`add-apt-repository: command not found`，安装一些包即可
```
sudo apt-get install software-properties-common python-software-properties  
```

安装jdk8
```
sudo apt-get install oracle-java8-installer
```

如果提示没有公钥，添加对应的公钥（deeepin系统会有这个问题）

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C2518248EEA14886
```

如果你有其他版本的 jdk， 更改默认
```
sudo update-alternatives --config java
```

输出
```
* 0            /usr/lib/jvm/java-7-oracle/jre/bin/java          1062      auto mode
  1            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java  1061      manual mode
  2            /usr/lib/jvm/java-8-oracle/jre/bin/java          1062      manual mode

Press enter to keep the current choice[*], or type selection number:
```

选择想默认使用的版本即可

---

# 十、更改 root 账户密码

```
sudo passwd root
```

---

# 十一、apt安装不成功，每次 apt install 都报错

oracle-java7-installer安装不成功，清除缓存，不要每次 apt install 都报错

```
sudo rm /var/lib/dpkg/info/oracle-java7-installer*

sudo apt-get purge oracle-java7-installer*

sudo rm /etc/apt/sources.list.d/*java*

sudo apt-get update
```

---

# 十二、使用openssh远程连接

查看是否已经安装openssh （如果装了，应该输出不止一个ssh）
```
ps -ef | grep ssh
```

安装
```
sudo apt install openssh-server
```

这样就可以在 win10 用 Xshell 连接虚拟机开多个终端了。

---

# 十三、ROOT账户没有环境变量

在/root/.bashrc文件尾部添加：
```
source /etc/profile
```

保存后执行：
```
./root/.bashrc
```

如果提示没有权限

```
chmod +x ./root/.bashrc
```

就ok了

---

# 十四、如何正确地配置环境变量

```shell
#Java env.
export JAVA_HOME=/your_Java_home
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:/usr/local/protobuf/bin:$PATH

#Scala env.
export SCALA_HOME=/your_Scala_home
export PATH=$SCALA_HOME/bin:$PATH

#Spark env.
export SPARK_HOME=/your_Spark_home
export PATH=$SPARK_HOME/bin:$PATH

#Python env.
export PYTHONPATH=/your_python_home

#Hadoop env.
export HADOOP_HOME=/your_hadoop_home
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export HADOOP_HOME_WARN_SUPPRESS=not_null

#Mahout env.
export MAHOUT_HOME=/your_Mahout_home
export MAHOUT_CONF_DIR=$MAHOUT_HOME/conf
export PATH=$MAHOUT_HOME/conf:$MAHOUT_HOME/bin:$PATH
```

---

# Ubuntu出现了内部错误

从 14.04 到 16.04 到 18.04 ，无论哪个版本动不动就报错，临时解决办法：

```
cd /var/crash/
sudo rm *
```

删除错误日志，这样下次开机不会继续报错。但这不代表系统就没错误了，下次遇到奇奇怪怪的问题是还是会报错。几乎无解。

---

# 完全卸载 Mysql

```
sudo apt-get purge mysql-server mysql-common mysql-client
sudo rm -rf /etc/mysql
sudo rm -rf /var/lib/mysql
sudo rm -rf /var/log/mysql
sudo rm -rf /var/run/mysqld
```
