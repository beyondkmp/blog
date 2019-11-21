---
title: "golang http的代码收集"
date: 2019-11-21T16:53:18+0800
lastmod: 2019-11-21T16:53:18+0800
draft: false
keywords: ["golang http"]
description: "golang http代码收集"
tags: ["golang http"]
categories: ["golang"]
author: "beyondkmp"

---

# golang http server

```go

```

# golang http post with json

```go
package main

import (
        "bytes"
        "encoding/json"
        "fmt"
        "io/ioutil"
        "net/http"
        "time"
)

// User is user struct
type User struct {
        ID      string `json:"id"`
        Balance uint64 `json:"balance"`
}

func main() {
        url := "http://restapi3.apiary.io/notes"
        user := User{ID: "USER123", Balance: 1234}
        jsonValue, err := json.Marshal(user)
        if err != nil {
                panic(err)
        }

        req, err := http.NewRequest("POST", url, bytes.NewBuffer(jsonValue))
        req.Header.Set("Content-Type", "application/json")

        client := &http.Client{
                Timeout: time.Second * 5,
        }

        resp, err := client.Do(req)
        if err != nil {
                panic(err)
        }
        defer resp.Body.Close()

        fmt.Println("response Status:", resp.Status)
        fmt.Println("response Headers:", resp.Header)
        body, _ := ioutil.ReadAll(resp.Body)
        fmt.Println("response Body:", string(body))
}

```

<!--more-->
