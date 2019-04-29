---
title: Localtunnel（Node.js 版） 使用教程
date: 2016-01-17 17:07:06
categories: [localtunnel]
tags: [nodejs, localtunnel]
---

## 简介

Localtunnel 是一个可以让内网服务器暴露到公网上的开源项目。


## 客户端

### 安装
```shell
$ npm install -g localtunnel
```

### 使用
假设本地服务器在 8000 端口，我们可以通过下面的命令把本地服务器暴露到公网中

```shell
$ lt --port 8000
your url is: https://uhhzexcifv.localtunnel.me
```

通过上面的命令，我们不需要做其他设置就可以通过 `https://uhhzexcifv.localtunnel.me` 来访问我们本地服务器了。

由于 `localtunnel.me` 是国外的服务器，访问速度有时候不太理想，这时候我们可以自己搭建 localtunnel 的服务端。

## 服务端

### 安装
```shell
$ git clone git://github.com/defunctzombie/localtunnel-server.git
$ cd localtunnel-server
$ npm install
```

### 使用

以监听 2000 端口为例：

```shell
# 直接使用
$ bin/server --port 2000

# 配合 pm2 使用
$ pm2 start bin/server --name lt -- --port 2000
```

启动服务端程序后，我们只要在使用客户端 `lt` 时加上 `--host` 参数，就可以指定服务端了。

```shell
# host 后面不要加 /
$ lt --host http://helloworld.com:2000 --port 8000
your url is: http://jhuyudvlum.helloworld.com:2000
```

这样，我们就可以通过自己的代理服务器来访问本地服务器了，不用经过第三方代理服务器，不必担心代理服务器的安全问题。

## 高级用法

### 反向代理

在 Github 上面有一份 Nginx 的[配置](https://github.com/localtunnel/server/blob/master/devops/nginx/sites/localtunnel)，我们可以直接使用，或者按照自己的需要做些修改。

### 指定子域名

有时候，用随机字符串作为子域名并不是一件好事，我们可能需要固定的域名来访问本地服务器。这时，`lt --subdomain` 就可以派上用场了。

```shell
# subdomain 限制长度为 4 ~ 63
$ lt --host http://helloworld.com:2000 --port 8000 --subdomain mysubdomain
your url is: http://mysubdomain.helloworld.com:2000
```

看到了吗？通过 `--subdomain`，我们就可以指定自己喜欢的子域名了。


## 坑

**然而**，如果我们通过 `--host` 来指定子域名，会发生什么？

```shell
$ lt --host http://mysubdomain.hello.com --port 8000
Error: localtunnel server returned an error, please try again
```

就算配置了 Nginx 的反向代理，你依然会得到这个错误。可以查看 [#21](https://github.com/localtunnel/server/issues/21) 和  [#31](https://github.com/localtunnel/server/issues/31) 来看更多的细节。

要解决这个问题，最简单的就是 **不用** `--host` 来指定子域名，而用 `--subdomain` 来指定。

其实有好几个 pull request 都尝试去解决这个问题的，但不知道什么原因，作者一直没去合并。或者再过一段时间，这个问题就会解决，到时候， localtunnel 就会变得更加好用了。

## 参考资料
[https://github.com/localtunnel/localtunnel](https://github.com/localtunnel/localtunnel)
[https://github.com/localtunnel/server](https://github.com/localtunnel/server)
