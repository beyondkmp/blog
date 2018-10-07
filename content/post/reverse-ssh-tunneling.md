---
title: "反向ssh代理隧道"
date: 2018-08-17T21:52:53+08:00
lastmod: 2018-08-17T21:52:53+08:00
draft: false
keywords: ["ssh","反向","隧道"]
description: "在没有外网的情况下，实现远程登录内网机器"
tags: ["ssh","反向代理"]
categories: ["运维人生"]
author: "beyondkmp"

---

## 反向ssh代理

反向ssh代理主要用来访问nat或者防火墙内的主机，也就是用来穿透nat。比如公司内网的机器，你想在家访问，但是内网的机器没有公网ip。接下来的内容就教你怎么实现在家访问公司内网的机器。**不过这个对公司的内部安全的危害非常大，一般不要乱用，如果使用不当，会把整个公司的内网都暴露给外面。**说白了，这就是一个反弹shell。

<!--more-->

## 步骤

我们要有一台拥有公网ip的机器(现在很方便了，各种云主机就行了)。假设我们这台机器的公网ip是:1.1.1.1,内网机器的ip是192.168.1.28.

1. 使用下面命令，从内网机器连通到公网机器，从而建立一条ssh隧道。19008是内网机器端口，要保证其它程序没在使用。22端口是公网机器的ssh端口，public-username是公网机器的用户名。

    ```bash
    ssh -f -N -T -R 19008:localhost:22 public-username@1.1.1.1
    # -f: tells the SSH to background itself after it authenticates, saving you time by not having to run something on the remote server for the tunnel to remain alive.
    # -N: if all you need is to create a tunnel without running any remote commands then include this option to save resources.
    # -T: useful to disable pseudo-tty allocation, which is fitting if you are not trying to create an interactive shell.
    ```

    ![ssh反向代理隧道](/imgs/ssh-reverse.png)

2. 现在你可以从公网机器连接到内网机器了，使用以下命令,输入内网机器root用户的密码就可以远程操作了

    ```bash
    ssh root@localhost -p 19008
    ```

3. 你还可以在其它内网，先连接到公网机器，再运行步骤2命令，这样就相当于架了一个桥，一个内网的机器可以访问另外一个内网机器。

4. 为了不了第一条命令断开，你可以连接上运行htop或者watch命令，这样就不会断开了。或者你可以直接写一个crontab判断是否断开，如果断开就重新连接

    ```bash
    ps -ef|grep "ssh -f -N -T -R 19008:localhost:22 public-username@1.1.1.1" | grep -v grep -q
    if [[ $? -ne 0  ]];then
        ssh -f -N -T -R 19008:localhost:22 public-username@1.1.1.1
    fi
    # crontab
    #* * * * * /usr/local/bin/ssh-reverse.sh &
    ```

## 参考

1. [What is Reverse SSH Port Forwarding](https://blog.devolutions.net/2017/03/what-is-reverse-ssh-port-forwarding)
2. [Reverse SSH Tunneling](https://www.howtoforge.com/reverse-ssh-tunneling)
