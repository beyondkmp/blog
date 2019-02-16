---
title: "Alfred2插件编写的方法"
date: 2015-07-10T00:00:00+0800
lastmod: 2015-07-10T00:00:00+0800
draft: false
keywords: ["Alfred2","插件"]
tags:  ["Alfred2","插件"]
description: "用Alfred插件来实现自动打开全局代理"
categories: ["常用技巧"]
tags: ["Alfred2","插件"]
---

**主要是实现全局代理的自动打开与关闭：**

![2](/imgs/Alfred2插件/2.png)

直接在Alfred 2中输入proxy，如果没有打开代理就会自动勾选那些选项，打开全局代理。如果打开了就会自动关闭关闭全局代理。如下图：

![3](/imgs/Alfred2插件/3.png)

主要步骤如下：

1. 第一步就是先找到用shell脚本来实现全局代理的打开与关闭的方法，具体见下面参考[1], 先是想改相应的配置文件，就是/Library/Preferences/SystemConfiguration/preferences.plist，这里面是xml有proxy的选项。但是用networksetup 命令来修改更加简单,主要有命令如下Wi-Fi可以通过选项-listallnetworkservices来查到的。

        networksetup -setwebproxystate Wi-Fi on 
        networksetup -setsecurewebproxystate Wi-Fi on
        networksetup -setftpproxystate Wi-Fi on
        networksetup -setsocksfirewallproxystate Wi-Fi on

2. 第二步就是编写相应的插件了,可以直接添加相应的模板：

    ![4](/imgs/Alfred2插件/4.png)

3. 如果直接用上面的shell 脚本命令添加到上图中的shell脚本中就会权限问题，每次运行都要输入密码，就要输入四次，这是很烦人的。具体见下面参考[2],方法就是用osascript命令来执行上面的命令，最后只要输入一次密码就可以了。具体代码如下：

        osascript -e  "do shell script \"networksetup -setwebproxystate Wi-Fi {query} &&   networksetup -setsecurewebproxystate Wi-Fi {query} && networksetup -setftpproxystate Wi-Fi {query} && networksetup -setsocksfirewallproxystate Wi-Fi {query};\" with administrator privileges"

4. 上面的只是实现了proxy on则打开代理，proxy off则关闭代理。但是受到wifi toggle这个插件的启发，觉的直接输入proxy，让他自己判断，如果是关的就打开，如果是开的就关闭。具体的代码如下（echo 后面的内容会输出到通知栏里面很不错的）

        #!/bin/bash

        networksetup -getwebproxy Wi-Fi | grep 'No' &>/dev/null && var=on || var=off 
        
        osascript -e  "do shell script \"networksetup -setwebproxystate Wi-Fi $var && networksetup -setsecurewebproxystate Wi-Fi $var && networksetup -setftpproxystate Wi-Fi $var && networksetup -setsocksfirewallproxystate Wi-Fi $var;\" with administrator privileges"

        echo "global proxy is $var"


#### 参考
1. [Change network proxy settings in via commandline in Mac OS X Lion](http://stackoverflow.com/questions/6796442/change-network-proxy-settings-in-via-commandline-in-mac-os-x-lion)
2. [Workflows requiring privileges](http://www.alfredforum.com/topic/686-workflows-requiring-privileges/)
3. [尝试给自己写Alfred Work](http://dalang.im/post/dev-logs/write-alfred-workflow)
4. [Alfred 2 Workflow Theme Tip List](http://www.alfredworkflow.com/)


