---
title: 使用Synergy在两台电脑之间共享键盘鼠标
categories: 瞎折腾
tags:
  - Ubuntu
abbrlink: da6510de
date: 2018-02-09 02:00:41
---

之前写过[如何在两台电脑中同步hexo博客编辑](https://jerrysheh.github.io/post/63b47f65.html)，利用 git 和 github 同步hexo的博客。

然后又经历了[一次玄幻的重装系统经历](https://jerrysheh.github.io/post/e84807f.html)

都是两台电脑引起的。

然鹅这次，一切又要从有两台电脑说起...

---

<!-- more -->

# 问题

有两台电脑，一台是 Windows 10 操作系统， 另一台是 Ubuntu 16.04， 两台电脑同时使用的时候，只有一套键鼠，插来拔去非常不方便。

有没有什么办法，一套键鼠，同时控制两台电脑呢？ 答案是肯定的。

# Synergy

网上搜寻了很多解决方案，最佳的貌似只有 Synergy 了。

Synergy是一款跨平台软件，可以实现多台电脑共用一个鼠标和键盘。

官方的介绍是：

> Synergy将您的桌上设备结合，升华成为一种综合性的体验。这是一款能够让您的几台电脑，共享一个鼠标和键盘的软件。它可在Windows、macOS和Linux上运行。

![](https://symless.com/img/synergy/homepage-explainer-graphic.png)


但是查资料发现，Synergy以前是有免费版的，现在好像要付费了，价钱是19美元/终身，淘宝也有一些代理只要19人民币永久授权。

~~不过，在 github 上面还是能搜到免费开源的版本~~

> 该版本已不可用，老老实实买正版吧

项目地址：https://github.com/brahma-dev/synergy-stable-builds/

下载地址：https://www.brahma.world/synergy-stable-builds/

---

# 开始使用

建议两个系统使用同一个版本，且最好下载最新版（目前2018/02/09是 1.8.8 版本），如果版本不一样或者虽然一样但是版本比较旧，可能会产生一些奇奇怪怪的问题。比如我一开始折腾的时候下载了1.8.7 和 1.6.0 （Ubuntu apt仓库的版本），就出现了鼠标在客户机无法移动的问题。

我这里 Windows 10 下载 Windows X64 ， Ubuntu 下载 Ubuntu/Debian - x86_64 （v1.8.8-stable）

## Windows 10 服务机配置

直接安装，然后选择 Server 作为服务端，点击设置服务端，将右上角的电脑拖到合适的位置（根据你的办公桌两台电脑摆放的相对位置），然后双击刚刚放上去的电脑图标，名字改为Ubuntu系统的电脑名字。

最后点击应用，然后点开始，服务就启动了。可以把软件最小化到托盘。

## Ubuntu 客户机配置

安装deb包
```
sudo dpkg -i synergy-v1.8.8-stable-Linux-x86_64.deb
```
安装相关依赖
```
sudo apt-get install -f
```
启动synergy
```
synergy
```

然后选择 Client 作为客户端，服务端ip填局域网服务机的 ip （在win10下打开cmd输入 ipconfig /all， 192.168.x.x 那个），或者在synergy软件可以看到这个ip地址。

然后直接点击应用，开始。

这样，两台电脑就实现了两台电脑共享一套键鼠的神操作了。

---

# 最后

Synergy 还支持复制粘贴文件，剪贴板等功能，不得不说确实很强大。

但是有时候在客户机操作的时候会发现有一点点卡顿，或者失灵，很大的可能性是wifi不稳定导致的。解决办法很简单，这里提供两个方法：

* 用一根网线将两台电脑连接起来，然后在以太网设置里给两部机分别分配一个ip，再重新填一下synergy客户机的ip地址，连接外网可以用wifi

* win10自带共享热点，在`设置-网络和Internet-移动热点`里面开一下热点，然后客户机连接这个热点上网即可，当然别忘了重新填一下synergy客户机的ip地址

![](../../../../images/win10_net_share.png)


（完）
