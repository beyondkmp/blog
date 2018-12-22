---
title: "macos上的神级输入法squirrel"
date: 2018-12-22T17:10:35+08:00
lastmod: 2018-12-22T17:10:35+08:00
draft: false
keywords: ["macos输入法","五笔","squirrel"]
description: "macos上的神级输入法"
tags: ["macos输入法","五笔","squirrel"]
categories: ["mac"]
author: "beyondkmp"

---
squirrel输入法真是极客最想要的输入法，比起所谓的国产输入法，都在忙着收集用户的输入数据，却没有把精力投入到输入法的核心功能。

squirrel目前的优势：

1. 配置相当丰富，可定制性，可玩性都非常高
2. 输入速度是非常快
3. 五笔支持的非常好
4. 不收集用户数据，这是我最看重的。

<!--more-->

## 编译安装

目前如果要安装squirrel最新版本是要自己编译安装。具体的编译过程如下:

``` sh
#安装Xcode的工具,运行下面的命令
xcode-select --install

# 使用 [Homebrew](http://brew.sh/)安装相差依赖库:
# dev tools:
brew install cmake
brew install git
# libraries:
brew install boost

#下载源代码
git clone --recursive https://github.com/rime/squirrel.git

#编译依赖库
make deps

#编译squirrel
make

#安装squirrel
sudo make install
```

## 参考

1. [How to Rime with Squirrel](https://github.com/rime/squirrel/blob/master/INSTALL.md)
2. [「鼠须管」的调教笔记](https://scomper.me/gtd/-shu-xu-guan-de-diao-jiao-bi-ji)
3. [我的鼠须管配置](https://placeless.net/blog/my-rime-squirrel-config)
4. [安装及配置 Mac 上的 Rime 输入法——鼠鬚管 (Squirrel)](https://placeless.net/blog/my-rime-squirrel-config)
