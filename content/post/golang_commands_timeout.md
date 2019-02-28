---
title: "Golang_commands_timeout"
date: 2019-02-25T15:36:27+0800
lastmod: 2019-02-25T15:36:27+0800
draft: true
keywords: []
description: ""
tags: []
categories: []
author: "beyondkmp"

---

在go中，运行命令，一般都是使用`exec.Command`, 不过这个函数没有超时时间，如果运行命令阻塞了，就会阻塞整个程序，进而导致程序不退出了。

在go1.7之前，context库没有进入标准库，一般的做法是使用channels和select.

```go
package main

import (
    "bytes"
    "fmt"
    "os/exec"
    "time"
)

func main() {
    cmd := exec.Command("ping", "-c 2", "-i 1", "8.8.8.8")
    var buf bytes.Buffer
    cmd.Stdout = &buf
    cmd.Start()

    done := make(chan error)
    go func() { done <- cmd.Wait() }()

    timeout := time.After(2 * time.Second)

    select {
    case <-timeout:
        cmd.Process.Kill()
        fmt.Println("Command timed out")
    case err := <-done:
        fmt.Println("Output:", buf.String())
        if err != nil {
            fmt.Println("Non-zero exit code:", err)
        }
    }
}

```

<!--more-->

```go
package main

import (
    "context"
    "fmt"
    "os/exec"
    "time"
)

func main() {
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel() // The cancel should be deferred so resources are cleaned up

    cmd := exec.CommandContext(ctx, "ping", "-c 4", "-i 1", "8.8.8.8")
    out, err := cmd.Output()
    if ctx.Err() == context.DeadlineExceeded {
        fmt.Println("Command timed out")
        return
    }

    fmt.Println("Output:", string(out))
    if err != nil {
        fmt.Println("Non-zero exit code:", err)
    }
}

```



