---
title: "os x 下面禁止并删除.DS_Store文件"
date: 2015-04-13T00:00:00+0800
lastmod: 2015-04-13T00:00:00+0800
draft: false
keywords: ["os x",".DS_STORE","删除"]
tags:  ["os x",".DS_STORE","删除"]
description: ".DS_Store如果不小心上传到github或者网站中会泄露自己的网站目录与路径的"
categories: ["常用技巧"]
tags: ["os x",".DS_STORE","删除"]
---

## 查看.DS_Store文件结构

先来看看怎么样解析.DS\_Store文件，代码来自: [https://github.com/ff0000team/Beebeeto-framework/blob/master/utils/tools/ds_store_viewer.py](https://github.com/ff0000team/Beebeeto-framework/blob/master/utils/tools/ds_store_viewer.py)


```python
#!/usr/bin/env python
# coding=utf8
# author=evi1m0@201503
# poc: http://www.beebeeto.com/pdb/poc-2015-0052/
# install: https://pypi.python.org/pypi/ds_store

import sys

from ds_store import DSStore


if len(sys.argv) < 2:
    print '[*] Usage: %s path/.DS_Store' % sys.argv[0]
    sys.exit()

filelist = []
filename = sys.argv[1]
try:
    with DSStore.open(filename, 'r+') as obj:
            for i in obj:
                filelist.append(i.filename)
except Exception, e:
    print '[-] Error: %s' % str(e)
for name in set(list(filelist)):
    print '[*] ' + name
```

运行上面的代码的结果如下：

```
beyond@birdcat:~/code/code-python|⇒  python ds_store_view.py ~/.DS_Store
[*] .oh-my-zsh
[*] hacker
[*] code
[*] .python-eggs
[*] Movies
[*] .zshrc
[*] .zsh-update
[*] Desktop
[*] Music
[*] .profile
[*] .fontconfig
[*] .zshrc-e
[*] .geeknote
[*] .subversion
[*] .489454.padl
[*] .zenmap-etc
[*] .Xauthority
[*] .vimrc
[*] Downloads
[*] .config
[*] .cheat
[*] .cache
[*] .
[*] .matplotlib
[*] .rvm
[*] .DS_Store
[*] .sqlmap
[*] .wireshark-etc
[*] .mkshrc
[*] .Trash
[*] .pip
[*] Public
[*] .bash_profile
[*] Documents
[*] .zprofile
[*] .bash_history
[*] Applications
[*] .viminfo
[*] Library
[*] Pictures
[*] .vim
```

## 删除.DS_Store

直接在终端下输入下面命令

```
find / -name ".DS_Store" -delete
```

## 禁止.DS_Store文件生成

在终端下面输入下面命令，就可以了,**注意一定要sudo权限才可以的,改好后要注销用户或者重启系统**。

```
禁止.DS_store生成：
defaults write com.apple.desktopservices DSDontWriteNetworkStores true
恢复.DS_store生成：
defaults delete com.apple.desktopservices DSDontWriteNetworkStores
```

## 参考
1. [Mac OS X v10.4 and later: How to prevent .DS_Store file creation over network connections - Apple 支持](https://support.apple.com/zh-cn/HT1629)
