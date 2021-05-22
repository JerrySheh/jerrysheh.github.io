---
title: 给自己的 git 备忘
categories: 
- 技能
- git
tags: git
abbrlink: 9f9a74a3
date: 2017-09-04 15:43:46
updated: 2017-09-04 17:04:01
---

给自己的 git 备忘：

1. git 使用流程
2. git 分支管理
3. git 撤销
4. 连接到 github
5. fork
6. IDEA git 项目颜色含义
7. git stash 暂存

<!-- more -->

---

# git使用流程

## 初始化

于一个目录下，初始化git

```
git init
```

## add

新建/修改文件后，把修改内容 add 到 git 上面

```
git add .
```

### 三个 add 的区别

- `git add . `
 他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。

- `git add -u `
 他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区。add -u 不会提交新文件（untracked file）。（git add --update的缩写）

- `git add -A `
  是上面两个功能的合集（git add --all的缩写）


## commit 提交到仓库

```
git commit -m "modified"
```

---

# 关联和同步

## 关联远程仓库

```
git remote add origin git@github.com:JerrySheh/repository_name.git
```

## 推送到远程

第一次推送，加 -u 参数

```
git push origin branch_name
```

## 同步

如果远程已经有文件，需要先 pull

```
git pull origin master
```

## 从远程仓库克隆

```
git clone git@github.com:jerrysheh/helloworld
```

---

## diff

`git diff  filepath` 工作区与暂存区比较

`git diff HEAD filepath` 工作区与HEAD ( 当前工作分支) 比较

`git diff --staged` 或 `--cached  filepath` 暂存区与HEAD比较

`git diff branchName filepath`  当前分支的文件与branchName 分支的文件进行比较

`git diff commitId filepath` 与某一次提交进行比较

---

# git 分支管理


新建并切换到分支

```
git checkout -b branch_name
```

把新建的本地分支 push 到远程

```
git push origin branch_name:branch_name
```

删除远程分支

```
git push origin -d branch_name
```

---

# git 撤销

查看更改日志，找到你想返回去的commit_id （一般第一条是你搞错了的，第二条就是上次你想返回去的id）

```
git log
```

撤销提交和修改过的代码

```
git reset --hard commit_id
```

只撤销提交，不撤销修改过的代码（可以直接通过 `git commit` 重新提交对本地代码的修改）

```
git reset commit_id
```

如果想撤销的commit已经提交到远程仓库了，在本地 reset 修改后，重新强制提交。这样上次的错误提交就消失了。但是这样做有个弊端，就是如果你的错误commit（如commit3）之后还有其他人再提交了(commit4)， commit4也会消失。

```
git push --force
```

---

# 在新电脑配置git，并连接到githiub

设置 git 的username 和 usermail

```
git config --global user.name "yourname"
git config --global user.email "youremail"
```

生成SSH密钥

查看是否已经有了ssh密钥：`cd ~/.ssh`
如果没有密钥则不会有此文件夹，有则备份删除
生成密钥：
```
ssh-keygen -t rsa -C “haiyan.xu.vip@gmail.com”
```

按3个回车，密码为空。
```
Your identification has been saved in /home/tekkub/.ssh/id_rsa.
Your public key has been saved in /home/tekkub/.ssh/id_rsa.pub.
The key fingerprint is:
………………
```

最后得到了两个文件：id_rsa和id_rsa.pub

在 .ssh 文件夹中执行 `ssh-add id_rsa`，再输入正确密码

在github上添加ssh密钥，这要添加的是“id_rsa.pub”里面的公钥。

打开  https://github.com/ ,在设置中添加密钥

测试：
```
$: ssh git@github.com
The authenticity of host ‘github.com (207.97.227.239)’ can’t be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added ‘github.com,207.97.227.239′ (RSA) to the list of known hosts.
ERROR: Hi Jerrysheh! You’ve successfully authenticated, but GitHub does not provide shell access
```

测试成功就可以push code和 git clone 之类的操作了。

---

# fork

一般如果要参与开源项目，都是先 fork 别人的项目到自己的github，然后通过 `git clone` 自己的仓库克隆到本地进行修改。修改完毕后，通过`pull request`向原作者提交合并申请。

但是我们 fork 了别人的项目之后， 原作者 commit 了新内容， 我们 fork 的项目并不会更新。解决办法为：在本地建立两个库的中介，把两个远程库都clone到本地，然后拉取原项目更新到本地，合并更新，最后push到你的github。

流程如下：

```
git clone https://我们fork的地址.git
```

cd 进入该目录，`git remote -v`，可以看到只有两条我们自己的。

```
git remote add nsd https://原作者的地址.git  
```

再次`git remote -v`，可以看到多出来两条。

> nsd 只是一个别名，可以任意取

把原作者的更新 fetch 到本地

```
git fetch nsd
```

查看分支

```
git branch -av
```

可以看到，出现一条 `remotes/nsd/master xxxxx` 的更新

合并

```
git checkout master
git merge nsd/master
```

如果有冲突，需要丢掉本地分支

```
git reset –hard hunter/master
```

这时你的当前本地的项目变成和原作者的主项目一样了，可以把它提交到你的GitHub库

```
$ git commit -am ‘更新到原作者的主分支’
$ git push origin
$ git push -u origin master -f –强制提交
```

- 参考：https://www.jianshu.com/p/633ae5c491f5

---

# LF/CRLF

LF will be replaced by CRLF

禁用自动转换即可

```
git config –global core.autocrlf false
```

---

# 代理

```shell
# 设置sock5代理
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'

# 设置http代理
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080

# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy
```

---

# IDEA git项目颜色含义

- 绿色： 创建了仓库没有的新文件，还未提交
- 蓝色： 仓库已有的文件，被修改了，还未提交
- 红色： 没有添加到版本控制的文件（包括 ignore 的）

参考：
- [IntelliJ IDEA 中git的使用图文教程](https://www.jb51.net/article/135583.htm)

---

# git stash 暂存

有时候我们在本地写了一些代码，之后可能紧急要切换到另一个分支做一些修复，可以先将本地变更的代码“暂存”，稍后再在任意分支恢复。

暂存代码

```
git stash
```

切换到别的分支工作

```
git checkout master
```

工作完换回来

```
git switch dev
```

查看暂存区

```
git stash list
```

暂存内容恢复

```
git stash pop
```