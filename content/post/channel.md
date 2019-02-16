---
title: "Golang Channel"
date: 2019-01-15T01:10:20+0800
lastmod: 2019-01-15T01:10:20+0800
draft: false
keywords: ["golang channel"]
description: "记录常用的golang channel"
tags: ["golang channel"]
categories: ["golang"]
author: "beyondkmp"

---

# channel基础语法
# channel


<!--more-->

### 读取关闭的无缓存通道

读取关闭后的无缓存通道，不管通道中是否有数据，返回值都为0和false。

```go
package main

import (
        "fmt"
        "time"
)

func main() {
        done := make(chan int)
        go func() {
                done <- 1
        }()

        time.Sleep(5 * time.Second)
        close(done)

        for i := 1; i <= 3; i++ {
                t, ok := <-done
                fmt.Println(i, ":", t, ok)
        }
}
```

运行结果:

```
1 : 0 false
2 : 0 false
3 : 0 false
```


### 读取关闭的有缓存通道

读取关闭后的有缓存通道，将缓存数据读取完后，再读取返回值为0和false。

```go
package main

import (
        "fmt"
        "time"
)

func main() {
        done := make(chan int, 1)
        go func() {
                done <- 1
        }()

        time.Sleep(5 * time.Second)
        close(done)

        for i := 1; i <= 3; i++ {
                t, ok := <-done
                fmt.Println(i, ":", t, ok)
        }
}

```

运行结果:

```
1 : 1 true
2 : 0 false
3 : 0 false
```


4、range遍历通道： 
通道写完后，必须关闭通道，否则range遍历会出现死锁。

## 参考
1. [Go 通道（chan）关闭和后续读取操作](https://blog.csdn.net/Tovids/article/details/77867284)
