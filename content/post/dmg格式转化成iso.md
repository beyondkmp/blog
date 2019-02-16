---
title: "dmg格式转化成iso"
date: 2015-03-17T00:00:00+0800
lastmod: 2015-03-17T00:00:00+0800
draft: false
keywords: ["dmg","iso","转化"]
tags:  ["dmg","iso","转化"]
description: "苹果的dmg安装软件转化成iso"
categories: ["常用技巧"]
tags: ["dmg","iso","转化"]
---

1. 安装dmg2img,在gentoo下的源里面直接有
    
    ```bash

    sudo emerge -av dmg2img

    ```
2. man下dmg2img,没啥，直接dmg2img有以下说明：
    
    ```bash
    beyond@beyond:~|⇒  dmg2img 

    dmg2img v1.6.5 (c) vu1tur (to@vu1tur.eu.org)

    Usage: dmg2img [-l] [-p N] [-s] [-v] [-V] [-d] <input.dmg> [<output.img>]
    or     dmg2img [-l] [-p N] [-s] [-v] [-V] [-d] -i <input.dmg> -o <output.img>

    Options: -s (silent) -v (verbose) -V (extremely verbose) -d (debug)         
             -l (list partitions) -p N (extract only partition N)
    ```
3. 用最简单的来生成

    ```bash
    dmg2img InstallESD.dmg InstallESD.iso
    ```

