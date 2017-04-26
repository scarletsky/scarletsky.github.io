---
title: PostgreSQL 中的客户端认证
date: 2017-04-26 08:05:06
tags: database, postgresql
---

## 简介

当客户端与数据库服务器连接时，它需要指定用哪个数据库用户的身份来连接。PostgreSQL 为我们提供了很多种客户端认证的方式，我们可以根据自己的需要来选择认证方式。

## pg_hba.conf

相信每个在生产环境中使用 PostgreSQL 的人都折腾过客户端认证的问题，明明用户名和密码都正确，而且用户也是数据库的拥有者，为什么 PostgreSQL 会禁止访问？
通常这是因为 `pg_hba.conf` 文件没有配置好。

`pg_hba.conf` 是 PostgreSQL 客户端认证的配置文件 (hba 是 host-based authentication 的缩写)，它位于 PostgreSQl 的配置目录下，通常是 `/usr/local/pgsql/data` 或者 `/var/lib/pgsql/data`。
如果两个位置都找不到，可以尝试在终端中输入 `ps aux | grep postgres`，它会输出和 PostgreSQL 相关的进程，找到类似下面的进程：

```
# ps aux | grep postgres
root   3815  0.0  0.0  10772   920 pts/1    S+   00:34   0:00 grep --color=auto postgres
postgres 24819  0.0  1.9 238588 20068 ?        S    Apr25   0:01 /usr/bin/postgres -D /var/lib/pgsql/data
....
```

其中 -D 后面的就是该目录的位置了。
找到 `pg_hba.conf` 的位置之后，我们可以查看里面的内容：

```
# cat pg_hba.conf
...
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
```


## 参考资料
https://www.postgresql.org/docs/9.6/static/client-authentication.html
https://www.postgresql.org/docs/9.6/static/creating-cluster.html
http://stackoverflow.com/questions/14025972/postgresql-how-to-find-pg-hba-conf-file-using-mac-os-x
