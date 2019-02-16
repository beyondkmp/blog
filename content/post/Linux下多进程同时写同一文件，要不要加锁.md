---
title: "Linux下多进程写同一文件，要不要加锁？"
date: 2016-02-04T00:00:00+0800
lastmod: 2016-02-04T00:00:00+0800
draft: false
keywords: ["Linux多进程",写文件","加锁"]
tags:  ["Linux多进程",写文件","加锁"]
description: "经常在写日志文件时，在多进程环境下要考虑要不要加锁,本文将给出一个答案"
categories: ["linux上c编程"]
tags: ["Linux多进程",写文件","加锁"]
---


本文翻译自：[Is lock-free logging safe?](http://www.jstorimer.com/blogs/workingwithcode/7982047-is-lock-free-logging-safe)

最近，我看到了mono_logger_project。它声称给ruby2.0提供了不用加锁的日志系统。当我看到这个项目时，我在想：Cool!我要看看他们用什么技术来实现这种日志系统的。一开始觉的他是非常简单易懂的。当我深入理解并将它与ruby中的标准库Logger比较时，发现他俩是完全一样的，不过减去了互斥(mutex).

这里让我不明白的是，无锁的日志系统安全吗？Ruby添加互斥锁的原因之一可能就是为了安全起见。如果在同一时间，多个线程同时写一个日志文件会怎么样？插入的日志信息会不会像下面：

![1](/imgs/linux多进程写文件/1.png)

## Unix beard too short...(unix胡须太短，等不及)

这是一个深奥的Unix问题。当我在mono_logger的bug跟踪系统中提出这个问题时，我承认我实在等不及想知道这个问题的答案。于是我转向求助我最后的Unix编程书籍：The Linux Programming Interface.

![2](/imgs/linux多进程写文件/2.png)

在书中找到我想要的答案。从几个知识点可以推导出这个问题的答案。

对于初学者，**所有的系统调用都是原子性的**。这句话来自TLPI:

> "All system calls are executed atomically. By this, we mean that the kernel guarantees that all of the steps in a system call are completed as a single operation, without being interrupted by another process of thread."

太好了！于是当两个线程或进程在同一时间写同一文件时，我们可以肯定他们的数据不会交叉插入。这使调用write(2)不需要互斥锁，而且还能保证原子性，因为内核已经帮我们做到那个了。

但是还有一个问题，一般写文件时，你需要找到你想插入的具体的位置，然后再进行真正的写入操作。不幸的是，这会涉及到两个系统调用，lseek(2)和write(2).他们各自是原子性的，但是两个操作作为一个整体就不是原子性的。让我们来证明下：

用下面的伪代码表示写文件的过程：

```c
lseek(fd,0,SEEK_END);  //seek to the end of the file
write(fd,"log message",len); //perform the write
```

在多线程环境下，这种操作是不安全的。通过这两个系统调用不能保证原子性，例如下面的情况：

* Thread A seeks to the end of the file
* -- the thread scheduler decides to switch context to Thread B --
* Thread B seeks to the end of the file
* Thread B writes its data
* -- thread scheduler switches context back to Thread A --
* Thread A has already done its lseek, so it doesn't do it again, it just performs its write

d'oh,线程A不再是指向真正的文件末尾,它指向的是此文件的前一个尾部，就是线程B写入信息的位置。当线程A进行写操作时，它会覆盖掉线程B写的信息。

通过这种方法写文件，有可能会丢失数据。

但是还有一种方法。

![3](/imgs/linux多进程写文件/3.png)

为了能保证seek和write两个操作发生的原子性，你可以设置O_APPEND标志，当你在打开一个文件时。然后，任何往文件追加的写操作都能保证是原子性的。这正是我们在写要共享日志文件时想要的情况。Logger和MonoLogger都是用这个标志来打开文件。


```c
open(filename,(FILE::WRONLY | FILE::APPEND))
```

因此MonoLogger可以原子性的追加到日志系统，而不需要互斥信号。

## PIPE\_BUF\_MAX?

在github的issue里面，我提出了一个关于PIPE\_BUF\_MAX的问题。在这里我不想过多讨论它，因为在写入文件方面并不是相关的。本质上而言，当写入目标是一个文件时，write(2)能保证任何大小的写是原子性的。但是当写入一个pipe时，wirte(2)只能当其小于PIPE\_BUF\_MAX时保证原子性。

我用python写了一个测试程序如下：

```python
import multiprocessing
import time

def func(msg):
    f=open("test.log",'a')
    for i in xrange(1000000):
        f.write(msg+'\n')

if __name__ == "__main__":
    cpuCount=multiprocessing.cpu_count()
    print "the number of cpu is %d" %cpuCount
    pool = multiprocessing.Pool(cpuCount)
    for i in xrange(cpuCount):
        msg = "hello %d" %(i)
        pool.apply_async(func,(msg,))
    pool.close()
    pool.join()
    print "sub-process(es) done."
```

测试结果如下：

```
beyond@birdcat-2:~/code/code-python|
⇒  python multi-pool.py
the number of cpu is 4
sub-process(es) done.
beyond@birdcat-2:~/code/code-python|
⇒  cat test.log| wc -l
 4000000
```

测试程序二：

```python
import multiprocessing
import time
import os

def func(msg):
    f=open("test.log",'w')
    f.seek(0,os.SEEK_END)
    for i in xrange(1000000):
        f.write(msg+'\n')

if __name__ == "__main__":
    cpuCount=multiprocessing.cpu_count()
    print "the number of cpu is %d" %cpuCount
    pool = multiprocessing.Pool(cpuCount)
    for i in xrange(cpuCount):
        msg = "hello %d" %(i)
        pool.apply_async(func,(msg,))
    pool.close()
    pool.join()
    print "sub-process(es) done."
```

测试结果：

```
beyond@birdcat-2:~/code/code-python|
⇒  python multi-pool.py
the number of cpu is 4
sub-process(es) done.
beyond@birdcat-2:~/code/code-python|
⇒  cat test.log| wc -l
 1000000
```
