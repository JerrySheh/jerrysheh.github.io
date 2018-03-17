---
title: Hadoop 配置遇到的一些坑
comments: true
date: 2018-03-15 22:20:00
categories: 大数据
tags: Hadoop
---

---

根据官方文档 Hadoop 3.0 配置，在

```
sbin/start-dfs.sh
```
的时候报错，
```
pdsh@ubuntu: localhost: connect: Connection refused
```

原因是pdsh默认采用的是rsh登录，修改成ssh登录即可，在环境变量/etc/profile里加入：

```
export PDSH_RCMD_TYPE=ssh
```

然后`source /etc/profile` 之后重新sbin/start-dfs.sh
