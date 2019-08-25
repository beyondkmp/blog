---
title: "golang ends example"
date: 2019-08-25T14:23:54+0800
lastmod: 2019-08-25T14:23:54+0800
draft: false
keywords: ["golang","edns"]
description: ""
tags: ["golang","edns"]
categories: ["golang"]
author: "beyondkmp"

---

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
        MAINLANDIP = net.ParseIP("120.253.225.34")
        OVERSEA    = net.ParseIP("38.143.9.29")
        TIMEOUT    = 5 * time.Second
)

type edns struct {
        client    *dns.Client
        server    string
        isOversea bool
}

func NewEdns() (*edns, error) {
        config, err := dns.ClientConfigFromFile("/etc/resolv.conf")
        if err != nil {
                return nil, fmt.Errorf("cannot read config from /etc/resolv.conf")
        }

        c := new(dns.Client)
        edns := &edns{
                client: c,
                server: net.JoinHostPort(config.Servers[0], config.Port),
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

func (e *edns) QueryEqual(domain, cname string, isOversea bool) (bool, error) {
        ctx, cancle := context.WithTimeout(context.Background(), TIMEOUT)
        defer cancle()

        clientIP := MAINLANDIP
        if isOversea {
                clientIP = OVERSEA
        }

        m := e.ednsMsg(domain, clientIP)
        r, _, err := e.client.ExchangeContext(ctx, m, e.server)

        if r == nil {
                return false, err
        }

        if r.Rcode != dns.RcodeSuccess {
                return false, fmt.Errorf("invalid answer name")
        }

        for _, a := range r.Answer {
                switch ans := a.(type) {
                case *dns.CNAME:
                        fmt.Println(ans.Target)
                        if ans.Target == cname+"." {
                                return true, nil
                        }
                }
        }
        return false, nil
}

func main() {
        domain := flag.String("d", "www.baidu.com", "domain")
        cname := flag.String("c", "www.a.shifen.com", "cname")
        isOversea := flag.Bool("o", false, "is oversea")
        flag.Parse()

        ed, err := NewEdns()
        if err != nil {
                panic(err)
        }

        equal, err := ed.QueryEqual(*domain, *cname, *isOversea)
        if err != nil {
                panic(err)
        }

        fmt.Printf("domain: %s, cname: %s  equal: %v\n", *domain, *cname, equal)
}
```
<!--more-->
