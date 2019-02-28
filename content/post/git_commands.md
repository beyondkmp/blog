---
title: "基础git命令收集"
date: 2019-02-28T15:07:41+0800
lastmod: 2019-02-28T15:07:41+0800
draft: false
keywords: ["git命令"]
description: "个人常用git命令收集"
tags: ["git"]
categories: ["运维人生"]
author: "beyondkmp"

---

## 打标签

### 列出所有标签

```
git tag
```

### 列出特定标签

```
git tag -l 'v1.8.5*'
```

### 显示标签信息

```
git show v1.5
```

### 创建标签

```
# 轻量标签(很像一个不会改变的分支)
git tag v1.4-lw

# 附注标签
git tag -a v1.4 -m 'my version 1.4'
```

### 后期打标签

```
git tag -a v1.2 9fceb02
```

### 上传标签

```
git push origin --tags
```

### 删除标签

```
# 本地删除
git tag -d v1.4-lw

# 远程删除( git push <remote> :refs/tags/<tagname> )
git push origin :refs/tags/v1.4-lw
```



<!--more-->



## 参考

1. [Git 基础 - 打标签](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)
2. [Git使用备忘](https://blog.fatedier.com/2014/10/16/git-use-for-remind/)
