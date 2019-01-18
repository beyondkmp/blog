---
title: "macos下制作系统安装盘"
date: 2019-01-18T17:15:41+0800
lastmod: 2019-01-18T17:15:41+0800
draft: false
keywords: ["macos","dd","iso刻录"]
description: "macos下使用dd命令刻录iso到u盘"
tags: ["macos","dd","iso刻录"]
categories: ["mac"]
author: "beyondkmp"

---

## 使用dd命令

目前这个dd命令只适合制作linux的安装启动盘，不适合win10。

<!--more-->

1. 先查看u盘的位置

    ```bash
    $ diskutil list
    /dev/disk0 (internal, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:      GUID_partition_scheme                        *121.3 GB   disk0
       1:                        EFI EFI                     209.7 MB   disk0s1
       2:          Apple_CoreStorage Macintosh HD            120.5 GB   disk0s2
       3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
    /dev/disk1 (internal, virtual):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:                  Apple_HFS Macintosh HD           +120.2 GB   disk1
                                     Logical Volume on disk0s2
                                     F9F82D9B-129B-4A0C-B5F5-1DF79AB1B1F7
                                     Unencrypted
    /dev/disk2 (internal, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:      GUID_partition_scheme                        *129.8 GB   disk2
       1:                        EFI NO NAME                 536.9 MB   disk2s1
       2: FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF               32.2 GB    disk2s2
       3: FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF               64.6 GB    disk2s3
       4:       Microsoft Basic Data Share                   32.4 GB    disk2s4
    /dev/disk3 (external, physical):
       #:                       TYPE NAME                    SIZE       IDENTIFIER
       0:     FDisk_partition_scheme                        *1.0 TB     disk3
       1:               Windows_NTFS Elements                1.0 TB     disk3s1
    ```


2. 插上u盘后，macos下会自动挂载，我们要写入时必须先卸载他。用下面的命令进行卸载：

    ```bash
    $ diskutil unmountDisk /dev/disk4
    ```

3. 写入的过程，如果在macos下要用`/dev/rdisk4`,而不是上面的`/dev/disk4`. rdisk指得是`raw disk`，在macos写入`raw disk`是非常快的。bs=1m表示1MB块大小。如果是linux系统的话要使用`bs=1M`

    ```bash
    $ sudo dd if=Downloads/kali-linux-1.0.9a-amd64.iso of=/dev/rdisk4 bs=1m
    ^[363+1 records in
    363+1 records out
    3053371392 bytes transferred in 369.204163 secs (8270143 bytes/sec)
    ```

4. 最后推出相应的u盘

    ```bash
    $ diskutil eject /dev/disk4
    ```

## 使用unetbootin制作win 10安装盘

1. 下载win10安装镜像，[官方地址](https://www.microsoft.com/zh-cn/software-download/windows10ISO)

    ![win10 download](/imgs/win10_download.png)

2. 格式化u盘, 使用macos自带的磁盘工具,选中插入的u盘，点击抹掉,然后选择下面的格式进行格式化

    ```
    Name: FAT32
    Format: MS-DOS (FAT)
    Scheme: Master Boot Record
    ```
    ![disk utility erase disk](/imgs/disk-utility-erase-disk.png)

3. 将usb刻录成win10安装盘, 下载unetbootin并打开，打开过程中要输入密码,选择下面的参数，最后点击ok，等待写入完毕就完成了。

    ```
    Diskimage: checked, set to ISO and browse to your Windows 10 ISO
    Type: USB Drive
    Drive: Your USB drive (you should only see one entry here)
    ```
    ![unetbootin](/imgs/unetbootin.png)

如果有多个usb盘选择，你可以通过下面的命令来确认哪个盘是你要写入的。

```
$ diskutil list FAT32
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *16.0 GB    disk2
   1:                 DOS_FAT_32 FAT32                   16.0 GB    disk2s1
```

## 参考

1. [USB flash installation media](https://wiki.archlinux.org/index.php/USB_flash_installation_media#In_Mac_OS_X)
2. [How to create a bootable USB stick on OS X](http://www.ubuntu.org.cn/download/desktop/create-a-usb-stick-on-mac-osx)
3. [Installing Windows 10 on a Mac without Bootcamp](http://fgimian.github.io/blog/2016/03/12/installing-windows-10-on-a-mac-without-bootcamp/)

