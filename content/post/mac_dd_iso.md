---
title: "macos下使用dd刻录iso"
date: 2019-01-18T17:15:41+0800
lastmod: 2019-01-18T17:15:41+0800
draft: false
keywords: ["macos","dd","iso刻录"]
description: "macos下使用dd命令刻录iso到u盘"
tags: ["macos","dd","iso刻录"]
categories: ["mac"]
author: "beyondkmp"

---

## 具体步骤

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


2. 插上u盘后，os x下会自动挂载，我们要写入时必须先卸载他。用下面的命令进行卸载：

    ```bash
    $ diskutil unmountDisk /dev/disk4
    ```

3. 写入的过程，如果在OS X下用dd,要用/dev/rdisk4,而不是上面的/dev/disk4 和bs=1m。rdisk指得是`raw disk`，在macos写入raw disk是非常快的。bs=1m表示1MB块大小。如果是linux系统的话要使用`bs=1M`

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

## 参考

1. [USB flash installation media](https://wiki.archlinux.org/index.php/USB_flash_installation_media#In_Mac_OS_X)
2. [How to create a bootable USB stick on OS X](http://www.ubuntu.org.cn/download/desktop/create-a-usb-stick-on-mac-osx)

