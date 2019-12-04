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

oh-my-zsh自带了非常多的插件用来提高我们使用zsh的效率，可以在.zshrc的通过下面配置开启对应的插件

```bash
plugins=(z git zsh-autosuggestions zsh-syntax-highlighting ruby python gem pip go)
```

### 安装第三方插件

1. 安装`zsh-autosuggestions`,直接git下载，并在.zshrc的plugins里面加下`zsh-autosuggestions`。此插件可以自动补全提示，从历史命令从获取的。具体效果如下图

    ![]()

    ```bash
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
    ```

2. 安装`zsh-syntax-highlighting`, 安装方法如1一样。此插件可以高亮命令，看起来更加的舒服。

    ```bash
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
    ```

### 配置主题

#### 安装Powerlevel9k

```bash
git clone https://github.com/bhilburn/powerlevel9k.git ~/.oh-my-zsh/custom/themes/powerlevel9k
```

在.zshrc里面启用此主题，并进行定制，配置如下:

```zsh
ZSH_THEME="powerlevel9k/powerlevel9k"
POWERLEVEL9K_MODE="awesome-fontconfig"
POWERLEVEL9K_SHORTEN_DIR_LENGTH=3
#POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(status context dir vcs)
POWERLEVEL9K_CUSTOM_NAME="echo dev"
POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(status custom_name dir dir_writable vcs)
POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=()
POWERLEVEL9K_STATUS_VERBOSE=false
POWERLEVEL9K_MODE='awesome-fontconfig'
POWERLEVEL9K_DIR_OMIT_FIRST_CHARACTER=true
POWERLEVEL9K_DIR_PATH_SEPARATOR=$' \uE0B1 '
POWERLEVEL9K_MODE='nerdfont-complete'
```

#### 安装nerd font

1. 安装hack nerd font

    [Nerd Fonts](https://github.com/ryanoasis/nerd-fonts)拥有很多编程字体并且自带了非常多符号，美观且实用。下面是mac osx的安装方法。

    ```bash
    brew tap homebrew/cask-fonts
    brew cask install font-hack-nerd-font
    ```

2. 安装monaco font patched with extra nerd glyphs, 在[monaco-nerd-fonts](https://github.com/beyondkmp/monaco-nerd-fonts)下载字体，然后双击安装就完成了。

3. 安装完成后，要对`iterm2`进行字体修改

    ```bash
    iTerm2配置 –> Profiles –> Text –> Use a different font for non-ASCII Text
    将Font设置为 Monaco Nerd Font
    将Non-ASCII Font设置为 Hack Nerd Font
    ```
4. 配合powerlevel9k主题，添加下面的配置

    ```bash
    POWERLEVEL9K_MODE='nerdfont-complete'
    ```

## 参考
1. [Building Zsh from Source and Configuring It on CentOS](https://jdhao.github.io/2018/10/13/centos_zsh_install_use/)
