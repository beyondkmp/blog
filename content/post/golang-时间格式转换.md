---
title: "golang时间格式转换"
date: 2017-01-08T00:00:00+0800
lastmod: 2017-01-08T00:00:00+0800
draft: false
keywords: ["golang","时间格式","转换"]
tags:  ["golang","时间格式","转换"]
description: "个人总结的golang语言时间标准库的常用方法"
categories: ["golang"]
tags: ["golang","时间格式","转换"]
---

1. 将YYYYmmddHHDD类型的时间转化成unix时间戳

    ```go
    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        stringTime := "201601010000"
        tmpTime, _ := time.ParseInLocation("200601021504", stringTime, time.Local)
        unixTime := tmpTime.Unix()
        fmt.Printf("The Unix time of %v is %v\n", stringTime, unixTime)
    }
    ```
2. 将unix时间戳转化成YYYYmmddHHDD格式的时间

    ```go
    package main

    import (
        "fmt"
        "strconv"
        "time"
    )

    func main() {
        i, err := strconv.ParseInt("1405544146", 10, 64)
        if err != nil {
            panic(err)
        }
        tm := time.Unix(i, 0)
        fmt.Println(tm.Format("200601021504"))
    }
    ```
