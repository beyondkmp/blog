---
title: "通过网络启动安装ubuntu"
date: 2020-05-13T17:37:05+0800
lastmod: 2020-05-13T17:37:05+0800
draft: false
keywords: ["pxe","netboot","ubuntu","linux"]
description: ""
tags: ["pxe","netboot","ubuntu","linux"]
categories: ["运维人生"]
author: "beyondkmp"

---

最近折腾nuc,默认系统是win10。一直想安装一个linux，结果开始动手安装后，真的是累啊。之前安装linux都是直接使用dd命令制作一个启动u盘，然后bios设置成u盘启动，就可以点下一步就可以安装完成了，不过这次nuc不知道为什么就是启动不成功，一直报"unable to find a medium containing a live file system".

网上有很多方法，没有一个有用的。改过bios，不使用uefi启动，系统启动盘也用其它的软件制作过，还有更加牛逼的方法：就是进入到ubuntu界面，拨出u盘再插上。我怀疑是我的usb3.0u盘的问题，不过手头没有2.0的u盘也没有办法证实了。

nuc又没有光驱，现在剩下的唯一方法就是pxe通过网络启动安装了，于是有了下面的pxe安装教程。

## 在win10创建空白压缩盘

在Windows10下直接进入磁盘管理器找到有足够空间的磁盘，通过压缩卷压缩出一部分空白分区，分区大小依据个人需求，这个大小是未来第二个系统所有的可用空间，直接给了个30G

<!--more-->

## 树莓派设置

### 下载ubuntu启动文件

1. 下载UEFI签名的grup启动文件到/srv/tftp. <http://archive.ubuntu.com/ubuntu/dists/trusty/main/uefi/grub2-amd64/current/grubnetx64.efi.signed>

2. 下载正确的ubtuntu的网络启动iso，这里我选择的是18.04的服务器版本。<http://cdimage.ubuntu.com/netboot/>

3. 解压 netboot.tar.gz 到/srv/tftp/

### 设置grub

1. 先创建文件`/srv/tftp/grub/grub.cfg`
2. 将下面的内容复制到grub.cfg里面

    ```bash
    menuentry "Install Ubuntu" {
    set gfxpayload=keep
    linux /ubuntu-installer/amd64/linux gfxpayload=800x600x16,800x600 live-installer/net-image=$PATH_TO_FILESYSTEM_SQUASHFS --- quiet
    initrd /ubuntu-installer/amd64/initrd.gz
    }
    ```

### 安装tftp和dhcp服务器

1. Install dnsmasq:

    sudo apt-get install dnsmasq

2. 将电脑设置成静态ip(一般路由器下面也不会变ip可以不用设置)

3.将下面的内容复制到`/etc/dnsmasq.conf`

        ```bash
        listen-address=127.0.0.1
        listen-address=192.168.50.2
        # DHCP options
        dhcp-range=192.168.50.100,192.168.50.249,12h
        dhcp-lease-max=100
        dhcp-option=option:router,192.168.50.1
        dhcp-option=option:dns-server,192.168.50.2
        dhcp-option=option:netmask,255.255.255.0

        dhcp-boot=grubnetx64.efi.signed
        enable-tftp
        tftp-root=/srv/tftp/
        ```

4. dnsmasq reload

    ```bash
    sudo service dnsmasq restart
    ```

## 设置路由器

1. 将网关设置路由器192.168.50.2，这样当nuc网络启动的时候会把树莓派当成网关，就可以网络启动安装了。
2. 因为路由器把网关设置成192.168.50.2, 这样在树莓上面的默认路由会指向自己，是不正确的，用下面的命令删除掉

    ```bash
    ip route delete default via 198.168.50.2
    ip route add default via 198.168.50.2
    ```

## 安装

1. 重启nuc, 将按F10,选择pxe启动
2. 进入界面进行安装


## 参考

1. [PXE-netboot-install](https://wiki.ubuntu.com/UEFI/PXE-netboot-install)
2. [Configuring PXE Network Boot Server on Ubuntu 18.04 LTS](https://linuxhint.com/pxe_boot_ubuntu_server/)
