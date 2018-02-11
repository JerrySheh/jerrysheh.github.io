---
title: Ubuntu 下配置 shadowsock-qt
categories: linux
tags: Linux
abbrlink: 879f3462
date: 2017-09-12 10:40:00
updated: 2017-09-12 10:40:00
---

# Ubuntu 下配置 shadowsocks-qt

## 1. 安装

```
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt update
sudo apt install shadowsocks-qt5
```



## 2. 配置

添加服务器地址、密码、连接方式。 gui友好界面

## 3. 配置Chrome ，使用插件SwitchyOmega

到墙内网站搜索 SwitchyOmega 插件的 crx 离线包

打开 Chrome 的扩展程序，将 crx 托进去及安装完毕

## 4. 设置 SwitchyOmega

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

## 5. Have fun :)
