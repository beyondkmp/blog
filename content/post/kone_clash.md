---
title: "在树莓派上面kone和clash"
date: 2019-01-18T22:52:13+0800
lastmod: 2019-01-18T22:52:13+0800
draft: false
keywords: ["树莓派","kone","clash"]
description: "在树莓派上面kone和clash"
tags: ["树莓派","kone","clash"]
categories: ["运维人生"]
author: "beyondkmp"

---

<!--more-->

# 安装kone

kone完全基于go语言开发,实现了tun2socks和fakeip功能。fakeip非常方便，可以接管你配置的域名流量.

## 硬件需求

1. 需要一个可以更改静态路由的路由器。

   普通的非智能路由器，如[tp-link](http://service.tp-link.com.cn/detail_article_28.html)之类，都有这个功能。反而智能路由器，如小米、极路由之类没有开放这个功能。不过也没事，智能路由器都是可以刷固件的，我目前刷的padavan，信号和稳定性比原生的系统还要好。

2. 树莓派可以正常使用互联网, 目前我将用网线将树莓派和路由器连接上，并设置静态路由, 一般也可以不用配置静态路由，基本上ip是不会变化的。

	```
    # 在 /etc/network/interfaces,修改eth0配置
    auto eth0
    iface eth0 inet static
        address 192.168.123.2
        netmask 255.255.255.0
        gateway 192.168.123.1
        dns-nameservers 114.114.114.114
	```

## 软件安装

kone官方直接在树莓派上面安装go，通过go来安装kone。不是非常喜欢，主要是太慢了，下载东西也慢。可以直接在我们自己电脑上面进行交叉编译。

先下载kone程序，程序比较老，要放到gopath下面才能编译安装,具体命令如下

```
go get github.com/xjdrew/kone
cd $GOPATH/src/github.com/xjdrew/kone
GOARCH=arm GOOS=linux GOARM=7 CGO_ENABLED=0 go build -ldflags '-w -s'
scp kone pi@192.168.123.2:/tmp
```

## 配置树莓派

* 把树莓派设置成路由模式(需要切换到root用户)

```
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p
```

## 配置并启动kone

* 修改kone的配置文件

在代码目录`[misc/example/example.ini](https://github.com/xjdrew/kone/blob/master/misc/example/example.ini)`，提供了一份默认配置文件。
为了简化问题，只需要把默认配置文件拷贝到合适的目录，命名为`my.ini`，然后把`[proxy "A"]`配置项下的url改成你拥有的代理，目前支持http, socks5代理。

```
[proxy "A"]
url = socks5://127.0.0.1:7891
default = yes
```

其他配置选项的含义，参考配置文件里面的说明,基本不直接用default配置就可以.

* 启动kone

    目前使用supervisor管理，配置文件`/etc/supervisor/conf.d/kone.conf `

    ```
    [program:kone]
	user=root
	command =/home/pi/kone/kone /home/pi/kone/my.ini
	autostart = true
	startsecs = 5
	autorestart = true
	startretries = 1024
	redirect_stderr = true
	stdout_logfile_maxbytes = 10MB
	stdout_logfile_backups = 10
	stdout_logfile = /tmp/kone.log
    ```

## 配置路由器
* 配置静态路由

在路由器上添加一条静态路由：

目的IP地址 | 子网掩码    | 网关           | 状态
---------- | ----------- | -------------- | ----
10.192.0.0 | 255.255.0.0 | 192.168.10.101 | 生效
10.192.0.0 | 255.255.0.0 | 192.168.10.101 | 生效
10.192.0.0 | 255.255.0.0 | 192.168.10.101 | 生效

* 修改默认dns

把路由器的默认dns修改为：10.192.0.1

tp-link 路由器的修改参考这里：<http://service.tp-link.com.cn/detail_article_575.html>

## 测试

断开wifi重新连接，查看dns默认dns是不是10.192.0.1, 并ping www.github.com看下连接地址是不是10.192.x.x

# 安装clash

1. 交叉编译

```
git clone https://github.com/Dreamacro/clash.git
cd clash
GOARCH=arm GOOS=linux GOARM=7 CGO_ENABLED=0 go build -ldflags '-w -s'
scp clash pi@192.168.123.1:/tmp
```

2. 配置clash,supervisor的配置文件, 具体的clash配置可以从网上下载

```
[program:clash]
user=root
command =/home/pi/clash/clash -d /home/pi/clash
autostart = true
startsecs = 5
autorestart = true
startretries = 1024
redirect_stderr = true
stdout_logfile_maxbytes = 10MB
stdout_logfile_backups = 10
stdout_logfile = /var/log/clash.log
```

3. clash不需要开dns的enhance mode,也不用配置iptables. 就像开一个平常的clash. 可以把clash-dashboard下载下来，这样管理起来非常方便，直接<http://192.168.123.2:9090/ui/#/proxies>

## 参考

1. [A rule-based tunnel in Go.](https://github.com/Dreamacro/clash)
2. [在树莓派上使用kone](https://github.com/xjdrew/kone/blob/master/misc/docs/how-to-use-with-raspberry-pi.md)
