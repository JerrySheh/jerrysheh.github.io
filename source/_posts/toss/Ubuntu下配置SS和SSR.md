---
title: Ubuntu 下配置SS和SSR
categories: 瞎折腾
tags: Linux
abbrlink: 879f3462
date: 2017-09-12 10:40:00
updated: 2017-09-12 10:40:00
---

# Ubuntu 下配置 shadowsocks-qt

注意：shadowsocks-qt只能用于SS，不能用于SSR

## 安装

```
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt update
sudo apt install shadowsocks-qt5
```

<!-- more -->

> 对于 UBuntu 18.04

> ppa:hzwhuang/ss-qt5 并没有18.04版本的源，所以再执行update时会出现

> E: 仓库 “http://ppa.launchpad.net/hzwhuang/ss-qt5/ubuntu bionic Release” 没有 Release 文件 的错误。

> 这时，只要编辑 /etc/apt/sources.list.d/hzwhuang-ubuntu-ss-qt5-bionic.list 文件，将bionic (18.04版本代号)改成xenial（16.04版本代号）。

## 配置

添加服务器地址、密码、连接方式。 gui友好界面。

## 配置Chrome ，使用插件SwitchyOmega

用你能想到的方法找到Chrome SwitchyOmega 插件的 crx 离线包

打开 Chrome 的扩展程序，将 crx 托进去及安装完毕

### 第一步

1. 新建一个情景模式，更名 shadowsock
2. 代理协议选择 sock5，代理服务器 127.0.0.1， 代理端口 1080

### 第二步

1. 选择 autoswitch
2. 规则列表设置选择 Autoproxy
3. 规则列表网址如下
  ```
  https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
  ```
4. 立即更新情景模式
5. 切换规则 - 规则列表规则选择 shadowsock

Have fun :)

---

# 使用 SSR 客户端

## 安装

```
wget http://www.texfox.com/ssr  
sudo mv ssr /usr/local/bin
chmod 766 /usr/local/bin/ssr  
ssr install  
```

> 一般安装的或者自己写的把全局脚本放在 usr/local/bin 下，这样在终端任意位置可以使用脚本命令

## 配置

```
ssr config
```

然后在弹出来的配置文件中填写你的服务器信息

```json
"server":"0.0.0.0",        //服务器ip  
"server_port":8388,        //端口  
"password":"m",            //密码  
"protocol":"origin",       //协议插件  
"obfs":"http_simple",      //混淆插件  
"method":"aes-256-cfb",    //加密方式  
```

一般配置完之后就会自动启动，如果没有，使用以下命令来启动

```
ssr start
```

使用以下命令来关闭

```
ssr stop
```

Chrome 配置同SS
