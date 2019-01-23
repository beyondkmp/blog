---
title: "修复升级xcode后zsh崩溃"
date: 2019-01-23T10:33:23+0800
lastmod: 2019-01-23T10:33:23+0800
draft: false
keywords: ["升级xcode","zsh崩溃","iTerm2"]
description: "升级xcode后zsh崩溃了"
tags: ["升级xcode","zsh崩溃","iTerm2"]
categories: ["mac"]
author: "beyondkmp"

---

昨天升级了xcode, 好久没有升级了，而且zsh也没有升级。一升级完，iTerm2和系统自带的终端就直接崩溃了。报错如下：

```
dyld: Library not loaded: /usr/local/opt/gdbm/lib/libgdbm.4.dylib
```

![zsh_crash](/imgs/zsh_crash.jpg)

<!--more-->

网上的解决方法都是升级zsh就可以解决，参考1的就是这样说。但是现在打开终端就直接如上图了，根本运行不了命令了。

幸好自带终端还留有一手，可以在属性里面直接修改默认shell,如下图，修改为系统自带的bash，就可以运行命令了。

![terminal_shell](/imgs/terminal_shell.png)

最后是更新zsh了，不过这个也出幺蛾子了。先是说已经安装了，然后删除重装，还是不行。最后总结命令如下：

```bash
brew uninstall zsh
brew install zsh
brew unlink zsh
brew link zsh
```


## 参考

1. [dyld: Library not loaded: /usr/local/opt/gdbm/lib/libgdbm.4.dylib](https://github.com/robbyrussell/oh-my-zsh/issues/7033)
2. [Error loading library in mac terminal](https://stackoverflow.com/questions/24103888/error-loading-library-in-mac-terminal)
