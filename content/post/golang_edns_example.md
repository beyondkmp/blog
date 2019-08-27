---
title: "golang edns实战"
date: 2019-08-25T14:23:54+0800
lastmod: 2019-08-27T11:00:42+0800
draft: false
keywords: ["golang","edns","使用","miekg dns"]
description: ""
tags: ["golang","edns"]
categories: ["golang"]
author: "beyondkmp"

---

目前需要从不同的地方访问一个域名的CNAME, 比如从国内和海外两个不同的地方访问, 并查看这个CNAME是不是设置正确。

<!--more-->

```go
package main

import (
    "context"
    "flag"
    "fmt"
    "net"
    "time"

    "github.com/miekg/dns"
)

var (
    TIMEOUT = 5 * time.Second
)

type edns struct {
    client *dns.Client
    server string
}

func NewEdns() (*edns, error) {
    server := "223.5.5.5"
    port := "53"

    c := new(dns.Client)
    edns := &edns{
        client: c,
        server: net.JoinHostPort(server, port),
    }
    return edns, nil
}

func (e *edns) ednsMsg(domain string, clientIP net.IP) *dns.Msg {
    m := new(dns.Msg)
    m.SetQuestion(dns.Fqdn(domain), dns.TypeCNAME)
    m.RecursionDesired = true

    o := &dns.OPT{
        Hdr: dns.RR_Header{
            Name:   ".",
            Rrtype: dns.TypeOPT,
        },
    }

    ed := &dns.EDNS0_SUBNET{
        Code:          dns.EDNS0SUBNET,
        Address:       clientIP,
        Family:        1,
        SourceNetmask: net.IPv4len * 8,
    }

    o.Option = append(o.Option, ed)
    m.Extra = append(m.Extra, o)
    return m
}

func (e *edns) Query(domain string, clientIP net.IP) ([]string, error) {
    ctx, cancle := context.WithTimeout(context.Background(), TIMEOUT)
    defer cancle()

    m := e.ednsMsg(domain, clientIP)
    r, _, err := e.client.ExchangeContext(ctx, m, e.server)

    if r == nil {
        return nil, err
    }

    if r.Rcode != dns.RcodeSuccess {
        return nil, fmt.Errorf("invalid answer name")
    }

    var result []string
    for _, a := range r.Answer {
        switch ans := a.(type) {
        case *dns.CNAME:
            result = append(result, ans.Target)
        }
    }
    return result, nil
}

func main() {
    domain := flag.String("d", "www.baidu.com", "domain")
    clientIP := flag.String("c", "38.143.9.29", "client ip")

    flag.Parse()

    ed, err := NewEdns()
    if err != nil {
        panic(err)
    }

    cnames, err := ed.Query(*domain, net.ParseIP(*clientIP))
    if err != nil {
        panic(err)
    }

    fmt.Printf("domain: %s, cnames: %v\n", *domain, cnames)
}
```

output:

```
$ ./dns -d  test.ictim2020.com -c 58.22.114.38
domain: test.ictim2020.com, cnames: [www.baidu.com.]
$ ./dns -d  test.ictim2020.com -c 1.1.1.1
domain: test.ictim2020.com, cnames: [www.google.com.]
```
