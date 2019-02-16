---
title: "OSX下好用的输入法rime"
date: 2016-01-17T00:00:00+0800
lastmod: 2016-01-17T00:00:00+0800
draft: false
keywords: ["OSX","输入法","五笔"]
tags:  ["OSX","输入法","五笔"]
description: "在mac下面用了好久的百度输入法，发现好多问题，终于发现了一个输入法神器了"
categories: ["常用技巧"]
tags: ["OSX","输入法","五笔"]
---

由于本人用五笔输入法，在linux下用过一段时间fcitx五笔，还是非常不错的，基本的功能需求都能达到。但是到mac下来，发现目前用的比较多的五笔输入法就是百度输入法与搜狗输入法了，搜狗输入法不支持词频调整直接弃用了。百度输入法支持，用了一段时间后发现其支持的词频调整并不是很好，还有一个问题我经常打"老师"这个词，奇葩的是竟然打不出来，用百度输入法的造词功能也不行。用了快一年的百度五笔输入法了，在launcher的搜索框中也不能切换输入法，问题是用五笔还经常打出一些好奇葩的词了，据说是百度词库牛逼啊，问题我不需要啊，而且那些奇葩的词还非常靠前。

基于上面这些原因，我准备不再用百度五笔输入法了，百度五笔输入法也基本没有更新了。去谷歌搜索，基本上定了以下三个输入法：万寿果输入法、清歌输入法和rime输入法。万寿果输入法官方都打不开了，在网上找的版本也是很久没有更新的，不试用了。去清歌输入法[官网](http://qingg.im)下载，最后一个版本是2014年7月更新的。用了下界面非常清爽，能手动调整词频，用ctrl+数字选择相应的词的就能调整到明前面。正如官网所说，具有以下特点：

![4](/imgs/rime/4.png)

经过本人使用后是真的，没有说大话，比起百度与搜狗的五笔输入法那是好用了不知道多少倍了，如果没有其它什么要求的话一般用这个输入法就非常适合了。

不过我还是再去试了试rime,鼠须管输入法，这个被很多人称为输入法神器。rime是一个跨平台的输入法，支持windows，linux和mac。在网上看到这个输入法的赞美数不胜数啊，能得到这么多人的赞美肯定有过人之处了。mac上可以到[官网下载](http://rime.im),也可以用命令`brew cask install squirrel`。

安装好后，打字速度真的是惊人的快啊，不过也有人说是因为其字库小所以打字速度才非常快，不过也正合我意了，我的字库不需要那么大，但是速度快是必须的。后面我发现的特性都把我惊呆了。

首先安装好后要设置相应的输入方案才能用五笔输入法。设置wubi86的输入方案，点击输入法图标，再点击**用户设定**,在打开的文件夹中找到相应的default.yaml,在default.yaml中的schema_list下添加下行,保存后点击**重新部署**就ok了。

```
- scheme: wubi86
```

不过最好是用default.custom.yaml文件来添加相应的配置，以后升级就不会覆盖掉自己的配置文件了。

默认的界面字段有点大，而且是竖排的，可能有很多人不习惯。通过以下配置来实现横排界面，在squirrel.yaml中有style选项，设置horizontal为true就可以实现横排了。不过我觉的竖排还是比较好用的。

```
style:
  horizontal: true  #记得冒号前加空格
  font_point: 18   #修改字体大小
  inline_preedit: false
```

默认的界面的编码与字不在同一界面，如下图所示，

![1](/imgs/rime/1.png)

将style下面的inline_preedit配置成false就可以实现以下效果：

![2](/imgs/rime/2.png)

默认的候选词是5个，可以修改spacing来添加更多的候选词。

```
style:
  spacing: 8
```

如果觉的默认主题不好看，可以通过color_scheme来修改主题，预置了不少主题，在squirrel里面都可以查看到。更换主题是非常的方便的。

```
style:
  color_scheme: clean_white
```

在相应的app中默认优先使用英文模式，而不是中文模式，在squirrel中添加就可以,com.alfredapp.Alfred,代表的是CFBundleIdentifier，可以在安装软件的包文件中的info.plist中查找到,也可以通过下面的命令来查看查看`osascript -e 'id of app "Finder"'`。

```
app_options:
  com.alfredapp.Alfred:
    ascii_mode: true
  com.runningwithcrayons.Alfred-2:
    ascii_mode: true
  com.blacktree.Quicksilver:
    ascii_mode: true
  com.apple.Terminal:
    ascii_mode: true
    no_inline: true
  com.googlecode.iterm2:
    ascii_mode: true
  org.vim.MacVim:
    ascii_mode: true
    no_inline: true
```
但是有一个问题就是launchpad里面的搜索框还是不能默认用英文模式，我以为是com.apple.launchpad.launcher，设置后还是没有反应，后面查看/var/log/system.log，得到,所以应该是com.apple.dock.ecti设置为ascii_mode模式才可以，不过在那个搜索框还是用不了shift切换中英文模式。可以直接用ctrl+shift+2来切换,这个快捷键在default.yaml里面可以找到。

```
Jan 17 17:40:40 birdcat-2 Squirrel[424]: createSession: com.apple.dock.ecti
Jan 17 17:40:40 birdcat-2 Squirrel[424]: setAppOptions: com.apple.dock.ecti
```

rime输入法还有造词的功能，比如打"习大大"，默认词库里面没有，先打"nddd",这个没有，再一个一个字，就会出来，rime会自动猜记新词,具体参考[小狼毫五笔输入法0.9.26版自动造词功能介绍](http://tieba.baidu.com/p/2646302720)

![3](/imgs/rime/3.png)

经过上面的简单配置后,鼠须管就是我理想中的输入法啊，配置太丰富了，可以完全定制，非常精悍。也向有同样需求的人推荐这款输入法,如果不习惯配置文件，也可以用界面，如[SCU（Squirrel 配置工具）](https://github.com/neolee/SCU)。

## 参考
1. [RimeWithSchemata](https://github.com/rime/home/wiki/RimeWithSchemata)
2. [致第一次安装RIME的你](https://www.zybuluo.com/eternity/note/81763)
3. [Rime输入法—鼠须管(Squirrel)词库添加及配置](http://www.jianshu.com/p/cffc0ea094a7)
4. [安装及配置 Mac 上的 Rime 输入法——鼠鬚管 (Squirrel)](http://www.dreamxu.com/install-config-squirrel/)
5. [鼠须管之五笔配置](http://old.mutoo.im/2013/01/using-wubi-in-squirrel/)
6. [获取程序的bundle identifier](https://cuitlongit.wordpress.com/2013/10/11/获取程序的bundle-identifier/)
