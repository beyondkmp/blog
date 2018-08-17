---
title: "在CentOS 7 上搭建 Cisco AnyConnect VPN"
date: 2018-08-16T19:25:21+08:00
lastmod: 2018-08-16T19:25:21+08:00
draft: false
keywords: ["centos7","ocserv","config"]
description: "Install Ocserv in Centos7"
tags: ["Centos","运维"]
categories: ["运维人生"]
author: "beyondkmp"

autoCollapseToc: true

---

ocserv 是一个 OpenConnect SSL VPN 协议服务端，0.3.0 版后兼容使用 AnyConnect SSL VPN 协议的终端。官方主页<http://www.infradead.org/ocserv/>。ocserv 已经在 epel 仓库中提供了，所以可以直接通过 yum 安装

<!--more-->

## 安装 ocserv

```bash
$ yum install epel-release
$ yum install ocserv
```

## 生成证书

[官方文档](http://ocserv.gitlab.io/www/manual.html),具体步骤如下

1. 创建工作文件夹
    
    ```bash
    $ mkdir anyconnect
    $ cd anyconnect
    ```
2. 生成ca证书
    
    ```bash
    $ certtool --generate-privkey --outfile ca-key.pem
    $ cat >ca.tmpl <<EOF
    cn = "VPN CA"
    organization = "Big Corp"
    serial = 1
    expiration_days = 3650
    ca
    signing_key
    cert_signing_key
    crl_signing_key
    EOF
    
    $ certtool --generate-self-signed --load-privkey ca-key.pem \
    --template ca.tmpl --outfile ca-cert.pem
    $ cp ca-cert.pem /etc/pki/ocserv/cacerts
    ```
3. 生成服务器证书

    ```
    $ certtool --generate-privkey --outfile server-key.pem
    $ cat >server.tmpl <<EOF
    cn = "www.baishancloud.com"
    organization = "Baishan"
    serial = 2
    expiration_days = 3650
    encryption_key
    signing_key
    tls_www_server
    EOF
    
    $ certtool --generate-certificate --load-privkey server-key.pem \
    --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem \
    --template server.tmpl --outfile server-cert.pem
    $ cp server-cert.pem /etc/pki/ocserv/public/server.crt
    $ cp server-key.pem /etc/pki/ocserv/private/server.key
    ```

4. 生成本地服务器证书(如果不用证书登录，此步骤可以不做）

    创建 gen-client-cert.sh，并将下面内容复制进去

    ```bash
    #!/bin/bash
    USER=$1
    CA_DIR=$2
    SERIAL=`date +%s`
    certtool --generate-privkey --outfile $USER-key.pem
    cat << _EOF_ >user.tmpl
    cn = "$USER"
    unit = "users"
    serial = "$SERIAL"
    expiration_days = 9999
    signing_key
    tls_www_client
    _EOF_
    certtool --generate-certificate --load-privkey $USER-key.pem --load-ca-certificate $CA_DIR/ca-cert.pem --load-ca-privkey $CA_DIR/ca-key.pem --template user.tmpl --outfile $USER-cert.pem

    openssl pkcs12 -export -inkey $USER-key.pem -in $USER-cert.pem -name "$USER VPN Client Cert" -certfile $CA_DIR/ca-cert.pem -out $USER.p12
EOF 
    ```

    创建用户文件夹并调用 gen-client-cert.sh 生成证书

    ```bash
    $ mkdir user
    $ cd user
    # user 指的是用户名，.. 指的是 ca 证书所在的目录
    $ ../gen-client-cert.sh user ..
    # 按提示设置证书使用密码，或直接回车不设密码
    ```

    最后，通过 http 服务器或其他方式将 user.p12 传输给客户端导入即可

## ocserv配置

1. 配置 ocserv, 配置文件`/etc/ocserv/ocserv.conf`**

    ```bash
    #ocserv支持多种认证方式，这是自带的密码认证，使用ocpasswd创建密码文件
    #ocserv还支持证书认证，可以通过Pluggable Authentication Modules (PAM)使用radius等认证方式
    auth = "plain[passwd=/etc/ocserv/ocpasswd]"
    #指定替代的登录方式，这里使用证书登录作为第二种登录方式
    enable-auth = "certificate"
    #证书路径
    server-cert = /etc/ocserv/server-cert.pem
    server-key = /etc/ocserv/server-key.pem
    #ca路径
    ca-cert = /etc/ocserv/ca-cert.pem
    #从证书中提取用户名的方式，这里提取的是证书中的 CN 字段作为用户名
    cert-user-oid = 2.5.4.3
        
    #最大用户数量
    max-clients = 16
        
    #同一个用户最多同时登陆数
    max-same-clients = 10
        
    #tcp和udp端口
    tcp-port = 4433
    udp-port = 4433
        
    #运行用户和组
    run-as-user = ocserv
    run-as-group = ocserv
        
    #虚拟设备名称
    device = vpns
        
    #分配给VPN客户端的IP段
    ipv4-network = 10.12.0.0
    ipv4-netmask = 255.255.255.0
        
    #DNS
    dns = 114.114.114.114
    dns = 8.8.8.8
        
    #压缩，提高效率
    compression = true
    no-compression-limit = 256
        
    #注释掉route的字段，这样表示所有流量都通过 VPN 
    # route配置表示这个网段会经过vpn,这个网段一般是客户的内网网段
    route = 192.168.1.0/255.255.255.0
    #route = 192.168.5.0/255.255.255.0
    ```
    
2. 建立用户
    
    ```bash
    #username为你要添加的用户名
    $ sudo ocpasswd -c /etc/ocserv/ocpasswd username
    ```
    
3. 系统配置

    1. 开启内核转发
    
        ```
        $ echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
        $ sysctl -p /etc/sysctl.conf
        ```
        
    1. 配置防火墙规则
        
        ```bash
        #使用firewalled
        firewall-cmd --permanent --add-port=4433/tcp
        firewall-cmd --permanent --add-port=4433/udp
        # Allow firewall to forward
        firewall-cmd --permanent --add-masquerade
        firewall-cmd --reload
        
        # 使用iptables,vpnnetwork为配置中的内网网址，此配置为10.12.0.0/24,eth为公网ip
        iptables -I INPUT -p tcp --dport 4433 -j ACCEPT
        iptables -I INPUT -p udp --dport 4433 -j ACCEPT
        iptables -I FORWARD -s ${vpnnetwork} -j ACCEPT
        iptables -I FORWARD -d ${vpnnetwork} -j ACCEPT
        iptables -t nat -A POSTROUTING -s ${vpnnetwork} -o ${eth} -j MASQUERADE
        #iptables -t nat -A POSTROUTING -j MASQUERADE
        service iptables save
        ```
        
    
## 日志输出

1. 修改systemctl的标准输出与错误输出
    
    ```bash
    vim /usr/lib/systemd/system/ocserv.service
    ExecStart=/usr/sbin/ocserv --pid-file /var/run/ocserv.pid --config /etc/ocserv/ocserv.conf -f
    #修改为
    ExecStart=/bin/sh -c '/usr/sbin/ocserv --pid-file /var/run/ocserv.pid --config /etc/ocserv/ocserv.conf -f >> /var/log/agent.log 2>&1'
    ```
1. 输出用户连接和断开的日志信息

    ```bash
    vim /etc/ocserv/ocserv.conf
    # 修改下面的配置
    connect-script = /etc/ocserv/connect-script
    disconnect-script = /etc/ocserv/connect-script
    cert-user-oid = 2.5.4.3
    cert-group-oid = 2.5.4.11
    
    vim /etc/ocserv/connect-script
    # 输入下面内容，可以自己定制
    #!/bin/sh
    #echo $USERNAME : $REASON : $DEVICE
    case "$REASON" in
      connect)
    echo `date` $USERNAME "connected"
    echo `date` $REASON $USERNAME $DEVICE $IP_LOCAL $IP_REMOTE $IP_REAL
        ;;
      disconnect)
    echo `date` $USERNAME "disconnected"
        ;;
    esac
    exit 0
    ```
        
## radius cli配置
1. **Radcli installation，目前yum 安装ocserv时会自动安装radcli**
    
2. **radcli configuration (No TLS)**

    ```bash
    vim /etc/radcli/radiusclient.conf
    # 修改下面配置
    nas-identifier fw01  
    authserver 192.168.5.5  
    acctserver 192.168.5.5  
    servers /etc/radcli/servers  
    dictionary /etc/radcli/dictionary  
    default_realm  
    radius_timeout 10  
    radius_retries 3  
    bindaddr *
    ```

    配置servers的密码
    
    ```
    vim servers
    # 根据radius server的配置来设置servers的密码
    
    192.168.5.5 radiustestsecretpassword 
    #Save and exit
    ```

3. **Security considerations and radcli configuration with TLS**

    如果安全考虑，使用adiusclient-tls.conf 和 servers-tls这两个配置设置tls.


4. **修改ocserv的认证方式**

    ```bash
    vim /etc/ocserv/ocserv.conf
    #auth = "pam"  
    #auth = "pam[gid-min=1000]"  
    #auth = "plain[passwd=./ocserv.passwd]"  
    #auth = "certificate"  
    #auth = "radius[config=/etc/radcli/radiusclient.conf,groupconfig=true]"  
    #Add a line as follows:

    auth = "radius [config=/etc/radcli/radiusclient.conf,groupconfig=true]"
    #If you need RADIUS accounting, add a line in the "Accounting methods available" Section:

    acct = "radius [config=/etc/radcli/radiusclient.conf,groupconfig=true]"
    ```

## 常用命令
### 简单用户操作

1. 添加用户

    ```
    ocpasswd -c /etc/ocserv/ocpasswd 【用户名】
    ```

1. 添加用户至某个分组

    ```
    ocpasswd -c /etc/ocserv/ocpasswd -g 【分组名称】 【用户名】
    ```
1. 锁定用户

    ```
    ocpasswd -c /etc/ocserv/ocpasswd -l 【用户名】
    ```
1. 解锁用户

    ```
    ocpasswd -c /etc/ocserv/ocpasswd -u 【用户名】
    ```

1. 删除用户

    ```
    ocpasswd -c /etc/ocserv/ocpasswd -d 【用户名】
    ```
    
### 查看当前状态:
1. 查看当前服务运行状态

    ```
    occtl -n show status
    ```

1. 查看当前在线用户详情:

    ```bash
    occtl -n show users
    ```

1. 踢掉当前在线用户:
    1. 通过用户名:

        ```bash
        occtl disconnect user 【用户名】
        ```

    1. 通过id

        ```bash
        occtl disconnect id 【id号】
        ```

## 参考
1. [在 CentOS 7 上搭建 Cisco AnyConnect VPN](https://ifreedom.one/2015/04/20/Setup-Cisco-AnyConnect-VPN-on-CentOS7/)
2. [ocserv 0.12.1 manual](http://ocserv.gitlab.io/www/manual.html)
3. [CentOS 7.x ocserv + freeradius验证 + SSL](http://blog.kompaz.win/2016/09/29/CentOS%207.x%20ocserv%20+%20freeradius%E9%AA%8C%E8%AF%81%20+%20ssl/)
4. [Ocserv Authentication - RADIUS (radcli)](https://ocserv.gitlab.io/www/recipes-ocserv-authentication-radius-radcli.html)
