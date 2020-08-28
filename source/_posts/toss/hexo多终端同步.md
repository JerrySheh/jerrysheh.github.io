---
title: Hexo多终端同步
categories: 瞎折腾
tags:
  - linux
abbrlink: 63b47f65
date: 2017-09-13 23:14:59
---

Hexo 多终端同步问题

我有两台电脑，一台 windows， 一台 Ubuntu 。之前在 Windows 机器下部署了 hexo 博客，现在想在另一台机子的 Ubuntu 系统下同步之前的博客，折腾了一晚上终于搞定。

<!-- more -->

# 一、将网站文件上传到 github

hexo部署完毕之后，在 yourname.github.io 上面默认 master 分支是 hexo 编译生成的静态网站，而我们需要将原始网站文件同步到一个新分支上。

## 1. 新建一个 stat 分支并切换到这个分支

```
git checkout -b stat
```

## 2. 将本地文件上传到 stat 分支

```
git add .
git commit -m "upload static file"
git push origin stat:stat
```

现在，你的 github 仓库 yourname.github.io 下就有两个分支了，一个是 master，存放 hexo编译生成的静态网站，一个是 stat，也就是原始文件。

---

# 二、 在新电脑下同步你的原始文件

## 1. 首先安装 nodejs

```
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs
```

## 2. 安装 git

```
sudo apt-get install git
```

## 3. 安装 hexo

```
sudo npm install hexo -g
```

## 4. 配置 git SSH key

如果是第一次在这台电脑用git，需要先添加 user.name 和 user.email

可参考我之前写的 [git 备忘](https://jerrysheh.github.io/post/9f9a74a3.html)

然后再配置SSH key

参考：[配置SSH key详细教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/001374385852170d9c7adf13c30429b9660d0eb689dd43a000)

## ~~5. 在合适的位置新建一个目录，并在这个目录下打开终端~~

## 5. 打开一个合适的目录位置


## 6. 克隆原始文件（stat）分支到本地

```
git clone git@github.com:JerrySheh/JerrySheh.github.io.git -b stat blog
```

这里的 stat 是远程分支名字， blog是新建名为blog的文件夹并把这个分支clone到这个文件夹里

## 7. 切换到克隆生成的目录

```
cd blog
```

## 8. 依次执行

```
npm install hexo
npm install
npm install hexo-deployer-git
```

注意：不需要 hexo init ！

* 这里有一个坑

由于Ubuntu下已经有一个名叫node的库，因此Node.js在ubuntu下默认叫nodejs，需要额外处理一下

```
sudo ln -s /usr/bin/nodejs /usr/bin/node
```

## 9. 尝试修改些什么，然后执行

```
hexo clean
hexo g
hexo s --debug
```

如果出现Error: Cannot find module 'hexo-util'错误，尝试重装util
```
npm install -- save-dev hexo-util
```

没问题了，开始部署

```
hexo d
```

## 10. 将静态文件上传到 github stat分支

```
git add .
git commit -m "somethings update"
git push origin stat
```

---

# 三、 回到旧电脑，拉取新电脑的更新

## 1. 同步

```
git pull origin stat:stat
```

如果遇到以下报错
```
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

git branch --set-upstream-to=origin/<branch>
```

将本地 static 和 远程 origin/static 链接即可

```
git branch --set-upstream-to=origin/stat stat
```

## 2. 开始写文章

## 3. 重新部署

```
hexo clean
hexo g
hexo d
```

部署前最好先在本地测试一下 ， 使用 `hexo s --debug`，然后在 127.0.0.1:4000 查看

## 4. 提交静态文件并推送到远程

```
git add .
git commit -m "somethings update"
git push origin stat
```
