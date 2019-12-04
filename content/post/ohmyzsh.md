---
title: "配置高效的zsh"
date: 2019-12-04T14:49:48+0800
lastmod: 2019-12-04T14:49:48+0800
draft: false
keywords: ["zsh","item2","macos"]
description: ""
tags: ["zsh","item2","macos"]
categories: ["运维人生"]
author: "beyondkmp"

---

Zsh是一个非常强大的shell, 拥有更好的自动补全和更加丰富的功能。zsh还有非常多的插件，可以帮助我们更加高效的使用命令行。在这个文章里面，我会介绍如果安装与配置一个高效的zsh环境。

## 安装zsh

各个平台的安装，参考[Installing ZSH](https://github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH), 版本一定要5.1.1以上，如果版本低了，有些功能会不正常的。在centos7一些老的系统上面可能要自己编译安装zsh版本。

<!--more-->

### 编译安装

#### 安装ncurese

zsh的编译安装依赖于ncurses库，如果你有root权限，可以直接安装, 安装ncurses-devel就行。如果没有的话，就要先编译安装ncurese库.

```bash
wget ftp://ftp.gnu.org/gnu/ncurses/ncurses-6.1.tar.gz
tar xvf ncurses-6.1.tar.gz
cd ncurses-6.1
# 如果有root权限，可以安装到/usr/local下面
./configure --prefix=$HOME/local CXXFLAGS="-fPIC" CFLAGS="-fPIC"
make -j && make install
```

编译的时候要带上 `CXXFLAGS="-fPIC" CFLAGS="-fPIC" `这些标志，如果没有加上的话，后面编译安装zsh的时候会遇到很多问题。具体的原因在[这里](https://unix.stackexchange.com/questions/123597/building-zsh-without-admin-priv-no-terminal-handling-library-found)


#### 编译安装zsh

官方的下载地址为[github](https://github.com/zsh-users/zsh) 或者[sourceforge](https://sourceforge.net/projects/zsh/files/latest/download)

```bash
wget -O zsh.tar.xz https://sourceforge.net/projects/zsh/files/latest/download
tar xf zsh.tar.xz
cd zsh
./configure --prefix="$HOME/local" \
    CPPFLAGS="-I$HOME/local/include" \
    LDFLAGS="-L$HOME/local/lib"
make -j && make install
```

如果没有root权限，那就改不了登录的时的默认shell, 一般默认登录是bash，我们可以在bash的配置里面添加配置，登录后自动转到zsh， 具体将下面的配置添加到`.bash_profile`里面就可以。如果有root权限就不用这么做了。

```bash
export PATH=$HOME/local/bin:$PATH
export SHELL=`which zsh`
[ -f "$SHELL" ] && exec "$SHELL" -l
```

## 配置

[Oh-my-zsh](https://ohmyz.sh/)是一个完整的配置框架, 已经包含了非常的功能，也集成了非常多的插件，还可以定制化自己的一些插件。

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```



## 参考
1. [Building Zsh from Source and Configuring It on CentOS](https://jdhao.github.io/2018/10/13/centos_zsh_install_use/)
