---
title: "Golang_short_declaration"
date: 2019-03-04T14:37:25+0800
lastmod: 2019-03-04T14:37:25+0800
draft: true
keywords: []
description: ""
tags: []
categories: []
author: "beyondkmp"

---

## 问题

```go
package main

import (
        "fmt"
)

func main() {
        var i int
        nums := []int{1, 2, 3, 4, 8, 5, 6}
        for i, v := range nums {
                if v == 8 {
                        i = i + 10
                        break
                }
        }
        fmt.Println(i)
}
```



<!--more-->
