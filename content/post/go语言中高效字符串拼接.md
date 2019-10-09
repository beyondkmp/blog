---
title: "go语言中高效字符串拼接"
date: 2016-03-06T00:00:00+0800
lastmod: 2019-10-10T00:55:33+0800
draft: false
keywords: ["golang","高效","字符串拼接"]
tags:  ["golang","高效","字符串拼接"]
description: "在相同的情况下比较了三种常见的字符串拼接的方法的时间效率"
categories: ["golang"]
tags: ["golang","高效","字符串拼接"]
---

目前主要有四种方法：

* 直接用"+="操作符，直接将多个字符串拼接。最直观的方法，不过当数据量非常大时用这种拼接访求是非常低效的。

* 用字符串切片([]string)装载所有要拼接的字符串，最后使用strings.Join()函数一次性将所有字符串拼接起来。在数据量非常大时，这种方法的效率也还可以的。

* 利用Buffer(Buffer是一个实现了读写方法的可变大小的字节缓冲)，将所有的字符串都写入到一个Buffer变量中，最后再统一输出。这种方法的效率最变态，达到“日天”级别。

* 使用strings.Builder, 最后输出字符串不用再转化一次

```golang
package main

import (
        "bytes"
        "strings"
        "testing"
)

const TEST = "test is here\n"

func AddStringByPlus(N int) string {
        var result string
        for i := 0; i < N; i++ {
                result += TEST
        }
        return result
}

func AddStringByJoin(N int) string {
        var strs []string
        for i := 0; i < N; i++ {
                strs = append(strs, TEST)
        }

        return strings.Join(strs, "")
}

func AddStringByBuffer(N int) string {
        var buffer bytes.Buffer
        for i := 0; i < N; i++ {
                buffer.WriteString(TEST)
        }
        return buffer.String()
}

func AddStringByStringBuilder(N int) string {
        var builder strings.Builder
        for i := 0; i < N; i++ {
                builder.WriteString(TEST)
        }
        return builder.String()
}

var TESTNUM = 100

func BenchmarkStringPlus(b *testing.B) {
        for i := 0; i < b.N; i++ {
                AddStringByPlus(TESTNUM)
        }
}

func BenchmarkStringJoin(b *testing.B) {
        for i := 0; i < b.N; i++ {
                AddStringByJoin(TESTNUM)
        }
}

func BenchmarkStringBuffer(b *testing.B) {
        for i := 0; i < b.N; i++ {
                AddStringByBuffer(TESTNUM)
        }
}

func BenchmarkStringBuilder(b *testing.B) {
        for i := 0; i < b.N; i++ {
                AddStringByBuffer(TESTNUM)
        }
}

```

**测试结果**

```shell
go test -bench=. -run=NONE -benchmem
goos: linux
goarch: amd64
pkg: hello
BenchmarkStringPlus-4             100000             22067 ns/op           69440 B/op         99 allocs/op
BenchmarkStringJoin-4             500000              2515 ns/op            5488 B/op          9 allocs/op
BenchmarkStringBuffer-4          1000000              2270 ns/op            6544 B/op          7 allocs/op
BenchmarkStringBuilder-4         1000000              2469 ns/op            6544 B/op          7 allocs/op
PASS
ok      hello   8.758s

```

## 参考
1. [golang 高效字符串拼接](http://studygolang.com/articles/3427)
