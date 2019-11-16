---
title: "Tmux配置"
date: 2019-09-24T14:31:07Z
lastmod: 2019-09-24T14:31:07Z
draft: false
keywords: ["tmux","powerline"]
description: "个人常用的tmux配置"
tags: ["tmux","powerline"]
categories: ["运维人生"]
author: "beyondkmp"

---

# 编译安装

## 安装依赖

```shell
apt-get install libevent-dev
apt-get install libcurses-ocaml-dev
```

```shell
$ wget  https://github.com/tmux/tmux/releases/download/2.9a/tmux-2.9a.tar.gz
$ tar -xvf tmux-2.9a.tar.gz
$ cd tmux-2.9a
$ ./configure
$ make
$ make install
```

<!--more-->

# install powerline

tmux-powerline已经不维护了，所以转为使用powerline.

## 安装

1. 安装pip源的版本，稳定版本

    ```shell
    pip install powerline-status
    ```
2. 安装github上面的最新版本

    ```shell
    pip install --user git+git://github.com/powerline/powerline
    ```

## 查看powerline的安装位置

```shell
$ pip3 show powerline-status
Name: powerline-status
Version: 2.7
Summary: The ultimate statusline/prompt utility.
Home-page: https://github.com/powerline/powerline
Author: Kim Silkebaekken
Author-email: kim.silkebaekken+vim@gmail.com
License: MIT
Location: /usr/local/lib/python3.7/site-packages
Requires:
Required-by:
```

如果是安装在用户目录的.local目录下面，要将下面的目录加入到PATH的环境变量中

```shell
export PATH=$PATH:$HOME/.local/bin
```

# 安装nerd-font字体

```shell
brew tap homebrew/cask-fonts
brew cask install font-hack-nerd-font
```

**修改iTerm字体**

iTerm2配置 --> Profiles --> Text --> Use a different font for non-ASCII Text

将Non-ASCII Font设置为 Knack Nerd Font

## Powerline 配置

安装完powerline后，找到对应的poweline.conf位置, 替换掉下面的配置

```shell
if-shell 'test -f ~/.local/lib/python2.7/site-packages/powerline/bindings/tmux/powerline.conf' 'source-file ~/.local/lib/python2.7/site-packages/powerline/bindings/tmux/powerline.conf'
```

# 配置文档

```shell
#设置PREFIX为Ctrl-a
set -g prefix C-Space
#解除Ctrl-b与PREFIX的对应关系
unbind C-b
#copy-mode将快捷键设置为vi模式
#setw -g mode-keys vi
# Use vim keybindings in copy mode

# Setup 'v' to begin selection as in Vim
# bind-key -t vi-copy v begin-selection
# bind-key -t vi-copy y copy-pipe "reattach-to-user-namespace pbcopy"

# Update default binding of Enter to also use copy-pipe
# unbind -t vi-copy Enter

set-window-option -g mode-keys vi

# vi-style copying
bind-key -T copy-mode-vi 'v' send -X begin-selection
bind-key -T copy-mode-vi 'y' send -X copy-selection-and-cancel


#将r键设置为加载配置文件，并显示"reloaded!"信息
bind r source-file ~/.tmux.conf \; display "Reloaded!"
#设置终端颜色为256色
set -g default-terminal "screen-256color"
#开启status-bar uft-8支持
#set -g status-utf8 on

# 开启鼠标模式
set-option -g mouse on
# 水平分割面板
unbind '"'
bind - splitw -v

# 垂直分割面板
unbind %
bind | splitw -h

# 绑定上j下k左l右h来方便在面板中切换
bind k selectp -U
bind j selectp -D
bind h selectp -L
bind l selectp -R

if-shell 'test -f ~/.local/lib/python2.7/site-packages/powerline/bindings/tmux/powerline.conf' 'source-file ~/.local/lib/python2.7/site-packages/powerline/bindings/tmux/powerline.conf'
```

# 参考

1. [Example tmux configuration](https://tony.github.io/tmux-config/)
2. [让命令行更炫酷](https://al03.github.io/%E8%AE%A9%E5%91%BD%E4%BB%A4%E8%A1%8C%E6%9B%B4%E7%82%AB%E9%85%B7/)
