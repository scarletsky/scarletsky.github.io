---
title: 用 GitLab CI 进行持续集成
date: 2016-07-29 09:22:20
tags: [ci, gitlab-ci]
---

# 简介
从 GitLab 8.0 开始，GitLab CI 就已经集成在 GitLab 中，我们只要在项目中添加一个 `.gitlab-ci.yml` 文件，然后添加一个 Runner，即可进行持续集成。 而且随着 GitLab 的升级，GitLab CI 变得越来越强大，本文将介绍如何使用 GitLab CI 进行持续集成。


# 一些概念
在介绍 GitLab CI 之前，我们先看看一些持续集成相关的概念。

## Pipeline
一次 Pipeline 其实相当于一次构建任务，里面可以包含多个流程，如安装依赖、运行测试、编译、部署测试服务器、部署生产服务器等流程。
任何提交或者 Merge Request 的合并都可以触发 Pipeline，如下图所示：

```
+------------------+           +----------------+
|                  |  trigger  |                |
|   Commit / MR    +---------->+    Pipeline    |
|                  |           |                |
+------------------+           +----------------+
```


## Stages
Stages 表示构建阶段，说白了就是上面提到的流程。
我们可以在一次 Pipeline 中定义多个 Stages，这些 Stages 会有以下特点：

- 所有 Stages 会按照顺序运行，即当一个 Stage 完成后，下一个 Stage 才会开始
- 只有当所有 Stages 完成后，该构建任务 (Pipeline) 才会成功
- 如果任何一个 Stage 失败，那么后面的 Stages 不会执行，该构建任务 (Pipeline) 失败

因此，Stages 和 Pipeline 的关系就是：

```
+--------------------------------------------------------+
|                                                        |
|  Pipeline                                              |
|                                                        |
|  +-----------+     +------------+      +------------+  |
|  |  Stage 1  |---->|   Stage 2  |----->|   Stage 3  |  |
|  +-----------+     +------------+      +------------+  |
|                                                        |
+--------------------------------------------------------+
```


## Jobs
Jobs 表示构建工作，表示某个 Stage 里面执行的工作。
我们可以在 Stages 里面定义多个 Jobs，这些 Jobs 会有以下特点：

- 相同 Stage 中的 Jobs 会并行执行
- 相同 Stage 中的 Jobs 都执行成功时，该 Stage 才会成功
- 如果任何一个 Job 失败，那么该 Stage 失败，即该构建任务 (Pipeline) 失败

所以，Jobs 和 Stage 的关系图就是：

```
+------------------------------------------+
|                                          |
|  Stage 1                                 |
|                                          |
|  +---------+  +---------+  +---------+   |
|  |  Job 1  |  |  Job 2  |  |  Job 3  |   |
|  +---------+  +---------+  +---------+   |
|                                          |
+------------------------------------------+
```


# GitLab Runner

## 简介
理解了上面的基本概念之后，有没有觉得少了些什么东西 —— 由谁来执行这些构建任务呢？
答案就是 GitLab Runner 了！

想问为什么不是 GitLab CI 来运行那些构建任务？
一般来说，构建任务都会占用很多的系统资源 (譬如编译代码)，而 GitLab CI 又是 GitLab 的一部分，如果由 GitLab CI 来运行构建任务的话，在执行构建任务的时候，GitLab 的性能会大幅下降。

GitLab CI 最大的作用是管理各个项目的构建状态，因此，运行构建任务这种浪费资源的事情就交给 GitLab Runner 来做拉！
因为 GitLab Runner 可以安装到不同的机器上，所以在构建任务运行期间并不会影响到 GitLab 的性能~

## 安装

安装 GitLab Runner 太简单了，按照着 [官方文档](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner) 的教程来就好拉！
下面是 Debian/Ubuntu/CentOS 的安装方法，其他系统去参考官方文档：

``` bash
# For Debian/Ubuntu
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.deb.sh | sudo bash
$ sudo apt-get install gitlab-ci-multi-runner

# For CentOS
$ curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
$ sudo yum install gitlab-ci-multi-runner
```

## 注册 Runner
安装好 GitLab Runner 之后，我们只要启动 Runner 然后和 CI 绑定就可以了：

- 打开你 GitLab 中的项目页面，在项目设置中找到 runners
- 运行 `sudo gitlab-ci-multi-runner register`
- 输入 CI URL
- 输入 Token
- 输入 Runner 的名字
- 选择 Runner 的类型，简单起见还是选 Shell 吧
- 完成

当注册好 Runner 之后，可以用 `sudo gitlab-ci-multi-runner list` 命令来查看各个 Runner 的状态：

```bash
$ sudo gitlab-runner list
Listing configured runners          ConfigFile=/etc/gitlab-runner/config.toml
my-runner                           Executor=shell Token=cd1cd7cf243afb47094677855aacd3 URL=http://mygitlab.com/ci
```


# .gitlab-ci.yml


# 踩过的坑


# 参考资料
https://about.gitlab.com/gitlab-ci/
http://docs.gitlab.com/ce/ci/yaml/README.html
http://docs.gitlab.com/ce/ci/variables/README.html
https://gitlab.com/gitlab-org/gitlab-ci-multi-runner
https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/issues/1232
http://stackbox.cn/2016-02-gitlab-ci-conf/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io

