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
package main

import (
        "fmt"
        "io/ioutil"
        "net/http"
)

func hello(w http.ResponseWriter, req *http.Request) {
        body, err := ioutil.ReadAll(req.Body)
        if err != nil {
                panic(err)
        }
        fmt.Println(string(body))

        fmt.Fprintf(w, "hello\n"+string(body))
}

func main() {
        http.HandleFunc("/hello", hello)
        http.ListenAndServe(":8090", nil)
}
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

# golang doing a GET request and building the Querystring

```go
package main

import (
        "fmt"
        "io/ioutil"
        "log"
        "net/http"
        "time"
)

func main() {
        url := "http://127.0.0.1:8090/hello"
        timeout := time.Duration(50 * time.Millisecond) //超时时间50ms
        client := &http.Client{
                Timeout: timeout,
        }

        req, err := http.NewRequest("GET", url, nil)
        if err != nil {
                log.Fatal(err)
        }

        req.Header.Set("Content-Type", "application/x-www-form-urlencoded")
        req.Header.Set("Cookie", "name=zhang")

        q := req.URL.Query()
        q.Add("domain", "www.baidu.com")
        req.URL.RawQuery = q.Encode()

        response, err := client.Do(req)
        if err != nil {
                log.Fatal(err)
        }
        defer response.Body.Close()

        body, err := ioutil.ReadAll(response.Body)
        if err != nil {
                log.Fatal(err)
        }
        fmt.Println(string(body))
}
```
