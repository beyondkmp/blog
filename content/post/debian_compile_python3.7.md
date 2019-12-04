---
title: "debian编译安装python3"
date: 2019-05-13T17:02:38+0800
lastmod: 2019-12-04T13:17:32+0800
draft: false
keywords: ["debian","python3.7","编译安装"]
description: "debian8编译安装python3"
tags: ["debian8","python3.7","编译安装"]
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
    #4是我机器的cpu核心数
    make -j4
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
    sudo apt update
    sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget
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
    # 不会直接覆盖原来的python3版本
    sudo make altinstall
    ```

## 注意
1. 开启--enable-optimizations时编译会报下面错误`SystemError: <built-in function compile> returned NULL without setting an error`, 如果是debian8的话, 由于gcc版本太低导致的。所以要编译安装gcc8
2. pip安装软件时报错`ModuleNotFoundError: No module named '_ctypes'`, 这个问题主要是要安装libffi-dev依赖，如果没有安装的话，要先安装，再重新编译安装一次python.


## 参考

1. [Building Python 3.7 from source on Ubuntu and Debian Linux](https://solarianprogrammer.com/2017/06/30/building-python-ubuntu-wsl-debian/)
2. [Ubuntu16.04 编译安装gcc8.2.0](https://www.wolfoot.com/index.php/archives/9/)
3. [SystemError: <built-in function compile> returned NULL without setting an error](ihttps://bugs.python.org/issue34112)
4. [How to Install Python 3.7 on Debian 9](https://linuxize.com/post/how-to-install-python-3-7-on-debian-9/)
