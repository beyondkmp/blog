---
title: "在树莓派上使用kone和clash"
date: 2019-01-18T22:52:13+0800
lastmod: 2019-01-18T22:52:13+0800
draft: false
keywords: ["树莓派","kone","clash"]
description: "在树莓派上使用kone和clash"
tags: ["树莓派","kone","clash"]
categories: ["运维人生"]
author: "beyondkmp"

---


## 安装kone

kone完全基于go语言开发,实现了tun2socks和fakeip功能。fakeip非常方便，可以接管你配置的域名流量.

### 硬件需求

* 需要一个可以更改静态路由的路由器。

    普通的非智能路由器，如[tp-link](http://service.tp-link.com.cn/detail_article_28.html)之类，都有这个功能,所以普通路由器就可以实现了。反而智能路由器，如小米、极路由之类没有开放这个功能。不过也没事，智能路由器都是可以刷固件的，我目前刷的padavan，信号和稳定性比原生的系统还要好。

<!--more-->

* 树莓派可以正常使用互联网


### 安装kone

kone官方直接在树莓派上面安装go，通过go来安装kone。不是非常喜欢，主要是太慢了，下载东西也慢。可以直接在我们自己电脑上面进行交叉编译。

```
go get github.com/xjdrew/kone
cd $GOPATH/src/github.com/xjdrew/kone
GOARCH=arm GOOS=linux GOARM=7 CGO_ENABLED=0 go build -ldflags '-w -s'
#上传到pi上
scp kone pi@192.168.123.2:/tmp
```

### 配置树莓派

* 把树莓派设置成路由模式(需要切换到root用户)

    ```
    echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
    sysctl -p
    ```
* 目前我将用网线将树莓派和路由器连接上，并设置静态路由, 一般也可以不用配置静态路由，基本上ip是不会变化的。

	```
    # 在 /etc/network/interfaces,修改eth0配置
    auto eth0
    iface eth0 inet static
        address 192.168.123.2
        netmask 255.255.255.0
        gateway 192.168.123.1
        dns-nameservers 114.114.114.114
	```

### 管理并启动kone

* 修改kone的配置文件

    在代码目录[misc/example/example.ini](https://github.com/xjdrew/kone/blob/master/misc/example/example.ini)，提供了一份默认配置文件。
    为了简化问题，只需要把默认配置文件拷贝到合适的目录，命名为`my.ini`，然后把`[proxy "A"]`配置项下的url改成你拥有的代理，目前支持http, socks5代理。

    ```
    [proxy "A"]
    url = socks5://127.0.0.1:7891
    default = yes
    ```

    其他配置选项的含义，参考配置文件里面的说明,直接用default配置就可以.

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

    配置完成后`supervisorctl update`下就可以。

### 配置路由器

* 配置静态路由

    在路由器上添加多条静态路由：

    ![static route](/imgs/static_route.png)

* 修改默认dns

    把路由器的默认dns修改为：10.192.0.1

    ![dns](/imgs/dns.png)

    tp-link 路由器的修改参考这里：<http://service.tp-link.com.cn/detail_article_575.html>

### 测试

断开wifi重新连接，查看dns默认dns是不是10.192.0.1, 并ping www.github.com看下连接地址是不是10.192.x.x

```
 $ ping www.github.com
 PING www.github.com (10.192.47.177) 56(84) bytes of data.
 64 bytes from 10.192.47.177 (10.192.47.177): icmp_seq=1 ttl=63 time=1.70 ms
 64 bytes from 10.192.47.177 (10.192.47.177): icmp_seq=2 ttl=63 time=1.96 ms
 64 bytes from 10.192.47.177 (10.192.47.177): icmp_seq=3 ttl=63 time=1.82 ms
 64 bytes from 10.192.47.177 (10.192.47.177): icmp_seq=4 ttl=63 time=2.05 ms
 64 bytes from 10.192.47.177 (10.192.47.177): icmp_seq=5 ttl=63 time=1.98 ms
 64 bytes from 10.192.47.177 (10.192.47.177): icmp_seq=6 ttl=63 time=2.81 ms
 64 bytes from 10.192.47.177 (10.192.47.177): icmp_seq=7 ttl=63 time=2.23 ms
 ^C
 --- www.github.com ping statistics ---
 7 packets transmitted, 7 received, 0% packet loss, time 18ms
 rtt min/avg/max/mdev = 1.696/2.078/2.813/0.342 ms

```

## 安装clash

###  交叉编译

具体的命令如下:

```
git clone https://github.com/Dreamacro/clash.git
cd clash
GOARCH=arm GOOS=linux GOARM=7 CGO_ENABLED=0 go build -ldflags '-w -s'
scp clash pi@192.168.123.1:/tmp
```

### 管理并启动clash

配置clash, 具体的clash配置可以从网上下载. supervisor的配置文件`/etc/supervisor/conf.d/clash.conf`

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

### clash配置说明

1. clash不需要开dns的enhance mode
2. 也不用配置iptables导流
3. 下载clash-dashboard，放到配置文件夹中，直接使用<http://192.168.123.2:9090/ui/#/proxies>管理.


## 参考

1. [A rule-based tunnel in Go.](https://github.com/Dreamacro/clash)
2. [在树莓派上使用kone](https://github.com/xjdrew/kone/blob/master/misc/docs/how-to-use-with-raspberry-pi.md)
