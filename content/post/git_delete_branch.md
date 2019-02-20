---
title: "git删除分支"
date: 2019-02-20T11:22:14+0800
lastmod: 2019-02-20T11:22:14+0800
draft: false
keywords: ["git","delete branch"]
description: "了解并掌握git删除分支"
tags: ["git","delete branch"]
categories: ["运维人生"]
author: "beyondkmp"

---
## 删除本地分支

```bash
$ git branch -d branch_name
$ git branch -D branch_name
```

Note: -d只能删除已经完全合并的分支，比如你在master分支上，要删除一个test分支，如果这个分支没有merge到master分支，这里用-d是删除不了的。可以用`git branch --no-merged`来查看没有合并的分支，`git branch --merged`来查看合并的分支

<!--more-->

## 删除远程分支

```bash
$ git push <remote_name> --delete <branch_name>
$ git push <remote_name> :<branch_name>
```

note:

**第一个命令**: `git push origin --delete test`,这样就会删除origin上面的test分支,如果是git2.8版本及之后版本可以直接用`git push origin -d test`

**第二个命令**: 比较巧妙，这里详细讲解下, 一般我们push到远程分支是使用`git push origin test`, 这个会被git展开为：`git push origin refs/heads/test:refs/heads/test`,第一个`refs/heads/test`代表本地的test分支，第二个代表为远程的test分支。如果你想远程的分支取一个更好的名字，可以改为：`git push origin test:awesome`,这样远程的上分支就叫awesome. 这个原理懂了，这个命令其实就是推一个空的本地到分支到远程分支上，这样也相当于删除了。

## 参考
1. [How do I delete a Git branch both locally and remotely?](https://stackoverflow.com/questions/2003505/how-do-i-delete-a-git-branch-both-locally-and-remotely)
2. [3.5 Git 分支 - 远程分支](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)
