---
title: "apple tv3完美越狱"
date: 2020-02-02T11:46:42+0800
lastmod: 2020-02-02T11:46:42+0800
draft: false
keywords: ["apple tv3","完美越狱","jailbreak"]
description: "apple tv3完美越狱的方法"
tags: ["apple tv3","完美越狱","jailbreak"]
categories: ["运维人生"]
author: "beyondkmp"

---

7年过去了，apple tv3终于可以完美越狱了。在这里由衷的感谢@tihmstar。可以在最新的apple tvos 7.4(ios8)上面进行完美越狱。在不久的将来，像NitoV, iOS emulators, kodi等app应该会推出iOS8的定制版本, 到时候apple tv3会有更多的玩法。

## 完美越狱的步骤

[@tihmstar](https://twitter.com/tihmstar) 提供了一种方法，可以完美越狱所有的运行最新的tvos7.4(iOS 8.4.3)系统的apple tv3。进行下面步骤前：1. 先升级系统并恢复出厂设置(不是适用所有人，我的apple tv3没有做这两个就没有成功), 2. 关闭系统自动更新。

## 设置DNS

1. 开机
2. 选择 设置 -> 通用 -> 网络 -> Wi-Fi
3. 选择你连接的网络，并点击进去
4. 点击设置DNS, 选择手动，然后再DNS修改成 46.166.144.59（046.166.144.059)

<!--more-->

## 添加配置文件

1. 选择 设置 -> 通用
2. 选择 发送数据到苹果
3. 双击摇控器上面的play按钮(一定要双击，才会出来添加界面)
4. 点击添加配置文件，并将下面的连接填入`http://trailers.apple.com/trailers.cer`(一定要仔细核对，不要写错了)
5. 点击提交完成

## 越狱

1. 回到主界面
2. 选择预告片，你将会看到`etasonATV`
3. 单击 #etason, 这样会下载工具，将自动进行越狱(这里要等待几分钟，或者更长时间，请耐心等等)
4. 越狱成功后，apple tv3会自动重启，然后我们再将dns设置回自动
5. ssh登录apple tv,通过网络配置查看自身ip, 然后安装untether(这个一定要安装下)

    ```
    ssh root@192.186.0.1
    password: alpine

    dpkg -i /var/root/untether.deb
    ```

越狱完成后，在主界面不会添加新的app icon. 这种越狱方法支持 AppleTV3,2 (released in 2013, A1469 for Rev A) and AppleTV3,1 (released in 2012, A1427).

apple tv3安装Niotv和kodi都可以，具体见参考1. 不过我安装了，安装的版本都是apple tv2的版本，目前兼容性还不是很好，还容易卡死。这些app还没有针对apple tv3做适配。等以后适配好了，再来更新具体的安装方法。



##  参考

1. [Jailbreak Apple TV 3 with iOS 8](https://kubadownload.com/news/jailbreak-apple-tv-3)
2. [[Tutorial] Apple TV 3 Jailbreak and XMBC install guide - Updated](https://www.reddit.com/r/jailbreak/comments/eu0nye/tutorial_apple_tv_3_jailbreak_and_xmbc_install/)
