---
title: "clash与树莓派组成旁路由"
date: 2019-01-18T22:52:13+0800
lastmod: 2020-05-13T19:13:46+0800
draft: false
keywords: ["树莓派","网关","fake-ip", "旁路由","clash"]
description: "使用clash的tun模式，将树莓派变成强大的旁路由"
tags: ["树莓派","网关","fake-ip", "旁路由","clash"]
categories: ["运维人生"]
author: "beyondkmp"

---


## 硬件需求

* 需要一个可以更改静态路由的路由器。

    普通的非智能路由器，如[tp-link](http://service.tp-link.com.cn/detail_article_28.html)之类，都有这个功能,所以普通路由器就可以实现了,不需要高大上的智能路由器。反而智能路由器，如小米、极路由之类没有开放这个功能。不过也没事，智能路由器都是可以刷固件的，我目前刷的padavan，信号和稳定性比原生的系统还要好。

* 树莓派可以正常使用互联网

<!--more-->

### 配置树莓派

* 把树莓派设置成路由模式(需要切换到root用户)

    ```bash
    echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
    sysctl -p
    ```
* 目前我将用网线将树莓派和路由器连接上，并设置静态路由, 一般也可以不用配置静态路由，基本上ip是不会变化的。

	```bash
    # 在 /etc/network/interfaces,修改eth0配置
    auto eth0
    iface eth0 inet static
        address 192.168.123.2
        netmask 255.255.255.0
        gateway 192.168.123.1
        dns-nameservers 114.114.114.114
	```

### 配置路由器

* 配置静态路由

    在路由器上添加多条静态路由：

    ![static route](/imgs/static_route.png)

* 修改默认dns

    把路由器的默认dns修改为：192.168.123.2

    ![dns](/imgs/dns.png)

    tp-link 路由器的修改参考这里：<http://service.tp-link.com.cn/detail_article_575.html>

* 保存配置后，重启路由器

### 测试

断开wifi重新连接，查看dns默认dns是不是192.168.123.2, 并`ping www.github.com`看下连接地址是不是192.18.x.x, 或者使用dig命令。

```bash
 $ ping www.github.com
 PING www.github.com (192.168.123.177) 56(84) bytes of data.
 64 bytes from 192.168.123.177 (192.168.123.177): icmp_seq=1 ttl=63 time=1.70 ms
 64 bytes from 192.168.123.177 (192.168.123.177): icmp_seq=2 ttl=63 time=1.96 ms
 64 bytes from 192.168.123.177 (192.168.123.177): icmp_seq=3 ttl=63 time=1.82 ms
 64 bytes from 192.168.123.177 (192.168.123.177): icmp_seq=4 ttl=63 time=2.05 ms
 64 bytes from 192.168.123.177 (192.168.123.177): icmp_seq=5 ttl=63 time=1.98 ms
 64 bytes from 192.168.123.177 (192.168.123.177): icmp_seq=6 ttl=63 time=2.81 ms
 64 bytes from 192.168.123.177 (192.168.123.177): icmp_seq=7 ttl=63 time=2.23 ms
 ^C
 --- www.github.com ping statistics ---
 7 packets transmitted, 7 received, 0% packet loss, time 18ms
 rtt min/avg/max/mdev = 1.696/2.078/2.813/0.342 ms

```

## 安装clash

clash完全基于go语言开发,实现了完善的规则分流，自动和主备等多种选路策略。

到github下载对应的二进制可运行文件,<https://github.com/Dreamacro/clash/releases/tag/premium>,树莓派对应的clash-linux-armv7-2020.05.08.gz

### clash配置

```bash
port: 7890
socks-port: 7891
#redir-port: 7892

allow-lan: true
mode: Rule
log-level: info
external-controller: :80
external-ui: dashboard
secret: "xxxx"

dns:
    enable: true
    listen: :53
    enhanced-mode: fake-ip
    ipv6: false
    nameserver:
      - "192.168.123.1"
      - "119.29.29.29"
      - "114.114.114.114"
      - "223.5.5.5"

experimental:
  interface-name: eth0
tun:
  enable: true
  stack: system

```

### 配置路由

如果没有通过域名导流，而是ip的静态路由导到树莓派，这样是不会到tun网卡上来的，现在需要做的就是要氢所有ip类型的通过添加路由导到tun上面。

1. 使用netmask计算子网

    ```bash
    pi@raspberrypi:~/clash $ netmask 192.168.123.3:192.168.123.255
      192.168.123.3/32
      192.168.123.4/30
      192.168.123.8/29
     192.168.123.16/28
     192.168.123.32/27
     192.168.123.64/26
    192.168.123.128/25
    ```
2. 添加路由

    ```bash
    ip rule add from 192.168.123.3/32 table 100
    ip rule add from 192.168.123.4/30 table 101
    ip rule add from 192.168.123.8/29 table 102
    ip rule add from 192.168.123.16/28 table 103
    ip rule add from 192.168.123.32/27 table 104
    ip rule add from 192.168.123.64/26 table 105
    ip rule add from 192.168.123.128/25 table 106

    ip route add default via 198.18.0.1 dev utun table 100
    ip route add default via 198.18.0.1 dev utun table 101
    ip route add default via 198.18.0.1 dev utun table 102
    ip route add default via 198.18.0.1 dev utun table 103
    ip route add default via 198.18.0.1 dev utun table 104
    ip route add default via 198.18.0.1 dev utun table 105
    ip route add default via 198.18.0.1 dev utun table 106
    ```

3. 如果需要删除使用下面的命令

    ```bash
    ip rule del from 192.168.123.3/32 table 100
    ip rule del from 192.168.123.4/30 table 101
    ip rule del from 192.168.123.8/29 table 102
    ip rule del from 192.168.123.16/28 table 103
    ip rule del from 192.168.123.32/27 table 104
    ip rule del from 192.168.123.64/26 table 105
    ip rule del from 192.168.123.128/25 table 106
    ```

### 使用supervisor管理

```bash
[group:clash]
programs=main,add-rule

[program:main]
user=root
command =/home/pi/clash/clash -d /home/pi/clash
priority=1
autostart = true
startsecs = 5
autorestart = true
startretries = 1024
redirect_stderr = true
stdout_logfile_maxbytes = 10MB
stdout_logfile_backups = 10
stdout_logfile = /var/log/clash.log

[program:add-rule]
user=root
command = /home/pi/clash/clash.sh
#command = /home/pi/clash/redir/set-redir-iptables.sh
priority=2
startsecs = 0
autorestart = false
startretries = 1
```

```bash
#!/bin/bash
# clash.sh

for (( ; ; ))
do
    sleep 3
    ip address | grep 198.18.0.1
    a=$?
    if [ ${a} -eq 0 ];then
        break;
    fi
done

ip rule del from 192.168.123.3/32 table 100
ip rule del from 192.168.123.4/30 table 101
ip rule del from 192.168.123.8/29 table 102
ip rule del from 192.168.123.16/28 table 103
ip rule del from 192.168.123.32/27 table 104
ip rule del from 192.168.123.64/26 table 105
ip rule del from 192.168.123.128/25 table 106


ip rule add from 192.168.123.3/32 table 100
ip rule add from 192.168.123.4/30 table 101
ip rule add from 192.168.123.8/29 table 102
ip rule add from 192.168.123.16/28 table 103
ip rule add from 192.168.123.32/27 table 104
ip rule add from 192.168.123.64/26 table 105
ip rule add from 192.168.123.128/25 table 106

ip route add default via 198.18.0.1 dev utun table 100
ip route add default via 198.18.0.1 dev utun table 101
ip route add default via 198.18.0.1 dev utun table 102
ip route add default via 198.18.0.1 dev utun table 103
ip route add default via 198.18.0.1 dev utun table 104
ip route add default via 198.18.0.1 dev utun table 105
ip route add default via 198.18.0.1 dev utun table 106
```

## 参考

1. [A rule-based tunnel in Go.](https://github.com/Dreamacro/clash)
2. [在树莓派上使用kone](https://github.com/xjdrew/kone/blob/master/misc/docs/how-to-use-with-raspberry-pi.md)
