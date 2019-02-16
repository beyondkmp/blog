---
title: "os x 10.10安装metasploit"
date: 2015-03-31T00:00:00+0800
lastmod: 2015-03-31T00:00:00+0800
draft: false
keywords: ["os x 10.10","metasploit"]
tags:  ["os x 10.10","metasploit"]
description: "在mac os x10.10.2上安装渗透神器metasploit"
categories: ["常用技巧"]
tags: ["os x 10.10","metasploit"]
---

##步骤一：安装metasploit
参考[Installing Metasploit Framework on Mountain Lion and Mavericks](http://www.darkoperator.com/installing-metasploit-framewor/)

##步骤二：安装postgreSQL

安装postgresql:

```bash
brew install postgresql --without-ossp-uuid
```
配置postgresql:

第一次安装要初始化数据库文件：

```bash
initdb /usr/local/var/postgres
```

然后会出现下面的提示：

```
WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

postgres -D /usr/local/var/postgres/data
or
pg_ctl -D /usr/local/var/postgres/data -l logfile start
```

运行上面的指令，不要运行pg_ctl(有问题)

```bash
postgres -D /usr/local/var/postgres/data
```

这就启动了postgresql，下次重启电脑后只要再运行上面的命令就可以了。可以用下面的命令查看下是否在启动成功

```bash
lsof -i :5432
```

上面只是启动,还要在postgresql建立相应的用户与数据库，在metasploit的安装目录下的config文件夹中找到database.yml,内容如下：

```
development: &pgsql
  adapter: postgresql
  database: msf 
  username: msf 
  password: msf
  host: localhost
  port: 5432
  pool: 5
  timeout: 5
```

根据上面的内容运行下面命令建立相应的数据库与用户：

```bash
createuser msf -P -h localhost
createdb -O msf msf -h localhost (大写的o)
```

##最后

进行msfconsole界面，运行`db_status`看下是否已经连接上。如果已经连接再运行`db_rebuild_cache`,建立数据库缓存。


