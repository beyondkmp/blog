---
title: "nginx配置参考"
date: 2019-02-22T21:19:44+0800
lastmod: 2019-02-25T14:51:17+0800
draft: false
keywords: ["nginx config","hsts"]
description: "nginx常用配置"
tags: ["nginx https","hsts"]
categories: ["运维人生"]
author: "beyondkmp"

---

## nginx编译安装

### 下载源码

1. 下载官方nginx版本[Stable version](http://nginx.org/download/nginx-1.14.2.tar.gz)
2. 下载openssl1.1.1版本，支持tls1.3,[openssl-1.1.1a.tar.gz](https://www.openssl.org/source/openssl-1.1.1a.tar.gz)

### 编译安装

在configure里面加上tls1.3, 开启httpv2模块

```bash
tar -xvf nginx-1.1.4.2.tar.gz
tar -xvf openssl-1.1.1a.tar.gz
cd nginx-1.1.4.2
./configure  --prefix=/usr/local/nginx \
--with-stream \
--with-stream_ssl_preread_module \
--with-http_v2_module \ 
--with-http_ssl_module  \
--with-pcre  \
--with-openssl=/tmp/openssl-1.1.1a \
--with-openssl-opt='enable-tls1_3'
make -j 4
make install
```

<!--more-->
## nginx中HTTPS配置

```nginx
server {
        listen 80;
        server_name beyondkmp.com;
        if ($scheme != "https") {
            return 301 https://beyondkmp.com$request_uri;
        }
}
server {
    listen       443 ssl http2;
    server_name  beyondkmp.com;

    client_max_body_size 50M;
    keepalive_timeout  300;

    ssl_prefer_server_ciphers On;
    ssl_certificate /etc/letsencrypt/live/beyondkmp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/beyondkmp.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/beyondkmp.com/chain.pem;
    #ssl_session_cache shared:SSL:128m;
    add_header Strict-Transport-Security "max-age=31557600; includeSubDomains";

    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_session_cache shared:le_nginx_SSL:1m;
    ssl_session_timeout 1440m;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

    ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305 ECDHE-RSA-CHACHA20-POLY1305 ECDHE-ECDSA-AES128-GCM-SHA256 ECDHE-ECDSA-AES256-GCM-SHA384 ECDHE-ECDSA-AES128-SHA ECDHE-ECDSA-AES256-SHA ECDHE-ECDSA-AES128-SHA256 ECDHE-ECDSA-AES256-SHA384 ECDHE-RSA-AES128-GCM-SHA256 ECDHE-RSA-AES256-GCM-SHA384 ECDHE-RSA-AES128-SHA ECDHE-RSA-AES128-SHA256 ECDHE-RSA-AES256-SHA384 DHE-RSA-AES128-GCM-SHA256 DHE-RSA-AES256-GCM-SHA384 DHE-RSA-AES128-SHA DHE-RSA-AES256-SHA DHE-RSA-AES128-SHA256 DHE-RSA-AES256-SHA256 EDH-RSA-DES-CBC3-SHA";

    location / {
        root   html/hugo-beyondkmp/public;
        index  index.html index.htm;

        if (!-e $request_filename) {
            rewrite ^/(.*) /index.html last;
            break;
        }
    }
}


```

### 申请let's encrypt免费证书

1. 下载certbot-auto脚本，并生成证书

    ```bash
    wget https://raw.githubusercontent.com/certbot/certbot/master/certbot-auto
    chmod +x certbot-auto
    cp certbot-auto /usr/local/bin
    /usr/local/bin/certbot-auto certonly -d beyondkmp.com -d www.beyondkmp.com  --standalone
    ```

3. 证书过期前手动运行下面命令更新证书

    ```bash
    /usr/local/bin/certbot-auto renew --force-renewal
    ```

4. 加入crontab每月自动更新证书

    ```bash
    0 9 1 * * systemctl stop nginx; /usr/local/bin/certbot-auto renew --force-renewal;systemctl start nginx
    ```


## nginx中HSTS配置

```nginx
add_header Strict-Transport-Security "max-age=31557600; includeSubDomains";
```

## 参考

1. [Nginx Quick Reference](https://github.com/trimstray/nginx-quick-reference)
