---
title: "配置aria2高速下载百度网盘资源"
date: 2016-01-08T00:00:00+0800
lastmod: 2016-01-08T00:00:00+0800
draft: false
keywords: ["aria2配置","百度网盘资源"]
tags:  ["aria2配置","百度网盘资源"]
description: "配置aria2,以后可以高速下载百度网盘资源了"
categories: ["常用技巧"]
tags: ["aria2配置","百度网盘资源"]
---

aria2 是一个轻量级的跨平台的命令行下载工具，支持多种协议。目前支持的协议有：HTTP/HTTP,FTP,SFTP,BitTorrent和MetaLink。aria2也可以通过JSON-PRC和XML-PRC接口来控制操作。

## 安装方法

```
brew install aria2
```

## aria2的优势

* **Multi-Connection Download**.aria2可以从通过多种协议或多个源头下载同一文件，这样可以最大化下载代宽。
* **Lightweight**. aria2运行时不会占用太多内存与cpu，非常轻量级。当磁盘缓存关闭时，aria2占用物理内存通常为4Mib(正常的HTTP/FTP下载)到9Mib(Bittorrent下载).使用Bittorrent以2.8Mib/sec速度下载，cpu占用率大概为6%。
* **Fully Featured Bittorrent Client**. 所有的BitTorrent客户端的功能都能使用:DHT,PEX,Encryption,Magnet URI,Web-Seeding,Selective Downloads,Local Peer Discovery 和UDP tracker.
* **Remote Control**. aria2运行PRC接口来控制aria2进程。支持JSON-PRC(over HTTP and WebSocket)和XML-PRC两种接口。

## 配置aria2

aria2有两种下载模式：一种是直接命令行模式下载，平时可以用用。另一种是RPC模式，在这种模式下aria2启动后会在后台安静的等待下载请求，下载完成后也仍然后安全的驻留在后台不会自动退出。本文使用RPC模式，用以下配置文件来配置此模式(此配置文件参考[1](https://blog.icehoney.me/posts/2015-01-31-Aria2-download)),此配置文件保存在~/.aria2/aria2.conf。

```
#用户名
#rpc-user=user
#密码
#rpc-passwd=passwd
#上面的认证方式不建议使用,建议使用下面的token方式
#设置加密的密钥
#rpc-secret=token
#允许rpc
enable-rpc=true
#允许所有来源, web界面跨域权限需要
rpc-allow-origin-all=true
#允许外部访问，false的话只监听本地端口
rpc-listen-all=true
#RPC端口, 仅当默认端口被占用时修改
#rpc-listen-port=6800
#最大同时下载数(任务数), 路由建议值: 3
max-concurrent-downloads=5
#断点续传
continue=true
#同服务器连接数
max-connection-per-server=5
#最小文件分片大小, 下载线程数上限取决于能分出多少片, 对于小文件重要
min-split-size=10M
#单文件最大线程数, 路由建议值: 5
split=10
#下载速度限制
max-overall-download-limit=0
#单文件速度限制
max-download-limit=0
#上传速度限制
max-overall-upload-limit=0
#单文件速度限制
max-upload-limit=0
#断开速度过慢的连接
#lowest-speed-limit=0
#验证用，需要1.16.1之后的release版本
#referer=*
#文件保存路径, 默认为当前启动位置
dir=/Users/beyond/Downloads
#文件缓存, 使用内置的文件缓存, 如果你不相信Linux内核文件缓存和磁盘内置缓存时使用, 需要1.16及以上版本
#disk-cache=0
#另一种Linux文件缓存方式, 使用前确保您使用的内核支持此选项, 需要1.15及以上版本(?)
#enable-mmap=true
#文件预分配, 能有效降低文件碎片, 提高磁盘性能. 缺点是预分配时间较长
#所需时间 none < falloc ? trunc << prealloc, falloc和trunc需要文件系统和内核支持
file-allocation=prealloc
```
第一次使用，直接copy[雪月秋水](https://blog.icehoney.me/)的配置文件。配置好上面后，直接在终端运行`aria2c --conf-path=<PATH> -D`,PATH必须是绝对路径，-D表示使Aria2在后台运行，即使关闭终端也不会停止运行。最后就是怎么管理Aria2的下载任务了，可以用binux的[YAAW](https://github.com/binux/yaaw),还可以使用[在线版本](http://binux.github.io/yaaw/demo/)。不过还有一个更美观的webui，就是ziahamza的[webui-aria2](https://github.com/ziahamza/webui-aria2)，也有在线版本的[在线版本webui-aria2](http://ziahamza.github.io/webui-aria2/)，如下图所示。

![webui-aria2](/imgs/配置aria2/overview.png)

如果是mac的话也可以用aria2GUI这个软件，[下载地址](http://bbs.feng.com/read-htm-tid-10217584.html)，界面如下图：

![aria2GUI](/imgs/配置aria2/2.png)

## 安装百度网盘导出插件

到[Baiduexporter](https://github.com/acgotaku/BaiduExporter)下载相应版本的游览器插件,安装好后打开百度网盘会多出一个导出设置，如下图所示，直接导出下载就可以了。

![百度网盘导出aria2-download](/imgs/配置aria2/1.png)

## 参考
1. [使用Aria2下载百度网盘和115的资源](https://blog.icehoney.me/posts/2015-01-31-Aria2-download)
2. [Mac 上使用百度网盘很烦躁？花点时间配置 aria2 吧](http://sspai.com/32167)
3. [aria2配置示例](http://blog.binux.me/2012/12/aria2-examples/)
