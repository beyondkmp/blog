---
title: "debian8编译安装python3"
date: 2019-05-13T17:02:38+0800
lastmod: 2019-05-13T17:02:38+0800
draft: false
keywords: ["debian8","python3.7"]
description: "debian8编译安装python3"
tags: ["debian8","python3.7"]
categories: ["运维人生"]
author: "beyondkmp"

---

## 编译安装gcc8
1. 下载[源代码](ftp://ftp.lip6.fr/pub/gcc/releases/gcc-8.3.0/gcc-8.3.0.tar.gz), 并解压。

    ```bash
    wget ftp://ftp.lip6.fr/pub/gcc/releases/gcc-8.3.0/gcc-8.3.0.tar.gz
    tar -xvf gcc-8.3.0.tar.gz
    ```
2. 安装相关依赖
    ```bash
    cd gcc-8.3.0
    ./contrib/download_prerequisites
    sudo apt install texinfo bison flex
    ```

3. 编译

    ```bash
    mkdir build
    cd build
    ../configure --prefix=/usr/local/gcc --enable-bootstrap  --enable-checking=release --enable-languages=c,c++ --disable-multilib
    make -j4 (4是我机器的cpu核心数)
    ```
4. 编译安装并配置环境变量

    ```bash
    sudo make install
    echo "PATH=/usr/local/gcc/bin:$PATH" >> ~/.zshrc
    source ~/.zshrc
    ```
5. 查看版本

    ```bash
    gcc -v
    ```

<!--more-->

## 编译安装python3.7

1. 安装相关依赖

    ```bash
    sudo apt install libssl-dev zlib1g-dev libncurses5-dev libncursesw5-dev libreadline-dev libsqlite3-dev
    sudo apt install libgdbm-dev libdb5.3-dev libbz2-dev libexpat1-dev liblzma-dev libffi-dev
    ```
2. 下载源码并解码

    ```bash
    wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz
    tar -xvf Python-3.7.3.tgz
    ```
3. 编译安装

    ```bash
    export LD_LIBRARY_PATH=/usr/local/gcc/lib/:/usr/local/gcc/lib64:$LD_LIBRARY_PATH
    cd Python-3.7.1
    ./configure --enable-optimizations
    make -j 4
    sudo make altinstall
    ```

## 注意
1. 开启--enable-optimizations时编译会报下面错误`SystemError: <built-in function compile> returned NULL without setting an error`, 主要是gcc版本太低导致的。所以要编译安装gcc8


## 参考

1. [Building Python 3.7 from source on Ubuntu and Debian Linux](https://solarianprogrammer.com/2017/06/30/building-python-ubuntu-wsl-debian/)
2. [Ubuntu16.04 编译安装gcc8.2.0](https://www.wolfoot.com/index.php/archives/9/)
3. [SystemError: <built-in function compile> returned NULL without setting an error](ihttps://bugs.python.org/issue34112)
