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


## 配置ssh相关文件

### 更新bashrc文件

```
SSH_ENV=$HOME/.ssh/environment

function start_agent {
     echo "Initialising new SSH agent..."
     /usr/bin/ssh-agent | sed 's/^echo/#echo/' > ${SSH_ENV}
     echo succeeded
     chmod 600 ${SSH_ENV}
     . ${SSH_ENV} > /dev/null
     /usr/bin/ssh-add;
}

# Source SSH settings, if applicable

if [ -f "${SSH_ENV}" ]; then
     . ${SSH_ENV} > /dev/null
     #ps ${SSH_AGENT_PID} doesn't work under cywgin
     ps -ef | grep ${SSH_AGENT_PID} | grep ssh-agent$ > /dev/null || {
         start_agent;
     }
else
     start_agent;
fi
```

把上面这段内容直接保存到```~/.bashrc```文件中，然后执行```source ~/.bashrc```进行加载。

### 添加key
然后进入```~/.ssh/config```目录，添加key：

```
cd ~/.ssh/config
ssh-add  bitbucket_id_rsa
```

### 测试
输入下面命令确认能与Bitbucket网站SSH通信：

```
ssh -T git@bitbucket.org
```

如果出现下面的信息：

```
conq: logged in as jscon.
You can use git or hg to connect to Bitbucket. Shell access is disabled.
```

则说明你已经能用这个key与bitbucket网站SSH通信了。


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
