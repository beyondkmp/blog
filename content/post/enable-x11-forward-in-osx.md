---
title: "在mac osx安装XQuartz并配置x11 forwarding"
date: 2017-02-21T00:00:00+0800
lastmod: 2017-02-21T00:00:00+0800
draft: false
keywords: ["x windows","XQuartz","xclip","x11 forwarding"]
tags:  ["x windows","XQuartz","xclip","x11 forwarding"]
description: "当远程连接服务器,并在vim里面编辑时总是不能愉快的copy和paste"
categories: ["常用技巧"]
tags: ["x windows","XQuartz","xclip","x11 forwarding"]
---


## 安装X window并配置
1. 下载[XQuartz](https://www.xquartz.org/),并安装
2. 配置远程服务器ssh服务
    * 安装 xorg-xauth and xorg-xhost 
    * 配置 /etc/ssh/sshd_config:
        
        ```bash
        AllowTcpForwarding  yes
        X11UseLocalhost  yes
        X11DisplayOffset 10
        X11Forwarding  yes
        ```
    * 重启sshd服务  
3. 在客户端可以用`-X`来连接服务器,或者将 `ForwardX11  yes` 加入到ssh配置中
4. 在客户端运行`ssh -X test@your_server_ip`, 连接后执行`xclock`,如果看到有一个时钟出来,就说明成功了. 如果没有的话加`-vvv`进行debug调试看下是哪里出错.

## 配置xclip与vim
1. 设置xquartz,在pasteboard中选中所有
	![xquartz pasteboard](/imgs/xquartz/1.jpg)

2. 为了可以将远程服务器的系统粘贴板直接复制到本地
    * 安装xclip
    * `ssh -X test@your_server_ip`到远程服务器, 运行`xclip -i /etc/passwd`, 本地就能直接粘贴/etc/passwd的内容

3. 配置好上面的系统粘贴板后, 如果要想vim里面的复制的内容也能直接输出到本地,那样拷贝部分代码,文档就非常方便了.
    * 查看vim是否支持X11
        ```
        vim --version | grep clipboard
        # 如果返回显示clipboard及xterm_clipboard是减号,就说明不支持
        ```
    * 可以通过安装gvim或者安装vim-x11, 下面给出archlinux的安装方法
        * 方法一: 安装pacman
            ```
            sudo pacman -S gvim
            ```
        * 方法二: abs编译安装,见参考

            ```
            # pacman -S abs
            # abs extra/vim
            # cp -r /var/abs/extra/vim ~/ && cd ~/vim
            修改 PKGBUILD 文件
            
            PKGBUILD
            ...
            pkgname=('vim' 'gvim' 'vim-runtime')
            # 去除不需要编译的包
            # 至于编译依赖，不了解可以像我一样选择不取消
            ...
            --with-x=noyes \
            # 把 Vim 编译选项中 --with-x 的 no 改为 yes
            ...
            编译并创建包
            
            $ makepkg -s
            安装 Vim
            
            # pacman -U vim-7.3.754-1-x86_64.pkg.tar.xz
            重新进行验证，果然生效
            ```
        * 方法三: 通过yaourt安装

            ```
            yaourt -S vim-x11
            ```
4. 配置vim,在vim中使用`"+y`可以将选中的内容复制到系统粘贴板中,在视图模式我们设置如下快捷键Ctrl+c就可以直接将选中的内容复制到本地系统粘贴板中

    ```
    vnoremap <C-c> "+y
    ```
    
## 参考
1. [XQuartz project](https://www.xquartz.org/)
2. [Apple OS X: Install X Window System XQuartz For SSH X11 Forwarding On a Mavericks or Yosemite](https://www.cyberciti.biz/faq/apple-osx-mountain-lion-mavericks-install-xquartz-server/)
3. [Vim 与系统剪切板交互](http://blog.fooleap.org/using-vim-with-clipboard-on-archlinux.html)
4. [Quick yank and paste](http://vim.wikia.com/wiki/Quick_yank_and_paste)



