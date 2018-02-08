---
title: 给自己的 git 备忘
categories: 技术&技巧
tags: git
abbrlink: 9f9a74a3
date: 2017-09-04 15:43:46
updated: 2017-09-04 17:04:01
---



给自己的 git 备忘

---

# git使用流程


## 1. 于一个目录下，初始化git


`git init`


## 2. 新建/修改文件后，把修改内容 add 到 git 上面


`git add .`


### 三个 add 的区别

  `git add . `
  他会监控工作区的状态树，使用它会把工作时的所有变化提交到暂存区，包括文件内容修改(modified)以及新文件(new)，但不包括被删除的文件。

  `git add -u `
  他仅监控已经被add的文件（即tracked file），他会将被修改的文件提交到暂存区。add -u 不会提交新文件（untracked file）。（git add --update的缩写）

  `git add -A `
  是上面两个功能的合集（git add --all的缩写）

## 3. 提交到仓库


`git commit -m "modified"`


## 4. 关联一个远程仓库


`git remote add origin git@github.com:JerrySheh/repository_name.git`


## 5. 推送到远程

`git push origin branch_name`



如果远程已经有文件，需要先pull


`git pull origin master`


<!-- more -->

---

# git 分支管理


## · 新建并切换到分支
`git checkout -b branch_name`


#### · 把新建的本地分支 push 到远程
`git push origin branch_name:branch_name`


#### · 删除远程分支
`git push origin -d branch_name`


---

# git 撤销


## · 查看更改日志，找到你想返回去的commit_id （一般第一条是你搞错了的，第二条就是上次你想返回去的id）
`git log`

## ·  撤销提交和修改过的代码
`git reset --hard commit_id`

## · 只撤销提交，不撤销修改过的代码（可以直接通过git commit 重新提交对本地代码的修改。）
`git reset commit_id `

## · 如果想撤销的commit已经提交到远程仓库了，在本地 reset 修改后，重新强制提交。这样上次的错误提交就消失了。但是这样做有个弊端，就是如果你的错误commit（如commit3）之后还有其他人再提交了(commit4)， commit4也会消失。

`git push --force`

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
