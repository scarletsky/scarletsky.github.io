---
layout: post
title: 'MongoDB 安全与认证'
date: 2014-10-25 17:18
categories: mongodb
---

# 安全和认证

要开启安全检查，需要在启动 mongod 时添加 --auth 选项
admin 数据库为管理员，在认证之后可以读写所有数据库，执行特定的管理命令。
在开启安全检查之前，一定要有一个管理员帐号。
数据库的用户存储在 admin 的 `db.system.users` 集合中。

创建管理员
```bash
> use admin
> db.createUser({user: 'root', pwd: 'password', roles: ['root']}) # 添加管理员
```

开启验证
```bash
$ mongod --config /usr/local/etc/mongod.conf --auth

# 验证方法1
$ mongo admin -u 'username' -p 'password' # 必须指定该用户有权限访问的数据库

# 验证方法2
$ mongo
> use admin
> db.auth('username', 'password')
> use otherDatabase
> db.users.find().....
```

开启验证后，我们在使用 URI 去连接数据库的时候需要注意，要加上 authSource=admin 参数才能通过验证。这里以 mongoose 为例。
```js
var mongoose = require('mongoose');
mongoose.connect('mongodb://user:pass@server:port/database?authSource=admin');
```

# 用户角色权限
Mongodb 内建了很多角色，详细可以去看看 [built-in-roles](http://docs.mongodb.org/manual/reference/built-in-roles/#built-in-roles)

这里只说说 Superuser roles。下面几种角色可以为任何用户分配任何权限在任何数据库上，这意味着拥有以下其中一种角色的用户可以分配他们自己的任何权限在任何数据库上面。
- dbOwner 角色，在 admin 数据库作用域中
- userAdmin 角色，在 admin 数据库作用域中
- userAdminAnyDatabase 角色
- root 角色拥有全部的权限

需要注意的是，root 角色并没有在 `system.users` 和 `system.roles` 集合中直接插入数据的能力。因此， root 并不适合用 mongorestore 恢复那些包含了这两个集合的数据。如果你需要进行这些操作，请给用户加上 `restore` 角色。


# 其他安全考虑
mongodb 传输协议是不加密的，如果需要加密，可以试用 `ssh` 或者类似的技术进行加密。
另外建议把 mongo 服务器布置在防火墙后或者布置在只有应用服务器能访问的网络中。同时建议使用 `--bindip` 选项，这样可以限制 ip 访问。运行 `mongod --bindip localhost` 后，只有本机的应用可以访问服务器。
最后，还可以用 `--noscripting` 选项完全禁止服务端 `JavaScript` 的执行。

# 参考文献
[Mongodb 官方文档](http://docs.mongodb.org/manual/reference/method/js-user-management/)


