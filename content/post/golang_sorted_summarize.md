---
title: "golang排序实践"
date: 2019-08-29T14:15:17+0800
lastmod: 2019-08-29T14:15:17+0800
draft: false
keywords: ["golang sorted","sort slice"]
description: "golang的排序总结"
tags: ["golang sorted","sort slice"]
categories: ["golang"]
author: "beyondkmp"

---

# 对int, float64, string列表进行排序

可以直接使用库函数, 进行升序排序, 目前有以下三个库函数`sort.Ints`,`sort.Strings`,`sort.Float64s`

```go
a := []int{1, 5, 4, 3, 2}
sort.Ints(a)
fmt.Println(a)
```
<!--more-->

# 使用自定义比较器

从go1.8后，可以直接使用`sort.Slice`, 使用自定义的比较函数`less(i, j int) bool`. 如果想稳定排序，可以使用`sort.SliceStable`

sort.Slice内部使用的接口和反射来实现的，效率上面会打一定的折扣。

```go
package main

import (
        "fmt"
        "sort"
)

type User struct {
        Name string
        Age  int
}

func main() {
        users := []User{
                User{"test", 12},
                User{"james", 32},
                User{"bob", 18},
                User{"alice", 16},
        }

        sort.Slice(users, func(i, j int) bool {
                return users[i].Age < users[j].Age
        })

        fmt.Println(users)
}
```

# 实现排序接口

1. 使用`sort.Sort`和`sort.Stable`函数
2. 只要实现了`sort.Interface`接口的集合，都可以通过上面的两个函数进行排序。

```go
type Interface interface {
        // Len is the number of elements in the collection.
        Len() int
        // Less reports whether the element with
        // index i should sort before the element with index j.
        Less(i, j int) bool
        // Swap swaps the elements with indexes i and j.
        Swap(i, j int)
}
```

```go
package main

import (
        "fmt"
        "sort"
)

type User struct {
        Name string
        Age  int
}

type ByAge []User

func (a ByAge) Len() int {
        return len(a)
}

func (a ByAge) Less(i, j int) bool {
        return a[i].Age < a[j].Age
}

func (a ByAge) Swap(i, j int) {
        a[i], a[j] = a[j], a[i]
}

func main() {
        users := []User{
                User{"test", 12},
                User{"james", 32},
                User{"bob", 18},
                User{"alice", 16},
        }

        sort.Sort(ByAge(users))
        fmt.Println(users)
}
```

# 性能对比

Slice vs Sorting Type (of Slice) with Sort implementation

```go

package main

import (
        "sort"
        "testing"
)

type User struct {
        Name string
        Age  int
}

var TestCases = []User{
        {"alice", 9},
        {"james", 35},
        {"wade", 15},
        {"test", 51},
        {"beyond", 25},
        {"brand", 59},
        {"bob", 10},
        {"twitter", 15},
        {"google", 22},
        {"facebook", 91},
}

// Example One - Sorting Slice Directly
// somehow - slowest way to sort it.
func SortSlice(users []User) {
        sort.Slice(users, func(i, j int) bool {
                return users[i].Name < users[j].Name
        })
}

func BenchmarkSlice(b *testing.B) {
        tmp := make([]User, len(TestCases))
        for i := 0; i < b.N; i++ {
                copy(tmp, TestCases)
                SortSlice(tmp)
        }
}

// Example Two - Sorting Slice Directly
// much faster performance
type ByName []User

// Sort interface implementation
func (n ByName) Less(i, j int) bool { return n[i].Name < n[j].Name }
func (n ByName) Len() int           { return len(n) }
func (n ByName) Swap(i, j int)      { n[i], n[j] = n[j], n[i] }

func SortStruct(users []User) {
        sort.Sort(ByName(users))
}

func BenchmarkStruct(b *testing.B) {
        tmp := make([]User, len(TestCases))
        for i := 0; i < b.N; i++ {
                copy(tmp, TestCases)
                SortStruct(tmp)
        }
}
```

测试结果:

```go
$ go test -bench=.
goos: linux
goarch: amd64
pkg: hello
BenchmarkSlice-2         2000000               976 ns/op
BenchmarkStruct-2        3000000               401 ns/op
PASS
ok      hello   4.564s
```

# 参考
1. [The 3 ways to sort in Go](https://yourbasic.org/golang/how-to-sort-in-go/)

