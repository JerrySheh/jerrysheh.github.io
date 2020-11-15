---
title: Android 10 ROOT
tags: Android
categories: 瞎折腾
abbrlink: 3903647e
date: 2020-08-21 23:25:16
---


# 前言

最近收到了一加推送的 OnePlus 5T 氢OS Android 10.0.1 的固件更新，不得不说一加还是很良心的，三年前的机型还保持着官方升级。立马更新尝鲜了，但是更新后，发现我的 ROOT 权限没了。于是到一加社区一番搜索，发现了 [这个帖子](https://www.oneplusbbs.com/forum.php?mod=viewthread&tid=5398217) 提到可以用 platform-tools （这个工具应该不陌生了，线刷必备） 手动刷 magisk.apk 和 magisk_patched.img。 magisk.apk 我知道是我们熟悉的面具软件，但直觉告诉我这个 img 应该是跟固件相关的，帖子里是 10.0 公测版的固件，img 肯定跟 10.0.1 不同了。

那么，从哪里制作对应固件的 magisk_patched.img 呢？

<!-- more -->

# Google It

经过一番 Google，在 [magisk 的官方文档](https://topjohnwu.github.io/Magisk/install.html) 里找到了这么一段话：

> You would want to choose this method if either your device does not have custom recoveries, your device is A/B and you don’t want to mix recovery and boot images, or your device is using system-as-root without A/B partitions.
>
> To use this method, you are required to obtain a copy of the stock boot/recovery image, which can be found by extracting OEM provided factory images or extracting from OTA update zips. If you are unable to obtain one yourself, you might be able to find it somewhere on the internet. The following instructions will guide you through the process after you have the copy of boot/recovery image.

意思是说，只要我们能拿到固件刷机包里的 boot image 文件， 用 Magisk 软件可以对其进行 patch， 实际上 magisk_patched.img 就是 boot.img 的修改。并给出了步骤：

- Copy the boot/recovery image to your device
- Download and install the latest Magisk Manager
- If you are patching a recovery image, manually check “Recovery Mode” in Advanced Settings!
- Press Install → Install → Select and Patch a File, and select your stock boot/recovery image file
- Magisk Manager will patch the image, and store it in [Internal Storage]/- Download/magisk_patched.img
- Copy the patched image from your device to your PC. If you can’t find it via MTP, you can pull the file with ADB:
- adb pull /sdcard/Download/magisk_patched.img
Flash the patched boot/recovery image to your device and reboot. For most devices, here is the fastboot command:
```
fastboot flash boot /path/to/magisk_patched.img
# or
fastboot flash recovery /path/to/magisk_patched.img
```

刚刚下载固件时忘记保存安装包了，更新后被系统升级自动删除了。于是又到一加社区搜到了全量包的下载地址，解压后把里面的 boot.img 提取出来，照着上面的方法很快就制作好了 magisk_patched.img 。


# Do It

简单复述下步骤，如下：

## 1. 解锁BootLoader

>  <font color='red'>警告：解锁BootLoader会清空数据，请提前备份 </font>

开发者选项OEM解锁打开，开启USB调试连接电脑，adb输入以下命令重启至 bootloader 模式

```
adb reboot bootloader
```

之后在手机上选择 unlock， 重启即解锁

### 2. 提取 boot.img

到官网下载固件全量包，如果是zip，解压缩能看到 boot.img， 直接把 boot.img 拖出来。如果是 payload.bin ，需要用 Payload_Dumber_x64 工具（网上下载）提取（源码参考 [github](https://gist.github.com/ius/42bd02a5df2226633a342ab7a9c60f15) ）。

### 3. platform-tools

使用 platform-tools （网上下载）安装 magisk（网上下载），使用 adb 安装到手机

```
adb install magisk.apk
```

### 4. 制作 magisk_patched.img

就是在 magisk app 里面点安装，再点选择并修补一个文件，将前期提取的固件里的 boot.img 放到手机存储里面，选择你的 boot.img ，然后 magisk 会自动修复，并生成 magisk_patched.img，将 magisk_patched.img 提取到电脑 platform-tools 目录下。

### 5. 刷入img

重启到 bootloader，刷入img。这一步的意思是使用我们的 magisk_patched.img 来驱动手机启动。

```
adb reboot bootloader
fastboot boot magisk_patched.img
```

手机自动重启后，即获得临时 root 权限。

### 6. 直接安装

到 magisk app 里面点安装，再点直接安装，即可获得永久 root 。
