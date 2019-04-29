---
title: PostgreSQL 中的客户端认证
date: 2017-04-26 08:05:06
categories: [database]
tags: [database, postgresql]
---

## 简介

当客户端与数据库服务器连接时，它需要指定用哪个数据库用户的身份来连接。
PostgreSQL 为我们提供了很多种客户端认证的方式，我们可以根据自己的需要来选择认证方式。

## psql

`psql` 是 PostgreSQL 的客户端程序，要连接 PostgreSQL 数据库，我们需要指定以下内容：

- `-d` or `--dbname` 数据库名
  - 默认情况下是连接与当前操作系统用户名字相同的数据库。
  - 如果该数据库不存在，会报 psql: FATAL:  database "root" does not exist。
- `-h` or `--host` 主机名
  - 默认情况下 psql 会通过 Unix socket 连接数据库。
  - 如果没有 Unix socket，那么会以 TCP/IP 连接到 localhost。
  - 如果需要通过 TCP/IP 连接到数据库，那么就需要指定主机名。
- `-p` or `--port` 端口号
  - 默认情况下是 5432 端口。
- `-U` or `--username` 用户名
  - 默认情况下是用当前操作系统用户名去连接数据库。
  - 如果该用户不存在，会报 psql: FATAL:  role "root" does not exist。

我们也可以用 URI 的方式连接数据库：

```
psql postgresql://dbmaster:5433/mydb?sslmode=require
```

## 常见情况

```
$ psql
postgres=# CREATE USER test WITH PASSWORD '123456';
postgres=# CREATE DATABASE test OWNER test;
postgres=# \q

$ psql -U test
psql: FATAL:  Peer authentication failed for user "test"
$ psql -U test -h 127.0.0.1
psql: FATAL:  Ident authentication failed for user "test"
```

相信每个在生产环境中使用 PostgreSQL 的人都折腾过客户端认证的问题。
明明用户名和密码都正确，而且用户也是数据库的拥有者，为什么 PostgreSQL 会禁止访问？
这通常是因为 `pg_hba.conf` 没有配置好。

## pg_hba.conf

### 位置

`pg_hba.conf` 是 PostgreSQL 客户端认证的配置文件 (hba 是 host-based authentication 的缩写)，它位于 PostgreSQL 的配置目录下，通常是 `/usr/local/pgsql/data` 或者 `/var/lib/pgsql/data`。
如果两个位置都找不到，但可以连接到数据库，可以直接在 psql shell 里面输入

```
postgres=# SHOW hba_file;
              hba_file
-------------------------------------
 /usr/local/var/postgres/pg_hba.conf
```

如果没有连接数据库的权限，可以尝试在终端中输入 `ps aux | grep postgres`，它会输出和 PostgreSQL 相关的进程，找到类似下面的进程：

```
# ps aux | grep postgres
root   3815  0.0  0.0  10772   920 pts/1    S+   00:34   0:00 grep --color=auto postgres
postgres 24819  0.0  1.9 238588 20068 ?        S    Apr25   0:01 /usr/bin/postgres -D /var/lib/pgsql/data
....
```

其中 -D 后面的就是该目录的位置了。

### 格式

找到 `pg_hba.conf` 的位置之后，我们可以查看里面的内容：

```
# cat pg_hba.conf
...
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     peer
host    all             all             127.0.0.1/32            ident
host    all             all             ::1/128                 ident
```

从内容可以看出，`pg_hba.conf` 是以行为单位来配置的，每一行包含了以下内容：

- `TYPE` 连接类型，表示允许用哪些方式连接数据库，它允许以下几个值：
  - `local` 通过 Unix socket 的方式连接。
  - `host` 通过 TCP/IP 的方式连接，它能匹配 SSL 和 non-SSL 连接。
  - `hostssl` 只允许 SSL 连接。
  - `hostnossl` 只允许 non-SSL 连接。

- `DATABASE` 可连接的数据库，它有以下几个特殊值：
  - `all` 匹配所有数据库。
  - `sameuser` 可连接和用户名相同的数据库。
  - `samerole` 可连接和角色名相同的数据库。
  - `replication` 允许复制连接，用于集群环境下的数据库同步。
  除了上面这些特殊值之外，我们可以写特定的数据库，可以用逗号 (,) 来分割多个数据库。

