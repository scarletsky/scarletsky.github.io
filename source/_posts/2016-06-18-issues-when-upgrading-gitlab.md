---
title: 升级 GitLab 过程中踩过的坑
draft: true
date: 2016-06-18 19:58:57
tags: gitlab
---


## 简介

最近帮公司的 Gitlab 从 7.14 升级到 8.7.6，升级的主要动机是整合 Gitlab CI，提升持续集成的效率。鉴于之前也是我把 Gitlab 从 7.x 升级到 7.14 的，我以为我已经有经验去处理了，但实际上，这次升级让我踩了不少以前没遇到过的坑...
本文主要记录升级 Gitlab 过程中踩过的一些坑~


## MySQL 突然启动失败

由于我们公司的 Gitlab 服务器内存不太够，只有 2G，所以偶尔会出现 500 的错误，但通常情况下，直接通过 `$ sudo service gitlab restart` 重启一次服务就可以恢复正常了。
但是，最近我遇到过 gitlab 启动失败，原因是 MySQL 无法启动。
当我启动 MySQL 的时候，报了这样的错误：

```
$ sudo service mysql restart
...
MySQL Job failed to start
```

报错就报错吧，去看看 error.log 就好了吧。 然而，我太天真了:

```
$ cat /var/log/mysql/error.log
$ cat /var/log/mysql/mysql.log
```

MySQL 的 error.log 是空的，mysql.log 也是空的...太诡异了，没 log 怎么 debug ?
还好有万能的 google，我找到了类似情况的问题：[mysql-job-failed-to-start](http://stackoverflow.com/questions/22909060/mysql-job-failed-to-start)。最高票的答案是重装 mysql 的，但我不敢冒这个风险。我注意到有个回答说：

> The given solution requires enough free HDD

于是我马上查了一下硬盘空间：

```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1       20G   20G  0G   100% /
...
```

原来是空间满了，于是我删掉了一些没用的东西，腾出了 3G 多的空间之后，MySQL 又可以正常启动了，Gitlab 也是！


## 迁移 MySQL 到 PostgreSQL 时找不到依赖

以前一直听说 PostgreSQL 很强大，而 Gitlab 也是官方推荐使用 PostgreSQL，再加上 Gitlab 有官方教程教我们如何把 MySQL 迁移到 PostgreSQL，于是我就萌生了迁移数据库的念头。

按照着 [Migrating GitLab from MySQL to Postgres](https://gitlab.com/gitlab-org/gitlab-ce/blob/master/doc/update/mysql_to_postgresql.md) 这篇文章，我很轻易就把 MySQL 中的数据迁移到 PostgreSQL 中了。
而且每个步骤都没报错，我以为一切正常，谁知道当我执行 `service gitlab start` 的时候，就报启动失败了。

我一下子就懵了，为什么按照官方教程做还会报错？ 而且报的还是这种抱不到依赖的错误？
```
Specified 'postgresql' for database adapter, but the gem is not loaded. Add `gem 'pg'` to your Gemfile.
```

我找了很多资料，但基本都是说把 `pg` 加到 Gemfile ，然后再 `bundle install` 就好了。
但我试过把 `vendor/bundle` 删掉再重装，还是有这个错误。
最后，我找到 [Gem::LoadError: Specified 'mysql2' after update to 7.13
](https://gitlab.com/gitlab-org/gitlab-ci/issues/227) 这个 issue，里面提到在 `.bundle/config` 里面有一些 bundle 的配置！我发现我里面的配置和这个 issue 提到的配置是一样的：

```
BUNDLE_WITHOUT: development:test:postgres:mysql
```

最后，我把上面的 `postgres` 部分删掉，然后再运行 `sudo -u git -H bundle install --without development test mysql --deployment
` 安装依赖，就能正常启动 gitlab 了。

这时候，我才体会到，不熟悉 ruby 技术栈去搞 gitlab 真是累啊~


## Docker 化时自动修改用户组引起其他问题

**严格来说这不是升级 Gitlab 升级过程中遇到的坑，这只是我在尝试把 gitlab docker 化时遇到的问题。**

有了上面的经历，我暗下决心，要把 Gitlab Docker 化，这样我就不用操心 Gitlab 相关的问题了，我只要启动下载镜像，运行容器就好了，其他 Gitlab 相关的事情就交回给官方处理好了。

于是，我找到了 [docker-gitlab](https://github.com/sameersbn/docker-gitlab)，我慢慢去尝试把 Gitlab 扔到 docker 里面运行。

最开始的时候，我直接用 docker-compose 来运行 docker-gitlab，我试着先用 MySQL ，于是我把 `docker-compose.yaml` 中的 PostgreSQL 部分的东西全部改成 MySQL，然后直接用 `docker-compose up` 来运行 gitlab 。

虽然运行过程中没看出什么问题，但我后来发现 MySQL 中的用户和用户组都被修改成 `messagebus:fuse` 了！这样直接导致了我本地的 MySQL 不能正常使用了。

经过调查，我发现是 docker 的 `-v` 指定已存在目录的时候，会修改文件目录的用户和用户组，最后把它改回 `mysql:adm` 才恢复正常~



## 参考资料
http://stackoverflow.com/questions/22909060/mysql-job-failed-to-start
https://gitlab.com/gitlab-org/gitlab-ci/issues/227
https://github.com/sameersbn/docker-gitlab
