---
layout: article
title: "Git命令（一）"
modified:
categories: linux
#excerpt:
#tags: []
image:
#    feature: /teaser/xxx
    teaser: /teaser/git-command.jpg
#    thumb:
date: 2015-12-03T22:14:08+08:00
---


> 本篇文章记录了一些git经常会用到的一些命令，以便今后查找


> 文章欢迎转载，但转载时请保留本段文字，并置于文章的顶部
> 作者：张纹华(shiziwen)
> 本文原文地址：<http://shiziwen.github.io{{ page.url }}>

# git命令

## git commit -amend
修改已经提交的commit。
> Sometimes a mistake is made in a commit, but this is easy to correct with git commit --amend. When you use this command, git creates a new commit with the same parent as the current commit. (The old commit will be discarded if nothing else references it.)

<div style="text-align: center">
<img src="../../images/linux/git_command_1/commit-amend.svg"/>
</div>

## diff

查看暂存区域文件和最近提交的历史文件有何区别：

```
git diff --staged
```

在最后一次提交之后所做的修改：

```
git diff HEAD
```

显示一些小的改动：

```
git diff --color-words
git diff --word-diff
```


查看修改了的文件：

```
git diff --stat
```

## log

查看简洁的log：

```
git log --oneline
```

查看每次提交包含的文件：

```
git log --stat
```

产看每次提价修改的内容：

```
git log --patch
```

组合使用：

```
git log --patch --oneline
```


用ASCII码显示代码结构：

```
git log --graph
```

```
git log --graph --all --decorate --oneline
```

## remove

使用下面的命令，git会遍历工作数，寻找之前git已经识别到的现在要消失的文件，然后作为新的删除来暂存：

```
git add -u .
```

不再跟踪某个文件：

```
git rm --cached files
```

## move

移动文件：

```
git mv source dest
```

从当前目录无线向下递归，发现所有的移动过去的新文件，删除原来所有的旧文件，然后解释为已经发生了移动：

```
git add -A .
```

使用日志命令来展示某个文件在移动过程中的历史，
获取某个文件的全部提交：

```
git log --stat -- filename
```

告诉日志在文件移动过程中跟踪文件：

```
git log --stat -M --follow -- filename
```


## ignore

显示哪些文件被忽略了：

```
git ls-files --others --ignored --exclude-standard
```

## branch

删除分支：

```
git branch -d XXX
```

强制删除：
```
git branch -D XXX
```

## checkout

清理最后一次commit的内容：

```
git checkout -- filename
```

## merge

清除你的工作目录，从你当前分支的最后一次提交中，它同样会清除你的暂存区：

```
git merge --abort
```

不想把历史汇聚起来，但是想要一个具体分支中的全部commit：

```
git merge --squash branch-name
```

它会在你目前切换到的分支上，为你创建一个新的提交，这代表了目标分支上的所有改变。


## remote

修改远端url：

```
git remote set-url origin http://XXX
```

删除：

```
git remote rm origin
```

查看远端分支：

```
git branch -r
```

## reset
reset有三种模式，mixed，soft，hard。

* mixed是默认模式，它修改了历史和工作目录；
* soft模式是选取一条或者多条commit，把它们的全部改变放到暂存区，让你在此基础上继续创建新的提交；
* hard是一个具有破坏性的操作，这意味着擦除掉你不想再保存的东西；


```git reset HEAD```，允许我们把这些改变移除暂存区。

```git reset --soft HEAD~5```，可以把最新的5条commit当做一次提交，压缩到暂存区，这对重塑历史非常有帮助。

如果你想完全丢掉一些你的工作，可能想丢掉一些不再发挥作用的变动，你可以使用```git reset --hard```来完全丢掉这些代码：

```
git reset --hard HEAD~3
```

## rebase

```
git checkout add-red-button
git rebase master
```

rebase允许一个分支在历史操作中重新定位，意味着，把所有仅在你分支的所有变动，表现为，他们仿佛发生在主分支下的当前工作之后。

这和反向合并的方式相比，有着更加干净的历史。

<div style="text-align: center">
<img src="../../images/linux/git_command_1/rebase.png"/>
</div>




## Reference
* [A Visual Git Reference](http://marklodato.github.io/visual-git-guide/index-en.html)
* [GitHub&Git入门基础](http://www.nowcoder.com/courses/2/1/7)
