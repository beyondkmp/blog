---
title: "libfaketime改变软件的运行时间"
date: 2019-01-27T16:40:07+0800
lastmod: 2019-02-25T14:32:25+0800
draft: false
keywords: ["libfaketime","surge"]
description: "libfaketime改变软件的运行时间"
tags: ["libfaketime","surge"]
categories: ["mac"]
author: "beyondkmp"

---

## libfaketime
[libfaketime](https://github.com/wolfcw/libfaketime):主要是拦截了程序调用获取当前时间的系统调用。然后会将你修改(假的)后的时间返回给这些程序。这样的话你可以单独修改一个程序的获取的时间，而不用修改整个系统的时间。

## 安装libfaketime

有两种方法安装，brew和编译源码安装. brew安装的版本有点老，在最新的macos系统已经不行了,运行时会报错。

1. 通过brew安装

    ```
    brew install libfaketime coreutils
    ```
2. 源码编译安装

    ```
    git clone https://github.com/wolfcw/libfaketime.git
    make
    sudo make install
    ```

## 使用faketime来运行surge
<!--more-->

### 下载surge

前提是surge在你电脑上面没有过试用期，过了试用期就没有办法了. 已经有了正版的序列号的也不要用这种方法，可能会被拉黑了.

先下载surge, 目前只能下载`Surge-2.5.1-528.zip`这个版本及以前版本，不能从官网下载，官网的所有版本都更新了相应的机制来预防libfaketime。

### 安装surge并运行

1. 先打开试用，更新配置，关闭软件的自动更新。正常使用后，退出软件。
2. 使用下面命令，通过faketime启动surge

    ```
    $ faketime '@2007-01-01 00:00:00' /Applications/Surge.app/Contents/MacOS/Surge &
    ```

    不过如果直接用上面的命令的运行，faketime的cpu占用非常大,让电脑一直发热。我们设置`FAKETIME_STOP_AFTER_SECONDS=10`, 就让它运行10s后自动关闭，因为软件只是在启动的时候检测一下时间，运行后退出libfaketime节省cpu。


    ```
    $ FAKETIME_STOP_AFTER_SECONDS=10 faketime -f '@2007-01-01 00:00:00' /Applications/Surge.app/Contents/MacOS/Surge &
    ```

3. 关闭终端

## Alfred启动

每次都要到终端打开有点太麻烦了，如果经常用alfred的，可以添加一个keywords的workflow, 将上面的命令放到里面。然后运行`fakesurge`,效果如下：

![fakesurge](/imgs/alfred_surge.png)

我已经把这个workflow导出来了，直接下载[fakesurge](/files/surge_faketime.alfredworkflow), 双击安装即可。
