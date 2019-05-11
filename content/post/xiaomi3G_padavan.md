---
title: "小米路由器3G安装padavan固件"
date: 2018-10-23T20:27:39+08:00
lastmod: 2019-05-11T18:57:17+0800
draft: false
keywords: ["小米路由器3G","padavan固件"]
description: "在小米路由器3G上面安装padavan固件"
tags: ["小米路由器","padavan固件"]
categories: ["运维人生"]
author: "beyondkmp"

---

# 获取路由器ssh权限

从京东下一台小米3G路由器，很快就送到了。接下来就要搞事情了...

## 工具准备

1. mac电脑一台
2. u盘一个，格式为FAT或者FAT32
3. mac的usb网络转接口一个

<!--more-->

## 升级开发版本系统

下载地址：<http://www.miwifi.com/miwifi_download.html>, 具体如下图:

![小米rom R3G开发版本](/imgs/rom_r3g_dev.png)

下载完成，进入路由器后台，点击用户下面系统升级，然后再点击手动升级。上传刚才下载的开发版本固件，等待5-10分钟就升级完成了。

## 获取ssh权限

1. 先要将路由器绑定小米账号，手机连接小米路由器，下载一个*小米WIFI*软件，然后登录小米账号进行对路由器绑定。
2. 绑定完成后，登录以下网站<http://www.miwifi.com/miwifi_open.html>,获取ssh工具包
    ![ssh工具包下载地址](/imgs/ssh_xiaomi.png)
    ![ssh工具包的密码](/imgs/ssh_pass_xiaomi.png)
3. 将下载的miwifi_ssh.bin文件拷贝到u盘的根目录下面，名字一定要是miwifi_ssh.bin, 最好再验证下md5。
4. 断开小米路由器的电源，将U盘插入USB接口； 按住reset按钮之后重新接入电源，指示灯变为黄色闪烁状态即可松开reset键; 等待3-5秒后安装完成之后，小米路由器会自动重启
5. 重启完成后，`ssh root@192.168.31.1`,输入ssh密码,这个密码在下载ssh的bin文件的页面是有的，如下图，当你看到`a you ok`时，说明已经获取ssh权限了。

## 刷入不死BREED

1. 下载breed-mt7621-xiaomi-r3g.bin系统,[官网下载](https://breed.hackpascal.net/)
2. 在终端下面运行`scp /path/breed-mt7621-xiaomi-r3g.bin root@192.168.31.1:/tmp`，将bin文件传送到路由器中
3. 在ssh连接上路由器, 运行下面命令进行md5验证下`md5sum breed-mt7621-xiaomi-r3g.bin`, 原文件的md5为`6c6965afc478d2ffab2cf5caa80a92b9`
4. 如果不需要备份，则可以直接干了。先给breed的文件取个简短的名字,`cd /tmp;cp breed-mt7621-xiaomi-r3g.bin breed.bin`, 然后运行下面的命令`mtd -r write /tmp/breed.bin Bootloader`,一定不要打错了这条命令。等待路由器重启...
    ```
    root@XiaoQiang:~# mtd -r write /tmp/breed.bin Bootloader
    Unlocking Bootloader ...

    Writing from /tmp/breed.bin to Bootloader ...
    Rebooting ...
    ```

## 使用BREED刷入padavan

1. 拔下电源,按住路由器的reset键,再插上电源开机，等到路由的灯狂闪的时候，松开reset键
2. 用网线将路由器和电脑连接，在电脑上在浏览器中输入192.168.1.1，进入不死breed的控制台
3. 此选择固件更新，如下图，再选择下载好的[老毛子Padavan固件MI-R3G_3.4.3.9-099.trx](http://opt.cn2qq.com/padavan/MI-R3G_3.4.3.9-099.trx), 最好等待刷机完成，就会出现一个pdcn的wifi, 默认密码是1234567890; 网页地址: <http://192.168.123.1>, 账号密码：admin/admin

![breed固件更新](/imgs/breed.png)

在刷机过程中遇到一个小插曲，两台路由器，第一台按上面的步骤一点问题都没有，同事的一台就是进不了padavan系统，一直在breed系统中，后来在[小米3G刷了不死breed 后再刷任何固件没有反应怎么办？？？](http://www.miui.com/thread-12895070-3-1.html),得到解决方法：在不死控制台里有个参数是效验ROM的MD5值的，把它删掉然后开刷就就正常了。

在breed界面有个环境变量设置，里面个ROM md5值验证，删除掉后，再重新刷入padavan就ok了。

# 参考
1. [小米路由器3G刷机教程——不死Breed后台+Padavan固件](https://www.bilibili.com/read/cv802996/)
2. [小米路由器3G刷breed及老毛子Padavan固件教程](http://www.right.com.cn/forum/thread-257423-1-1.html)
3. [老毛子Padavan固件MI-R3G_3.4.3.9-099.trx本站备份](/files/MI-R3G_3.4.3.9-099.trx)
4. [breed-mt7621-xiaomi-r3g.bin本站备份](/files/breed-mt7621-xiaomi-r3g.bin)

