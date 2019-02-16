---
title: "在mac os x 下面使用szrz"
date: 2016-05-21T00:00:00+0800
lastmod: 2016-05-21T00:00:00+0800
draft: false
keywords: ["os x","szrz"]
tags:  ["os x","szrz"]
description: "经常要通过终端从服务器上传或下载文件,使用此命令可以方便很多"
categories: ["常用技巧"]
tags: ["os x","szrz"]
---

## 安装步骤

1. 安装lrzsz软件

    ```
    brew install lrzsz
    ```
2. 将<https://github.com/mmastrac/iterm2-zmodem>下载的 iterm2-send-zmodem.sh and iterm2-recv-zmodem.sh脚本保存到/usr/local/bin下面,并添加可执行权限

3. 在iTerm 2下面设置相应的Triggers,Triggers位于`Preferences->Profiles->Advanced->Triggers`,直接edit就会出现下面的界面:

    ![triggers]({{IMAGE_PATH}}/osx-szrz使用/1.png)


    ```
    具体的内容如下:
    Regular expression: rz waiting to receive.\*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-send-zmodem.sh

    Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
    ```

## 使用方法

**发送文件到服务器:**

1. 在服务器上运行"rz"命令
2. 从本地弹出的窗口中选择要上传的文件
3. 等待传输完成

**从服务下载文件到本地**

1. 在服务器上运行"sz filename1 filename2 ... filenameN"
2. 从本地弹出的窗口中选择存放的位置
3. 等待传输完成

## 常用的使用技巧

1. 要上传二进制文件,直接用`rz -be`和`sz -be`,如果不加-be参数,会直接卡死.因为rzsz默认传输的ascii文件.
2. 要覆盖上传下载,用命令`rz -y`和`sz -y`,默认是如果存在相同文件名的文件,是不覆盖的.

