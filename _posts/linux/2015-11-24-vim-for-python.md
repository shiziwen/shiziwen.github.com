---
layout: article
title: "Vim for Python"
modified:
categories: linux
#excerpt:
#tags: []
image:
#    feature: /teaser/xxx
    teaser: /teaser/vim-brain.png
#    thumb:
date: 2015-11-24T14:54:18+08:00
---

> 本文介绍使用与Python开发环境的Vim配置

> 文章欢迎转载，但转载时请保留本段文字，并置于文章的顶部
> 作者：张纹华(shiziwen)
> 本文原文地址：<http://cenalulu.github.io{{ page.url }}>

## 资料
* [Vim 与 Python 真乃天作之合](http://gold.xitu.io/entry/564d61e560b294bc12973107)
* [VIM and Python - a Match Made in Heaven](https://realpython.com/blog/python/vim-and-python-a-match-made-in-heaven/)
* [Vim配置文件链接](https://github.com/j1z0/vim-config/blob/master/vimrc)

## vim版本
vim的版本需要>7.3

```
$vim --version
```

## 安装Vundle
```
$git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

## 编写.vimrc
```
$touch ~/.vimrc
```

可以参考文章开头列出的资料

## 安装插件
在vim中，输入命令

```
:PluginInstall
```

## 安装YCM
```
$sudo apt-get install build-essential cmake
$cd ~/.vim/bundle/YouCompleteMe
$./install.py --clang-completer
```

