---
title: "unpv3源码在64位Linux上编译"
date: 2015-08-12T00:00:00+0800
lastmod: 2015-08-12T00:00:00+0800
draft: false
keywords: ["unpv3","源码编译"]
tags:  ["unpv3","源码编译"]
description: "在编译过程中会遇到不少问题，本文将一一来解决"
categories: ["linux上c编程"]
tags: ["unpv3","源码编译"]
---

1. 从[unpbook.com](http://unpbook.com)下载源代码
2. 解压，安装README中的步骤安装

    ```
    ./configure    # try to figure out all implementation differences

    cd lib         # build the basic library that all programs need
    make           # use "gmake" everywhere on BSD/OS systems

    cd ../libfree  # continue building the basic library
    make

    cd ../libroute # only if your system supports 4.4BSD style routing sockets
    make           # only if your system supports 4.4BSD style routing sockets

    cd ../libxti   # only if your system supports XTI
    make           # only if your system supports XTI

    cd ../intro    # build and test a basic client program
    make daytimetcpcli
    ./daytimetcpcli 127.0.0.1
    ```

3. 在libfree里面编译时会出现下面的错误：

    ```
    ⇒  make
    gcc -I../lib -g -O2 -D_REENTRANT -Wall   -c -o inet_ntop.o inet_ntop.c
    inet_ntop.c: 在函数‘inet_ntop’中:
    inet_ntop.c:62: 错误：实参‘size’与原型不符
    /usr/include/arpa/inet.h:65: 错误：原型声明
    make: *** [inet_ntop.o] 错误 1
    ```
4. 上面的错误说明已经说的很清楚了，到/usr/include/arpa/inet.h里面看到size的类型是socklen\_t而不是size\_t,将inet\_ntop.c中62行里面的size类型改成socklen_t,就可以编译通过了。
5. 在libroute里面也会编译出错，这个不用管。
6. 在libgai里面make,在上一级目录会生成libunp.a静态库，拷贝到/usr/lib文件夹下面

    ```
    # cd libgai
    #  make
    ar rv ../libunp.a
    ranlib ../libunp.a
    # cd ..
    # cp libunp.a /usr/lib
    # cp libunp.a /usr/lib64
    ```

7. 将lib下  的unp.h和上一级目录下config.h拷贝到/usr/include下面,先将unp.h里面的`#include "../config.h"`改成`#include "config.h"`

    ```
    # cp lib/unp.h /usr/include
    # cp config.h /usr/include
    ```

8. 现在可以开始编译第一个程序了：`gcc daytimetcpcli.c -o test -lunp`,直接运行会出错，因为linux没有安装xinetd,安装好后将/etc/xinetd.h下面的daytime-stream和daytime-dgram里面的disable的值改为no，再启动服务就可以了。

    ```
    # yum install xinetd
    # /etc/init.d/xinetd start
    ```

9. 最后的运行结果如下：

    ```
    ./test 127.0.0.1
    12 AUG 2015 14:43:11 CST
    ```

## 参考
1. [unix network programming book code has bugs due to old OS, how to solve this or where to get new code ?](http://stackoverflow.com/questions/7947960/unix-network-programming-book-code-has-bugs-due-to-old-os-how-to-solve-this-or)
2. [\<unix网络编程\> 的环境配置](http://www.cnblogs.com/hancm/p/3866731.html)
3. [在64位系统上配置UNP环境](http://www.iippu.com/?p=15)
