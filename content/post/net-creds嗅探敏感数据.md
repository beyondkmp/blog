---
title: "net-creds嗅探敏感数据"
date: 2015-04-30T00:00:00+0800
lastmod: 2015-04-30T00:00:00+0800
draft: false
keywords: ["python代码","net-creds","嗅探敏感数据"]
tags:  ["python代码","net-creds","嗅探敏感数据"]
description: "本文主要介绍net-creds来实现嗅探网络中的敏感数据"
categories: ["代码收集"]
tags: ["python代码","net-creds","嗅探敏感数据"]
---

主要功能从网络接口或者pcap文件中嗅探出密码或者hash。

```
beyond@gcc:~/Desktop/net-creds-master$ python net-creds.py -p ../http.pcap
[192.168.137.2] GET api.bing.com/asjson.aspx?query=172&form=APMCS2
[192.168.137.2] GET api.bing.com/asjson.aspx?query=172.16.69.88/&form=APMCS2
[192.168.137.2] GET api.bing.com/asjson.aspx?query=172.16.69.88/&form=APMCS2
[192.168.137.2] GET api.bing.com/asjson.aspx?query=172.16.69.88/&form=APMCS2
[192.168.137.2] GET 172.16.69.88/old/
[192.168.137.2] GET 172.16.69.88/wordpress
[192.168.137.2] GET 172.16.69.88/old/
[192.168.137.2] GET 172.16.69.88/old/
[172.16.69.60] GET 172.16.69.88/old/
[172.16.69.60] GET 172.16.69.88/old/wp-content/themes/Invoker/timthumb.php?src=http://172.16.69.88/old/wp-content/...
[172.16.69.60] GET 172.16.69.88/old/wp-content/themes/Invoker/timthumb.php?src=http://172.16.69.88/old/wp-content/...
[172.16.69.60] GET 172.16.69.88/old/wp-login.php
[172.16.69.60] GET 172.16.69.88/old/wp-includes/css/buttons.min.css?ver=4.0.1
[172.16.69.60] GET 172.16.69.88/old/wp-includes/css/dashicons.min.css?ver=4.0.1
[172.16.69.60] GET 172.16.69.88/old/wp-admin/css/login.min.css?ver=4.0.1
[172.16.69.60] GET 172.16.69.88/old/wp-admin/images/wordpress-logo.svg?ver=20131107
[172.16.69.60] POST 172.16.69.88/old/wp-login.php
[172.16.69.60] POST load: 
[172.16.69.60] POST 172.16.69.88/old/wp-login.php
[172.16.69.60] POST load: log=beyond&pwd=qweqweqwe&wp-submit=%E7%99%BB%E5%BD%95&redirect_to=http%3A%2F%2F172.16.69.88%2Fold%2...
[172.16.69.60] POST 172.16.69.88/old/wp-login.php
[172.16.69.60] POST load: 
[172.16.69.60] POST 172.16.69.88/old/wp-login.php
[172.16.69.60] POST load: log=beyond&pwd=qweQWEqwe&wp-submit=%E7%99%BB%E5%BD%95&redirect_to=http%3A%2F%2F172.16.69.88%2Fold%2...
[172.16.69.60] GET 172.16.69.88/old/wp-admin/
[172.16.69.60] GET 172.16.69.88/old/wp-content/languages/zh_CN-administration-screens.css?ver=20111120
[172.16.69.60] GET 172.16.69.88/old/wp-content/themes/Invoker/css/admin-post.css?ver=1.0.1
[172.16.69.60] GET 172.16.69.88/old/wp-admin/load-scripts.php?c=1&load%5B%5D=jquery-core,jquery-migrate,utils,json...
[172.16.69.60] GET 172.16.69.88/old/wp-includes/js/thickbox/thickbox.css?ver=4.0.1
[172.16.69.60] GET 172.16.69.88/old/wp-admin/css/colors/blue/colors.min.css?ver=4.0.1
[172.16.69.60] GET 172.16.69.88/old/wp-admin/load-styles.php?c=1&dir=ltr&load=dashicons,admin-bar,wp-admin,buttons...
[172.16.69.60] GET 172.16.69.88/old/wp-content/themes/Invoker/js/admin-post.js?ver=1.0.1
```
### 代码源地址:[net-creds](https://github.com/DanMcInerney/net-creds)
### 备份地址: [beyondkmp/net-creds](https://github.com/beyondkmp/net-creds)
