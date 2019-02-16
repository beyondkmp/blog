---
title: "python指定网卡ip进行http访问"
date: 2017-02-20T00:00:00+0800
lastmod: 2017-02-20T00:00:00+0800
draft: false
keywords: ["python","指定网卡ip","http"]
tags:  ["python","指定网卡ip","http"]
description: "当有多个网卡ip时,要指定ip进行http访问"
categories: ["python学习"]
tags: ["python","指定网卡ip","http"]
---

python3 提供的http底层库已经支持指定souce_addr, 这个参数是一个元组类型,包括ip与端口,端口为0时系统会随机指定端口

```python
import json
import http.client
import urllib.parse


def http_client():
    params = {"a": "123"}
    headers = {"Content-type": "application/json"}
    conn = http.client.HTTPConnection(
        "www.baidu.com", 5005, source_address=("1.2.3.4", 0))
    conn.request("POST", "/test/nihao", json.dumps(params), headers)
    response = conn.getresponse()
    print(response.status, response.reason)
    data = response.read().decode()
    print(data)
    conn.close()


def main():
    http_client()


if __name__ == '__main__':
    main()
```

## 参考
1. [http.client — HTTP protocol client](https://docs.python.org/3/library/http.client.html)
