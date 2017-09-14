---
title: Ubuntu的一些奇技淫巧
date: 2017-09-12 20:14:40
tags: linux
---

# Ubuntu的一些奇技淫巧

### 一. 调整鼠标速度

```
xset m N
```

* 其中 N 是速度速度值，0（最慢）- 10（最快）

</br>
### 二. 解决 win10 + Ubuntu 双系统 时间不同步的问题

#### 1. 先在 Ubuntu 下更新时间，确保时间无误

```
sudo apt-get install ntpdate
sudo ntpdate time.windows.com
```

#### 2. 然后将时间更新到硬件上

```
sudo hwclock --localtime --systohc
```

#### 3. Enjoy！
