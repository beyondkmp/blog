---
title: "go语言中高效字符串拼接"
date: 2016-03-06T00:00:00+0800
lastmod: 2016-03-06T00:00:00+0800
draft: false
keywords: ["golang","高效","字符串拼接"]
tags:  ["golang","高效","字符串拼接"]
description: "在相同的情况下比较了三种常见的字符串拼接的方法的时间效率"
categories: ["golang学习"]
tags: ["golang","高效","字符串拼接"]
---

目前主要有三种方法：

* 直接用"+="操作符，直接将多个字符串拼接。最直观的方法，不过当数据量非常大时用这种拼接访求是非常低效的。

* 用字符串切片([]string)装载所有要拼接的字符串，最后使用strings.Join()函数一次性将所有字符串拼接起来。在数据量非常大时，这种方法的效率也还可以的。

* 利用Buffer(Buffer是一个实现了读写方法的可变大小的字节缓冲)，将所有的字符串都写入到一个Buffer变量中，最后再统一输出。这种方法的效率最变态，达到“日天”级别。


以下是我写的一段go语法代码对三种方法进行验证，同时拼接10万条字符串，方法一用时大概到11-12秒，方法二用时16-17毫秒，方法三用时4毫秒。

```go
package main

import (
	"bytes"
	"fmt"
	"strings"
	"time"
)

func main() {
	var buffer bytes.Buffer

	s := time.Now()
	for i := 0; i < 100000; i++ {
		buffer.WriteString("test is here\n")
	}

	//fmt.Println("拼接后的结果为-->", buffer.String())
	sb := buffer.String()
	e := time.Now()
	fmt.Println("taked time is ", e.Sub(s).Seconds())

	s = time.Now()
	str := ""
	for i := 0; i < 100000; i++ {
		str += "test is here\n"
	}

	e = time.Now()
	fmt.Println("taked time is ", e.Sub(s).Seconds())
	if sb == str {
		fmt.Println("is same")
	}

	s = time.Now()
	var sl []string
	for i := 0; i < 100000; i++ {
		sl = append(sl, "test is here\n")
	}
	sc := strings.Join(sl, "")
	e = time.Now()
	fmt.Println("taked time is", e.Sub(s).Seconds())
	if str == sc {
		fmt.Println("is same too")
	}

}
```

程序的输出结果：

![result]({{IMAGE_PATH}}/go字符串拼接/2.png)


## 参考
1. [golang 高效字符串拼接](http://studygolang.com/articles/3427)
