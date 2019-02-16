---
title: "ios及os x 下技巧收集"
date: 2015-04-10T00:00:00+0800
lastmod: 2015-04-10T00:00:00+0800
draft: false
keywords: ["ios","os x","技巧收集"]
tags:  ["ios","os x","技巧收集"]
description: "本文将收集一些ios与os x 常用技巧"
categories: ["常用技巧"]
tags: ["ios","os x","技巧收集"]
---

## ios技巧收集

#### 技巧一：禁止屏幕旋转

在主屏幕上往上拉出下面的快捷界面，点击那个小锁就可以。如下图：
![禁止屏幕旋转]({{ IMAGE_PATH}}/ios及os的技巧收集/1.png)

#### 技巧二：打开连接wifi界面

在主屏幕上往上拉出下面的快捷界面，长按住wifi的小图标，就会自动跳转到wifi连接界面。
![wifi连接]({{ IMAGE_PATH}}/ios及os的技巧收集/1.png)

### 技巧三：不用电源键关机

调出ios的assisitiveTouch,在控制->通用->辅助功能里面。就是屏幕上的小灯泡样的东西。调出后点击，再点设备，可以看到下面的锁定屏幕直接长按就会出来关机界面。

![不用电源键关机]({{ IMAGE_PATH}}/ios及os的技巧收集/3.png)

### 技巧四：在锁屏状态下挂电话

在锁屏状态下没有直接挂电话的界面，可以连续按两下电源键就可以挂电话了。按一下电源键就是静音，但是没有挂电话。

### 技巧五：ios越狱后忘记openssh密码怎么修改

* 下载iFunbox或者iFile
* 打开/etc/master.passwd,用iFile的文本编辑功能打开master.passwd或者用iFunbox来在电脑上修改
* 找到root那行，大致如：root:UlD3amElwHEpc:0:0::0:0:System
* 将上面的UlD3amElwHEpc换成ab3z4hnHA5WdU，你的openssh的密码就变成了abc123,用terminal或者ssh连接上修改密码。

## os x技巧收集

#### 技巧一：用空格查看图片

你要查看图片，可以直接选中，按下空格就可以查看图片了，还有选中多张一起查看。
![用空格查看图片]({{ IMAGE_PATH}}/ios及os的技巧收集/2.png)

#### 技巧二：修改文件名
行路文件，直接按下enter就可以修改文件名了。

### 技巧三：恢复回收站里的文件
选中回收站的文件，按command+delete键就恢复了。

### 技巧四：显示隐藏文件

打开终端，输入下面的命令，并重启Finder就可以了。

```
defaults write com.apple.finder AppleShowAllFiles -bool true  #这是显示
defaults write com.apple.finder AppleShowAllFiles -bool false  #这是隐藏
```

### 技巧五：选择默认程序

* 先选择要相关的文件类型，例如.c文件,右键选择显示简介
* 在简介界面有个打开方式，选择相应的程序，再点击下面的全部更改就ok了!

![选择默认程序]({{ IMAGE_PATH}}/ios及os的技巧收集/4.png)

## 技巧六：输入法安装文件的位置

```
beyond@birdcat:/Library/Input Methods|
⇒  pwd
/Library/Input Methods
beyond@birdcat:/Library/Input Methods|
⇒  ls
SogouWBInput.app WBIM.app
beyond@birdcat:/Library/Input Methods|
⇒
```
## 技巧七：修改Caps lock与control之间的重映射

在Apple Menu -> System Preferneces ->Keyboard -> 修饰键，如下图：

![keyboard setting]({{ IMAGE_PATH}}/ios及os的技巧收集/5.png)

## 技巧八：修改状态栏上小图标的位置

比如要将蓝牙的位置移动到wifi的后面，**按住command键**再拖动到wifi后面就可以了。如果不需要直接将其拖出状态栏就可以了！！！

## 技巧九： 删除config导入的vpn配置

用config导入vpn设置，但是在网络设置中删除不了vpn配置。打开系统偏好设置--> 描述文件--> vpn配置文件，删除即可。


