---
title: "iterm2使用szrz"
date: 2016-05-21T00:00:00+0800
lastmod: 2019-11-24T23:59:58+0800
draft: false
keywords: ["macos","iterm2","szrz"]
tags:  ["macos","iterm2","szrz"]
description: "经常要通过终端从服务器上传或下载文件,使用rzsz会非常方便"
categories: ["运维人生"]
---

## 安装步骤

1. 安装lrzsz软件

    ```bash
    brew install lrzsz
    ```
2. 将下面的iterm2-zmodem-recv和iterm2-zmodem-send脚本保存到/usr/local/bin下面,并添加可执行权限(`chmod +x /usr/local/bin/iterm2-zmodem-*`)

    * iterm2-zmodem-recv

        ```bash
        #!/usr/bin/env bash
        # vim: fdm=marker foldlevel=0 sw=2 ts=2 sts=2

        FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose folder with prompt "Choose a folder to place received files in"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`

        if [[ -z "$FILE" ]]; then
          echo " Cancelled"
          # Send ZModem cancel
          echo -e \\x18\\x18\\x18\\x18\\x18
          sleep 1
          echo
          echo " # Cancelled transfer"
        else
          cd "$FILE"
          "${HOMEBREW_PREFIX:-/usr/local}/bin/rz" -E -e -b
          sleep 1
          echo
          echo " # Sent -> $FILE"
        fi

        ```
    * iterm2-zmodem-send

        ```bash
        #!/usr/bin/env bash
        # vim: fdm=marker foldlevel=0 sw=2 ts=2 sts=2

        FILE=`osascript -e 'tell application "iTerm" to activate' -e 'tell application "iTerm" to set thefile to choose file with prompt "Choose a file to send"' -e "do shell script (\"echo \"&(quoted form of POSIX path of thefile as Unicode text)&\"\")"`

        if [[ -z "$FILE" ]]; then
          echo " Cancelled"
          # Send ZModem cancel
          echo -e \\x18\\x18\\x18\\x18\\x18
          sleep 1
          echo
          echo " # Cancelled transfer"
        else
          "${HOMEBREW_PREFIX:-/usr/local}/bin/sz" "$FILE" -e -b
          sleep 1
          echo
          echo " # Received $FILE"
        fi
        ```

3. 在iTerm 2下面设置相应的Triggers,Triggers位于`Preferences->Profiles->Advanced->Triggers`,直接edit就会出现下面的界面:

    ![triggers](/imgs/osx-szrz使用/1.png)


    ```
    具体的内容如下:
    Regular expression: rz waiting to receive.\*\*B0100
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-zmodem-send

    Regular expression: \*\*B00000000000000
    Action: Run Silent Coprocess
    Parameters: /usr/local/bin/iterm2-zmodem-recv
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

