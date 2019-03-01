---
title: "json.Marshal 返回不同值: var vs := vs make"
date: 2019-03-01T11:14:02+0800
lastmod: 2019-03-01T11:14:02+0800
draft: false
keywords: ["golang json marshal","var make"]
description: ""
tags: ["var","json marshal"]
categories: ["golang"]
author: "beyondkmp"

---

## 问题

>`var`, `:=`两个定义的slice, 经过json.Marshal输出不同的值。

```go
package main

import (
        "encoding/json"
        "fmt"
)

func main() {
        var u1 []int
        fmt.Println("var define")
        j1, _ := json.Marshal(&u1)
        fmt.Println("origin:", u1)
        fmt.Println("json:", string(j1))

        fmt.Println()

        fmt.Println(":= define")
        u2 := []int{}
        j2, _ := json.Marshal(&u2)
        fmt.Println("origin:", u2)
        fmt.Println("json:", string(j2))
}

```
<!--more-->

**输出结果**

```
var define
origin: []
json: null

:= define
origin: []
json: []

Press ENTER or type command to continue
```

## 改进输出

>使用"%#v"输出更加详细的元素信息

```go
package main

import (
        "encoding/json"
        "fmt"
)

func main() {
        var u1 []int = nil
        fmt.Println("var define")
        j1, _ := json.Marshal(&u1)
        fmt.Printf("detail: %#v\n", u1)
        fmt.Println("json:", string(j1))

        fmt.Println()

        fmt.Println(":= define")
        u2 := []int{}
        j2, _ := json.Marshal(&u2)
        fmt.Printf("detail: %#v\n", u2)
        fmt.Println("json:", string(j2))
}
```

**输出结果**

```
var define
detail: []int(nil)
json: null

:= define
detail: []int{}
json: []
```

## 分析

使用var和`:=`定义了一个空的slice, 使用`fmt.Println`输出的结果是一样的，但是通过详细的输出可以看出还是不同，所以json marshal返回的值是不一样的：`nil`和`[]`.

var定义出来的slice是一个`nil`的slice,也就是说这个类型的指针是空的。而`:=`和`make`出来的是一个**empty**的slice, 它的指针不是空的, 只是header里面的length和capacity是0，ZerothElement指针是空的。具体的差别如下：

```go
// u := []int{} or u := make([]int,0)
sliceHeader{
    Length:        0,
    Capacity:      0,
    ZerothElement: nil,
}

//var u []int
sliceHeader{}
```


这样的接口返回数据格式就会让使用者中招了，之前和第三方公司合作的时候也遇到过这样的问题，文档里面说空会返回[],结果有时就返回null, 搞的程序直接不工作了。

## 疑问

在参考2中有这样一段话:

>As should be clear, an empty slice can grow (assuming it has non-zero capacity), but a nil slice has no array to put values in and can never grow to hold even one element.

明确说明nil的slicce是不能增加和拥有元素的，但是下面代码首先定义一个nil的slice，然后就向这个里面添加了元素并成功了。其实这个append是重新返回一个新的slice给u1, 不是向u1里面添加元素。

```go
var u1 []int
u1 = append(u1, 1)
```

## 参考
1. [JSON.Marshal Difference: var vs make](https://www.reddit.com/r/golang/comments/6viyhw/jsonmarshal_difference_var_vs_make/)
2. [The Go Blog: Arrays, slices (and strings): The mechanics of 'append'](https://blog.golang.org/slices#TOC_11.)