- `USER` 可连接数据库的用户，值有三种写法：
  - `all` 匹配所有用户。
  - 特定数据库用户名。
  - 特定数据库用户组，需要在前面加上 `+` (如：`+admin`)。

- `ADDRESS` 可连接数据库的地址，有以下几种形式：
  - `all` 匹配所有 IP 地址。
  - `samehost` 匹配该服务器的 IP 地址。
  - `samenet` 匹配该服务器子网下的 IP 地址。
  - ipaddress/netmask (如：172.20.143.89/32)，支持 IPv4 与 IPv6。
  - 如果上面几种形式都匹配不上，就会被当成是 hostname。
  **注意: 只有 host, hostssl, hostnossl 会应用个字段。**

- `METHOD` 连接数据库时的认证方式，常见的有几个特殊值：
  - `trust` 无条件通过认证。
  - `reject` 无条件拒绝认证。
  - `md5` 用 md5 加密密码进行认证。
  - `password` 用明文密码进行认证，不建议在不信任的网络中使用。
  - `ident` 从一个 ident 服务器 (RFC1413) 获得客户端的操作系统用户名并且用它作为被允许的数据库用户名来认证，只能用在 TCP/IP 的类型中 (即 host, hostssl, hostnossl)。
  - `peer` 从内核获得客户端的操作系统用户名并把它用作被允许的数据库用户名来认证，只能用于本地连接 (即 local)。
  - 其他特殊值可以在 [官方文档](https://www.postgresql.org/docs/9.6/static/auth-pg-hba-conf.html) 中查阅。
  **简单来说，ident 和 peer 都要求客户端操作系统中存在对应的用户。**
  **注意: 上面列举的只有 md5 和 password 是需要密码的，其他方式都不需要输入密码认证。**

了解完这些字段之后，我们可以看看 `pg_hba.conf` 初始化的内容了。

```
local   all             all                                     peer
# 表示本机上的所有用户可以以 Unix socket 的方式连接数据库。
host    all             all             127.0.0.1/32            ident
# 同上，但只允许 127.0.0.1/32 (IPv4) 这个 IP 连接数据库。
host    all             all             ::1/128                 ident
# 同上，但只允许 ::1/128 (IPv6) 这个 IP 连接数据库。
```


## 修复

在文章开头，我们看见过两种错误，分别是 Peer authentication failed 和 Ident authentication failed。
这是因为 `pg_hba.conf` 中限定了认证方式是 `peer` 和 `ident`。
要让我们的 test 用户连接到数据库，我们有以下选择：

- 让 test 用户通过 `peer` 和 `ident` 认证
- 修改 `pg_hba.conf` 的认证方式

### 让用户通过 peer 和 ident 认证

从上面可以知道，要让用户通过 peer 和 ident 认证，我们需要在操作系统中创建对应的用户。

```
$ sudo su -
# useradd test
# psql -U test
psql: FATAL:  Peer authentication failed for user "test"
# psql -U test -h 127.0.0.1
psql: FATAL:  Ident authentication failed for user "test"
# su - test
$ psql
test=>
```

如果使用了 ident 的方式进行认证，而客户端的操作系统中没有 ident 服务器，那么客户端的操作系统中需要先安装 ident 服务器。

```
# su - test
$ psql -U test -h 127.0.0.1
psql: FATAL:  Ident authentication failed for user "test"

$ dnf install oidentd
$ systemctl enable oidentd
$ systemctl start oidentd

$ psql -U test -h 127.0.0.1
test=>
```


### 修改 pg_hba.conf

打开 `pg_hba.conf`，把规则中的 ident 修改成 md5，即：

```
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

然后重启 PostgreSQL 服务器即可生效。

```
$ vim /var/lib/pgsql/data/pg_hba.conf
$ systemctl restart postgresql
$ psql -U test -h 127.0.0.1
Password for user test:
test=>
```

## 参考资料
https://www.postgresql.org/docs/9.6/static/app-psql.html
https://www.postgresql.org/docs/9.6/static/client-authentication.html
https://www.postgresql.org/docs/9.6/static/auth-methods.html
https://www.postgresql.org/docs/9.6/static/creating-cluster.html
http://www.davidpashley.com/articles/postgresql-user-administration/
http://stackoverflow.com/questions/14025972/postgresql-how-to-find-pg-hba-conf-file-using-mac-os-x
http://stackoverflow.com/questions/2942485/psql-fatal-ident-authentication-failed-for-user-postgres
