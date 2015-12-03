---
layout: article
title: "ssh无密码访问"
modified:
categories: linux
#excerpt:
#tags: []
image:
#    feature: /teaser/xxx
    teaser: /teaser/ssh.png
#    thumb:
date: 2015-11-18T01:52:18+08:00
---

> 本文介绍通过ssh无密码连接服务器的方法

> 文章欢迎转载，但转载时请保留本段文字，并置于文章的顶部
> 作者：张纹华(shiziwen)
> 本文原文地址：<http://cenalulu.github.io{{ page.url }}>

```
ssh-keygen -t rsa
```

有询问的地方直接按回车，在/root/.ssh/下生成一对秘钥，id_rsa为私钥，id_rsa.pub为公钥（需要下发到被控主机用户.ssh目录，同时要求重命名为authorized_keys文件）

使用ssh-copy-id公钥拷贝工具：

```
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.1.21
```
