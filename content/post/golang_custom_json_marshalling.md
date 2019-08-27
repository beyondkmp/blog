---
title: "[翻译]Golang自定义json转化格式"
date: 2019-08-11T13:06:23+0800
lastmod: 2019-08-27T12:19:46+0800
draft: false
keywords: ["golang","json","custom json marshalling"]
description: "custom json marshalling of golang"
tags: ["golang","json","custom json marshalling"]
categories: ["golang"]
author: "beyondkmp"

---

Golang标准库里面有encoding/json可以非常方便的将结构体转化成json数据, 具体例子如下:

```go
package main

import (
    "encoding/json"
    "os"
    "time"
)

type MyUser struct {
    ID       int64     `json:"id"`
    Name     string    `json:"name"`
    LastSeen time.Time `json:"lastSeen"`
}

func main() {
    _ = json.NewEncoder(os.Stdout).Encode(
        &MyUser{1, "Ken", time.Now()},
    )
}
```

<!--more-->

Output:

```json
{"id":1,"name":"Ken","lastSeen":"2009-11-10T23:00:00Z"}
```

如果我们想要修改结构体里面的某项的显示方式，比如，把`LastSee`改成时间戳的显示方式。最简单的解决办法是使用一个辅助的结构体，并将正式的格式填充到这个辅助结构体里面。

```go
func (u *MyUser) MarshalJSON() ([]byte, error) {
    return json.Marshal(&struct {
        ID       int64  `json:"id"`
        Name     string `json:"name"`
        LastSeen int64  `json:"lastSeen"`
    }{
        ID:       u.ID,
        Name:     u.Name,
        LastSeen: u.LastSeen.Unix(),
    })
}
```

这样修改是没有问题的。不过如果结构体里面有很多值的话，这样写起来就会感觉很累赘的。如果我们可以把原始结构体嵌套到辅助的结构体里面，并且辅助结构体可以直接继承原始结构体。

```go
func (u *MyUser) MarshalJSON() ([]byte, error) {
    return json.Marshal(&struct {
        LastSeen int64 `json:"lastSeen"`
        *MyUser
    }{
        LastSeen: u.LastSeen.Unix(),
        MyUser:   u,
    })
}
```

上面代码的主要问题是辅助结构体也会继承原始结构体的`MarshalJSON`方法，这样会导致了限入死循环。解决方法就是给原始类型别名化，这样别名会有所有属性，但是不会有任何方法的。

```go
func (u *MyUser) MarshalJSON() ([]byte, error) {
    type Alias MyUser
    return json.Marshal(&struct {
        LastSeen int64 `json:"lastSeen"`
        *Alias
    }{
        LastSeen: u.LastSeen.Unix(),
        Alias:    (*Alias)(u),
    })
}
```

上面的方法同样适用于实现`UnmarshalJSON`方法.

```go
func (u *MyUser) UnmarshalJSON(data []byte) error {
    type Alias MyUser
    aux := &struct {
        LastSeen int64 `json:"lastSeen"`
        *Alias
    }{
        Alias: (*Alias)(u),
    }
    if err := json.Unmarshal(data, &aux); err != nil {
        return err
    }
    u.LastSeen = time.Unix(aux.LastSeen, 0)
    return nil
}
```

## 参考
1. [Custom JSON Marshalling in Go](http://choly.ca/post/go-json-marshalling/)
