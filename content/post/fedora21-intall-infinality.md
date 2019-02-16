---
title: "给fedora 21安装infinality补丁"
date: 2015-01-23T00:00:00+0800
lastmod: 2015-01-23T00:00:00+0800
draft: false
keywords: ["fedora21","infinality"]
tags:  ["fedora21","infinality"]
description: "目前fedora官方还没有正式给出infinality,所以有此文"
categories: ["常用技巧"]
tags: ["fedora21","infinality"]
---


###在Fedora 21中安装

1. [RPM: freetype-infinality-2.4.12-1.20130514_01.fc21.x86_64.rpm](/imgs/fedora21-intall-infinality/freetype-infinality-2.4.12-1.20130514_01.fc21.x86_64.rpm)
2. [RPM: fontconfig-infinality-1-20120615_1.noarch.rpm](/imgs/fedora21-intall-infinality/fontconfig-infinality-1-20120615_1.noarch.rpm)

再安装gnome-tweak-tool来设置slight和rgb，安装包

[RPM: gnome-tweak-tool](/imgs/fedora21-intall-infinality/gnome_tweak_tool_installer-1-0.noarch.rpm)

安装好后将提示(hintig)设置成SLIGHT,将次像素次序(antialiasing)设置成RGBA,具体如下图：

![gnome-tweak-tool-setting](/imgs/fedora21-intall-infinality/gnome-tweak-tool-setting.png)


####参考

1. [HOW TO INSTALL FREETYPE INFINALITY IN FEDORA 21](http://fedora-apps.blogspot.com/2014/12/how-to-install-freetype-infinality.html)
