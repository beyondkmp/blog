---
title: glide使用总结
date: 2017-05-30T00:00:00+0800
lastmod: 2017-05-30T00:00:00+0800
draft: false
keywords: ["glide使用", "golang包管理"]
tags:  ["glide使用", "golang包管理"]
description: "使用glide管理golang依赖包版本"
categories: ["golang学习"]
---

当我们从github下载开源的go项目时，开源项目一般都会引入很多其它的开源库，这样我们直接去编译的时候就会提示缺少哪些库，然后一个一个用`go get`命令安装上，这样是在浪费生命啊！使用glide直接一键安装，还可以管理依赖包的版本号，这样当相关依赖包进行了重大升级时，我们也不怕。

## Install glide

* 方法一: 使用`go get`安装

    ```
    go get github.com/Masterminds/glide
    ```
* 方法二: 使用官方安装脚本

    ```
    curl https://glide.sh/get | sh
    ```
* 方法三: 使用系统自带的安装软件

    ```
    # mac osx
    brew install glide
    #arch linux
    pacman -S glide
    ```

## 使用

```
$ glide create                            # Start a new workspace
$ vim glide.yaml                         # and edit away!
$ glide get github.com/Masterminds/cookoo # Get a package and add to glide.yaml
$ glide install                           # Install packages and dependencies
# work, work, work
$ go build                                # Go tools work normally
$ glide up                                # Update to newest versions of the package
```

### glide create(或init)

初始化一个开发环境。在初始化过程中，会自动创建glide.yaml文件，并会尝试猜测依赖包的版本号，并将其写入yaml配置文件中。比如，你的项目使用的Godep管理的，glide会自动探测Godep使用的版本号。

```
$ glide create
[INFO]	Generating a YAML configuration file and guessing the dependencies
[INFO]	Attempting to import from other package managers (use --skip-import to skip)
[INFO]	Scanning code to look for dependencies
[INFO]	--> Found reference to github.com/Masterminds/semver
[INFO]	--> Found reference to github.com/Masterminds/vcs
[INFO]	--> Found reference to github.com/codegangsta/cli
[INFO]	--> Found reference to gopkg.in/yaml.v2
[INFO]	Writing configuration file (glide.yaml)
[INFO]	Would you like Glide to help you find ways to improve your glide.yaml configuration?
[INFO]	If you want to revisit this step you can use the config-wizard command at any time.
[INFO]	Yes (Y) or No (N)?
n
[INFO]	You can now edit the glide.yaml file. Consider:
[INFO]	--> Using versions and ranges. See https://glide.sh/docs/versions/
[INFO]	--> Adding additional metadata. See https://glide.sh/docs/glide.yaml/
[INFO]	--> Running the config-wizard command to improve the versions in your configuration
```

### glide install

要安装`glide.lock`文件里面的特定版本的依赖包，使用`glide install命令`

```
$ glide install
```

这个命令会读取`glide.lock`文件，并会安装特定commit id的版本到此目录下面的vender文件夹中,如果这个文件不存在，运行`glide install`时将会执行一次update并生成lock文件

## 使用实例

```
cd $GOPATH/src
git clone https://github.com/xtaci/kcptun.git
cd kcptun
glide create
glide install
cd server
go build
```

### 说明

1. 一定要放到$GOPATH/src目录下面，要不`go build`找不到`vender`目录下面的依赖包。这个是由于`go build`查找相应的依赖包的机制有关。
2. 不要直接用`go build`, 这样会带上debug信息，生成的可执行文件非常大,使用下面命令编译，不带上debug信息

    ```
    go build -ldflags "-s -w"
    ```
3. 使用`-s`选项后安装包还是非常大，我们再使用upx工具对其进行终极压缩，可以缩小几倍
    * 安装goupx

        ```
        $ pacman -S upx
        $ go get github.com/pwaller/goupx
        ```
    * 编译大小对比,upx可以对可以执行文件进行压缩并进行一些混淆,这里主要用来作为压缩功能。此处压缩后的大小与gz压缩后的大小差不多

        ```
        $ go build
        $ ls -lh
        total 9.5M
        -rw-r--r-- 1 vagrant vagrant 1.2K May 30 23:53 config.go
        -rw-r--r-- 1 vagrant vagrant  11K May 30 23:53 main.go
        -rwxr-xr-x 1 vagrant vagrant 9.5M May 31 00:06 server
        -rw-r--r-- 1 vagrant vagrant  391 May 30 23:53 signal.go


        $ go build -ldflags "-s -w"
        $ ls -lh
        total 6.4M
        -rw-r--r-- 1 vagrant vagrant 1.2K May 30 23:53 config.go
        -rw-r--r-- 1 vagrant vagrant  11K May 30 23:53 main.go
        -rwxr-xr-x 1 vagrant vagrant 6.4M May 31 00:06 server
        -rw-r--r-- 1 vagrant vagrant  391 May 30 23:53 signal.go


        $ goupx server
        2017/05/31 00:06:24 {Class:ELFCLASS64 Data:ELFDATA2LSB Version:EV_CURRENT OSABI:ELFOSABI_NONE ABIVersion:0 ByteOrder:LittleEndian Type:ET_EXEC Machine:EM_X86_64 Entry:4570928}
        2017/05/31 00:06:24 File fixed!
                               Ultimate Packer for eXecutables
                                  Copyright (C) 1996 - 2017
        UPX 3.94        Markus Oberhumer, Laszlo Molnar & John Reiser   May 12th 2017

                File size         Ratio      Format      Name
           --------------------   ------   -----------   -----------
           6644512 ->   2400840   36.13%   linux/amd64   server

        Packed 1 file.
        $ ls -lh
        total 2.4M
        -rw-r--r-- 1 vagrant vagrant 1.2K May 30 23:53 config.go
        -rw-r--r-- 1 vagrant vagrant  11K May 30 23:53 main.go
        -rwxr-xr-x 1 vagrant vagrant 2.3M May 31 00:06 server
        -rw-r--r-- 1 vagrant vagrant  391 May 30 23:53 signal.go
        ```



