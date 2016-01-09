---
layout: article
title: "使用ssh和shadowsocks加速Bitbucket"
modified:
categories: 
#excerpt:
#tags: []
image:
#    feature: /teaser/xxx
    teaser: /teaser/bitbucket.jpg
#    thumb:
date: 2016-01-10T01:37:55+08:00
---

> 本文介绍如何使用shadowsocks来加速在Bitbucket上提交代码的速度

> 文章欢迎转载，但转载时请保留本段文字，并置于文章的顶部
> 作者：张纹华(shiziwen)
> 本文原文地址：<http://shiziwen.github.io{{ page.url }}>


## add ssh key
### 生成ssh key
```
$ssh-keygen
```
提示让你输入密钥的地址与名字，默认是 ```~/.ssh/id_rsa```，这里我们使用 ```~/.ssh/bitbucket_id_rsa```.

### 在Bitbucket账号中添加公钥
```
cat ~/.ssh/bitbucket_id_rsa.pub
```
将获得的代码复制bitbucket配置页面上面。


## 创建config文件
在```~/.ssh/config```文件中，填写如下内容：

```
Host bitbucket
 Hostname bitbucket.org
 User [username]
 IdentityFile /Users/mac/.ssh/bitbucket_id_rsa
 IdentitiesOnly yes
 ProxyCommand nc -x 127.0.0.1:1080 -X 5 %h %p
```
这里的“User”指的是你在Bitbucket上面的账号，ProxyCommand配置成你的shadowsocks的地址。

## 更新remote配置
```
git remote set-url <name> <newurl>
```
这里的```<newurl>```的格式为```git@bitbucket.org:XXX/XXX.git```

完成这些步骤之后，执行：

```
git push -u origin master
```

你会发现速度较之前是天壤之别。


## 参考资料
* [Bitbucket上使用SSH协作](http://www.cnblogs.com/boychenney/archive/2013/04/09/bitbucket_collaboration_with_ssh.html)
* [Bitbucket ssh public key is being denied but their ssh test connects with no issue](http://stackoverflow.com/questions/31660263/bitbucket-ssh-public-key-is-being-denied-but-their-ssh-test-connects-with-no-iss)
* [Set up SSH for Git](https://confluence.atlassian.com/bitbucket/set-up-ssh-for-git-728138079.html)
