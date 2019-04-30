---
title: MongoDB 运维基础
date: 2014-10-16 17:59
categories: [database]
tags: [mongodb]
---

可以试用 `mongod --help` 查看所有选项


# 启动 MongoDB:

```bash
mongod --dbpath xxx --port 1111 --config yyy.conf

#for ubuntu
service mongod start
```


# 使用配置文件获取配置信息(-f  或者 --config):

```bash
mongod --config ~/.mongodb.conf
```


# 停止 MongoDB:

```bash
kill -2 pid (SIGINT)
kill pid (SIGTEAM)
```

这两种方式都会稳妥退出，安全。
千万不能试用 kill -9 (SIGKILL) 这种方式去停止 MongoDB，这样会导致数据库直接关闭，又可能损坏数据库。
另一种方式是在 mongo shell 中试用 shutdownServer 命令，这个命令要在 admin 数据库下才能试用。
```bash
> use admin
> db.shutdownServer();
```


# 升级 MongoDB:

```bash
#for OS X
brew update
brew upgrade mongodb

#for ubuntu
http://docs.mongodb.org/manual/tutorial/install-mongodb-on-ubuntu/
需要指定安装版本
```


# 监控

- 使用 http 管理接口。即用浏览器访问 http://localhost:28017
   如果想要关闭该接口，需要在 mongod 启动时使用 --nohttpinterface 选项
- 使用 serverStatus。在 mongo shell 下使用 db.rubCommand({'serverStatus;: 1})
- mongostat 命令。
- 第三方插件


# 其他安全考虑

mongodb 传输协议是不加密的，如果需要加密，可以试用 ssh 或者类似的技术进行加密。
另外建议把 mongo 服务器布置在防火墙后或者布置在只有应用服务器能访问的网络中。同时建议使用 --bindip 选项，这样可以限制 ip 访问。运行 `mongod --bindip localhost` 后，只有本机的应用可以访问服务器。
最后，还可以用 --noscripting 选项完全禁止服务端 JavaScript 的执行。


# 备份和修复

mongodb 把所有数据都存放在数据目录下，默认目录是 /var/lib/mongo/ 下，如果只是想要备份，只需要简单创建文件副本就可以了。
mongodump 是一种可以在运行时备份的方法。这个工具使用普通的查询机制，备份时的查询会对其他客户端的性能产生不理的影响。
mongorestore 可以从备份中恢复数据的工具。
```bash
mongodump -d test -o backup/test
mongorestore -d foo --drop backup/test
```

`mongodump` 和 `mongorestore` 能不停机备份，但却失去了实时数据视图的能力。`fsync` 命令能在 mongodb 运行时赋值数据目录还不会损坏数据。
```bash
> use admin
> db.runCommand({'fsync': 1, 'lock': 1})  # 强制服务器将所有缓冲区写入磁盘，上锁阻止对数据库的写入。
> db.fsyncUnlock()  # 解锁
> db.currentOp()  # 查看
```

# 删除 MongoDB

```bash
#for ubuntu
sudo service mongod stop
sudo apt-get purge mongodb-org*
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongodb
```


# 参考资料

- [MongoDB权威指南](http://book.douban.com/subject/6068947/)
- [MongoDB官方文档](http://docs.mongodb.org/)
