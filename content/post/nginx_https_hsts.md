---
title: "nginx配置参考"
date: 2019-02-22T21:19:44+0800
lastmod: 2019-11-24T22:43:10+0800
draft: false
keywords: ["nginx config","hsts"]
description: "nginx常用配置"
tags: ["nginx https","hsts"]
categories: ["运维人生"]
author: "beyondkmp"

---

## nginx编译安装

### 下载源码

```shell
wget https://nginx.org/download/nginx-1.16.1.tar.gz
wget https://www.openssl.org/source/openssl-1.1.1d.tar.gz
wget https://www.zlib.net/zlib-1.2.11.tar.gz
wget https://ftp.pcre.org/pub/pcre/pcre-8.43.tar.gz
```

### 安装依赖

```shell
apt install -y gcc perl libperl-dev libgd3 libgd-dev libgeoip1 libgeoip-dev geoip-bin libxml2 libxml2-dev libxslt1.1 libxslt1-dev
```

### 编译安装

在configure里面加上tls1.3, 开启httpv2模块

```bash
tar -xvf nginx-1.16.1.tar.gz
tar -xvf openssl-1.1.1a.tar.gz
tar -xvf zlib-1.2.11.tar.gz
tar -xvf pcre-8.43.tar.gz

useradd nginx

cd nginx-1.16.1
./configure --prefix=/usr/local/nginx \
            --user=nginx \
            --group=nginx \
            --build=Ubuntu \
            --builddir=nginx-1.16.1 \
            --with-select_module \
            --with-poll_module \
            --with-threads \
            --with-file-aio \
            --with-http_ssl_module \
             --with-http_v2_module \
            --with-http_realip_module \
            --with-http_addition_module \
            --with-http_xslt_module=dynamic \
            --with-http_image_filter_module=dynamic \
            --with-http_geoip_module=dynamic \
            --with-http_sub_module \
            --with-http_dav_module \
            --with-http_gunzip_module \
            --with-http_gzip_static_module \
            --with-http_auth_request_module \
            --with-http_random_index_module \
            --with-http_secure_link_module \
            --with-http_degradation_module \
            --with-http_slice_module \
            --with-http_stub_status_module \
            --with-http_perl_module=dynamic \
            --with-perl_modules_path=/usr/share/perl/5.26.1 \
            --with-perl=/usr/bin/perl \
            --with-stream=dynamic \
            --with-stream_ssl_module \
            --with-stream_realip_module \
            --with-stream_geoip_module=dynamic \
            --with-stream_ssl_preread_module \
            --with-compat \
            --with-pcre=../pcre-8.43 \
            --with-pcre-jit \
            --with-zlib=../zlib-1.2.11 \
            --with-openssl=../openssl-1.1.1d \
            --with-openssl-opt='enable-tls1_3'
make
make install
useradd nginx
```

<!--more-->

## systemd管理nginx启动

将下面内容写入`/lib/systemd/system/nginx.service`,

```bash
[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t
ExecStart=/usr/local/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```

开机自启动并启动nginx服务

```shell
# 加载nginx service
systemctl daemon-reload
# 开机启动 
systemctl enable nginx
# 启动nginx服务
systemctl start nginx
```

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
2. [本博客 Nginx 配置之性能篇](https://imququ.com/post/my-nginx-conf-for-wpo.html)
