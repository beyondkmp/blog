---
title: "Coins_change"
date: 2019-02-26T14:08:33+0800
lastmod: 2019-02-26T14:08:33+0800
draft: true
keywords: []
description: ""
tags: []
categories: []
author: "beyondkmp"

---

1. 递归算法


```go

package main

import (
        "fmt"
        "math"
)

func min(a, b int) int {
        if a < b {
                return a
        }
        return b
}

func coinChange(coins []int, amount int) int {
        if amount < 0 {
                return -1
        }
        if amount == 0 {
                return 0
        }

        result := math.MaxInt64
        for _, c := range coins {
                tmp := coinChange(coins, amount-c)
                if tmp >= 0 {
                        result = min(result, tmp+1)
                }
        }

        return result
}

func main() {
        coins := []int{1, 6, 7}
        fmt.Println(coinChange(coins, 30))
}

```

<!--more-->
