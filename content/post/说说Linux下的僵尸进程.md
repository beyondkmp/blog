---
title: "说说Linux下的僵尸进程"
date: 2015-10-24T00:00:00+0800
lastmod: 2015-10-24T00:00:00+0800
draft: false
keywords: ["Linux","僵尸进程"]
tags:  ["Linux","僵尸进程"]
description: "僵尸进程是怎么产生的，怎么预防僵尸进程的产生"
categories: ["常用技巧"]
tags: ["Linux","僵尸进程"]
---

## 僵尸进程的产生

为了理解僵尸进程是什么和僵尸进程是怎么产生的，我们需要先来理解下在Linux下进程是怎么工作的。

在Linux下当一个进行死掉的时候，它占有的所有资源并不会立即释放。它的进程描述符还停留在内存里面(不过进程描述符仅仅占了一点点内存)。这个时候进程的状态变成EXIT\_ZOMBIE,也被称作为僵尸进程,同时此僵尸进程会向其父里程发送SIGCHLD信号。然后父进程接收到SIGCHLD信号，执行wait()这个系统调用来读取子进程的退出状态和其它信息。Linux下允许父进程从dead process中读取信息的。当wait()执行完后，僵尸进程的所有资源都会从内存中移除。僵尸进程也就被清理干净了。

上段描述的是正常情况下僵尸进程的产生与消亡的过程，由于上面的过程可能发生的太快以至于在系统中根本没有看到僵尸进程的产生。但是，当父进程没有正常的执行wait()系统调用或者就忘了执行，那么它的僵尸子进程的进程描述符会一直停留在内存中，直到管理员自己去把他们清楚掉。

在Linux我们可以用ps和top命令来查看僵尸进程,接下来我们自己来产生一个僵尸进程看看了！

```C
#include<stdio.h>
#include<sys/types.h>
#include<stdlib.h>

int main()
{
    pid_t pid=fork(); // call fork() to fork a child process

    if(pid>0)  //one call, return two value,child process return "0",parent process return child's process id
    {
        printf("This is parent process!!!\n");
        sleep(60);
        printf("after sleeping,and exit!\n");
    }
    else if(pid==0)
    {
        printf("This is the child process,and exit!\n");
        exit(0);
    }
    return 0;
}
```
上述代码中父进程没有调用wait()系统调用函数，当子进程退出的时候就变成僵尸进程一直停留在内存中，我们可以用下述命令来编译运行上述代码，然后再看看系统的进程状态。

```bash
beyond@gcc:~/code/c|⇒  gcc zombie.c
beyond@gcc:~/code/c|⇒  ls
a.out  zombie.c
beyond@gcc:~/code/c|⇒  ./a.out
This is parent process!!!
This is the child process,and exit!
after sleeping,and exit!
```

结果如下:

![僵尸进程]({{IMAGE_PATH}}/僵尸进程/1.png)

我们可以从上图中看到，产生了一个僵尸进程。但是当父进程在60秒后睡来时，我们再将查看系统的进程信息时，又找不到此僵尸进程了。为什么会这样呢？实际上是父进程退出后，它的僵尸子进程会被init进程(进程id为1)领养，init进程调用wait系统调用读取到僵尸进程的信息与状态，并将其完全移除。

## 僵尸进程的危害

僵尸进程并不会用光系统的所有资源(实际上每一个进程都只会使用很小一点的内存来保存其进程描述符)。不过，每一个僵尸进程都会保留并占用一个进程ID号(PID)。Linux系统下进程ID的数目是有限的---> 32位系统下默认是32767。有一种情况是非常危险的，当僵尸进程以非常快的速率积累并生成时，比如一个不正常的服务器软件运行时不断的产生僵尸进程，那么没被使用的进程ID号最终都会分配给僵尸进程，最后会导致正常的进程产生不了。

但是，当只有很少量的僵尸进程停留在内存是没有什么大问题的，它们的存在可能说明父进程在系统中存在相应的bug。

## 远离僵尸进程和杀死僵尸进程

我们不能像杀死一个正常进程一样来杀死一个僵尸进程,即使给僵尸进程发送SIGKILL信号也是没用的，因为僵尸进程已经死掉了，只是剩下一个进程描述符停留在内存中。请记住，我们不需要时时关注并避免僵尸进程，除非有大量的僵尸进程在我们的系统中。少量的僵尸进程是无害的，不过，我们还是可以用下面几种方法来杀死并远离僵尸进程的。

**方法一:**

我们可以手动发送SIGCHLD信号给僵尸进程的父进程。这个信号的作用就是要求父进程执行wait()系统调用并清除它的僵尸进程。发信号的命令是kill命令，下面的pid换成相应的父进程的PID就可以了。

```bash
ps -o ppid= -p zombie_pid  #查找僵尸进程的父进程
kill -s SIGCHLD pid   # 此处的pid应该是父进程的pid
```

**方法二:**

但是，当父进程编写的时候有问题，忽略所有的SIGCHLD信号，那么方法一就不起作用了。那我们只有杀死或者关闭僵尸进程的父进程。当创建僵尸进程的父进程结束后，init进程就会继承僵尸进程，并成为它们的新的父进程。(init进程是Linux在启动时的第一个进程并分配pid为1)。init进程会周期性执行wait()系统调用来清除它的所有的僵尸进程,因此init进程会花费一小部分时间来处理僵尸进程。我们在关闭其父进程后还可以重启此进程。

当子进程死掉的时候，会向其父进程发送SIGCHLD信号，而上面的c代码中父进程是直接忽略了SIGCHLD信号，我们编写程序的时候应该自己增加此信号的处理函数，在处理函数中我们可以调用wait(),这样父进程就会自动清除僵尸进程了。

```c
#include<stdio.h>
#include<sys/types.h>
#include<stdlib.h>
#include<signal.h>
#include<sys/wait.h>
#include<string.h>

sig_atomic_t child_exit_status;

void clean_up_child_process(int signal_num)
{
    //remove child process
    int status;
    wait(&status);

    //save exit status in a global variable
    child_exit_status=status;
}
int main()
{
    //handle SIGCHLD by calling clean_up_child_process
    struct sigaction sigchild_action;
    memset(&sigchild_action,0,sizeof(sigchild_action));
    sigchild_action.sa_handler=&clean_up_child_process;
    sigaction(SIGCHLD,&sigchild_action,NULL);


    //fork a child process
    pid_t c_pid=fork(); // call fork() to fork a child process

    if(c_pid>0)  //one call, return two value,child process return "0",parent process return child's process id
    {
        printf("This is parent process!!!\n");
        sleep(60);
    }
    else if(c_pid==0)
    {
        printf("This is the child process,and exit!\n");
        exit(0);
    }
    else
    {
        printf("fork failed!\n");
    }
    return 0;
}
```

## 参考
1. [How to deep understand the Zombie Process in linux](http://www.itsprite.com/how-to-deep-understand-the-zombie-process-in-linux/)
2. [HTG Explains: What is a Zombie Process on Linux?](http://www.howtogeek.com/119815/htg-explains-what-is-a-zombie-process-on-linux/)
3. [Zombie process](https://en.wikipedia.org/wiki/Zombie_process)
4. [How to get parent PID of a given process in GNU/Linux from command line?](http://superuser.com/questions/150117/how-to-get-parent-pid-of-a-given-process-in-gnu-linux-from-command-line)
