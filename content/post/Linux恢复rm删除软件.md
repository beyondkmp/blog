---
title: "Linux恢复rm删除文件"
date: 2015-04-08T00:00:00+0800
lastmod: 2015-04-08T00:00:00+0800
draft: false
keywords: ["Linux","rm删除文件"]
tags:  ["Linux","rm删除文件"]
description: "如果不小心用rm删除了一些文件，可以用这个方法恢复"
categories: ["常用技巧"]
tags: ["Linux","rm删除文件"]
---

##步骤一：将磁盘挂载为只读

这个是必须一定得放在第一步来做。如果不做，其它进程写了这个磁盘那就很难恢复了。磁盘规划时一定要做功能分区，/root 、/home、/tmp等等最好不要放在一个区。

比如/data挂载在/dev/sdc1上，用mount命令查看挂载信息：

```
beyond@localhost:~|⇒  mount
/dev/mapper/vg_gcc-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw)
/dev/sda1 on /boot type ext4 (rw)
/dev/mapper/vg_gcc-lv_home on /home type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
gvfs-fuse-daemon on /home/beyond/.gvfs type fuse.gvfs-fuse-daemon (rw,nosuid,nodev,user=beyond)
/dev/sdc1 on /data type ext4 (rw)
```

从`/dev/sdc1 on /data type ext4 (rw)`可知/data挂载在/dev/sdc1

用命令`mount -r -n -o remount /data` 来将/dev/sdc1进行重新挂载，挂载模式为只读

如果运行上面命令出现`mount: /data is busy`,是还有别的进程占用了这个磁盘。将其统统杀掉。。。。

先查看哪些命令在使用/data
`fuser -v -m /data`
从这里可以看到好多进程在使用，赶紧全部干掉啊！！！

干完后再运行重挂载命令：
`mount -r -n -o remount /data`

最后测试下有没有成功改为只读模式：

```
#touch test
#touch: cannot touch 'test': Read-only file system
```

上面的方法有点复杂了，来点简单粗暴的,直接卸载磁盘：

```
# fuser -k /data
umount /data
```
## 步骤二：安装数据恢复软件extundelete

1. 下载软件 [在此下载](wget http://superb-dca2.dl.sourceforge.net/project/extundelete/extundelete/0.2.4/extundelete-0.2.4.tar.bz2)
2. 安装依赖包`yum -y install e2fsprogs e2fsprogs-libs e2fsprogs-devel`
3. 解压并安装

    ```
    tar -xvf extundelete-0.2.4.tar.bz2
    ./confiuger
    make
    sudo make install
    ```

### 步骤三：数据恢复

1. 先查看删除了哪些文件,根据inode查看，先用`ls -id /data`查看inode,再用下面命令查看/data删除的文件：
`extundelete /dev/sdb2 --inode id(此处的id就是上面的id,2一般代码根目录)`

2. 从上面可以得到被删除的文件名，如果文件不是很多可以用下面的命令恢复：

    ```
    开始恢复单个文件
    # extundelete /dev/sdc1 --restore-file test(test是你    从上面查看的文件名，要恢复文件的名字)
    恢复一个目录
    # extundelete /dev/sdc1 --restore-directory test
    ```

    上面都是默认将恢复的数据拷贝到当前目录的RECOVERED_FILES目录中，所以千万要注意你当前文件夹的容量一定要大于要恢复文件的大小，否则就会放不小。

    如果删除的东西很多，最好用全盘恢复，命令如下：
 
        # extundelete /dev/sdc1 --restore-all
 
3. 还可以从时间来恢复和结点来恢复文件

    #### 根据时间来恢复文件
    删除文件的时候应该记的，我们可以恢复从这个时间之后被删除的文件

    ```
    # date -d "2015-04-03 16:00:00" +%s
    1428048000
    # extundelete --restore-all --after "1428048000" /data
    ```

    #### 根据inode来恢复文件
    ```
    ls -id /data
    1234
    # extundelete --restore-inode 1234 /deb/sdc1
    ```

    最后从实际效果来看还是基本都恢复了。最好用restore-all选项来恢复。

从这次血的教训，还是对rm命令启用别名，防止沉默式删除。

```
vim /etc/rc 或者 /etc/zshrc

# do not delete / or prompt if deleting more than 3 files at a time #
alias rm='rm -I --preserve-root'

# 都进行提示 #
alias mv='mv -i'
alias cp='cp -i'
alias ln='ln -i'
```

### 参考：
1. [CentOS 恢复 rm -rf * 误删数据](http://m.oschina.net/blog/222670)
2. [linux rm -rf * 文件恢复记](http://abloz.com/2013/09/12/linux-rm-rf-file-recovery-record.html)
3. [EXT4中恢复使用rm命令误删除的文件](http://www.linuxyunwei.com/2012/08/ext4%E4%B8%AD%E6%81%A2%E5%A4%8D%E4%BD%BF%E7%94%A8rm%E5%91%BD%E4%BB%A4%E8%AF%AF%E5%88%A0%E9%99%A4%E7%9A%84%E6%96%87%E4%BB%B6/)
4. [linux用extundelete恢复ext2、ext3、ext4下rm -rf误删除的数据](http://longgeek.com/2012/11/25/extundelete-recovery-for-linux-ext2-ext3-ext4-rm-rf-accidental-deletion-of-data/)
5. [使用grep恢复被删文件内容](http://coolshell.cn/articles/2822.html)
