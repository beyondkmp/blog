---
title: "apue.3e源码在centos6.5上编译"
date: 2015-08-11T00:00:00+0800
lastmod: 2015-08-11T00:00:00+0800
draft: false
keywords: ["apue.3e","源码编译"]
tags:  ["apue.3e","源码编译"]
description: "在编译过程会出现heapsort错误,主要是缺少libbsd包"
categories: ["linux上c编程"]
tags: ["apue.3e","源码编译"]
---

1. 到[官网](http://www.apuebook.com/code3e.html)下载源代码。
2. 直接make会出现下面的错误:

    ```
    ../lib -lapue -pthread -lrt -lbsd
    gcc -ansi -I../include -Wall -DLINUX -D_GNU_SOURCE  barrier.c -o barrier
    -L../lib -lapue -pthread -lrt -lbsd
    /tmp/ccaYTQPh.o: In function `thr_fn':
    barrier.c:(.text+0x80): undefined reference to `heapsort'
    collect2: ld 返回 1
    make[1]: *** [barrier] 错误 1
    make[1]: Leaving directory `/home/beyond/code/apue.3e/threads'
    make: *** [all] 错误 1
    ```
3. 到下面的**参考1**中找到了相应的包，备份如下：

    [libbsd-0.2.0-4.el6.elrepo.x86_64.rpm](/imgs/apue.3e/libbsd-0.2.0-4.el6.elrepo.x86_64.rpm)

    [libbsd-devel-0.2.0-4.el6.elrepo.x86_64.rpm](/imgs/apue.3e/libbsd-devel-0.2.0-4.el6.elrepo.x86_64.rpm)

4. 安装上面的包后可以顺利编译通过，再参照下面**参考2**中,将头文件和生成的静态链接库放到gcc的默认搜索路径下，以后就不要自己添加了。(libapue.a是apue.h头文件中包含的所有函数及宏定义的具体实现，是一个静态链接库)

    ```
    cp ./include/apue.h /usr/include/
    cp ./lib/libapue.a /usr/local/lib/
    ```

## 参考

1. [CentOS 7 下安装libbsd-dev 编译apue3时出错处理](http://blog.csdn.net/macanv/article/details/41457425)
2. [Linux下配置APUE的编译 报错之后如何处理](http://www.shazidoubing.com/cplus/639.html)
