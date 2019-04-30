---
title: 在 Node.js 中为 Restful API 编写单元测试
date: 2016-10-05 11:57:26
categories: [javascript]
tags: [javascript, nodejs]
---

## 简介

单元测试是针对程序模块来进行正确性检验的测试工作，程序单元是应用的最小可测试部件。
在 Web 应用中，我们可以把 Restful API 看作是构成应用的单元。 
Restful API 比较好测试，测试起来也比较简单。
本文将介绍编写测试的原因和原则，然后以 Node.js 为例子介绍测试 Restful API 的方法。

## 为什么要编写测试

每个开发者都知道单元测试的重要性，但并不是每个开发者都会去编写单元测试。原因也很容易理解：

- 编写测试需要更多的时间，会拖慢项目进度
- 编写测试需要写更多的代码，更容易出现错误
- 当需求更变后，我们要花更多的精力去修改测试代码
- 有时候测试代码可能会比源代码多几倍

我以前也抱有类似的想法，认为人工测试就足够了，没必要特意去编写单元测试。
然而随着项目的规模的增大，代码也会慢慢出现意外。
最典型的例子是为了某需求修改了 A 位置，需求完成了，而 B 位置就出现了 Bug。

在读过 [王垠大神的《测试的道理》](http://www.yinwang.org/blog-cn/2016/09/14/tests) 之后，我更加明白了一个道理：我没有大神般的编码能力，我只能通过单元测试来检验自己的编码。

除了检验代码的正确性之外，我认为单元测试还有一个很重要的作用：为日后重构项目做准备。
只要单元测试覆盖得够好，以后重构的时候就很容易发现问题，节约大量的时间。

轮子哥在 [知乎](https://www.zhihu.com/question/28729261/answer/94964928) 上说过一句很有意思的话：
> 所以那些专门写不需要维护的软件的人，讨厌测试，也是情有可原的。

如果你要编写一个长期维护的软件，那么你最好添加单元测试。


## F.I.R.S.T 原则

当我们决定要编写单元测试之后，我们就要考虑怎样 **写好** 单元测试，换句话说就是编写单元测试时需要注意哪些原则。
那么，有哪些原则是我们需要注意的呢？

- **Fast**: 测试必须是快速的
- **Isolated / Independent**: 
  - 每个测试都要做 3 A => Arrange(准备), Act(行动), Assert(断言)
  - Arrange: 测试过程中用到的数据不能依赖于运行环境，测试中用到的数据应是测试中的一部分
  - Act: 调用你想要测试的方法 / API
  - Assert: 根据返回结果进行断言
  - 测试结果不能依赖运行环境
  - 测试结果不依赖运行测试的顺序
- **Repeatable**:
  - 每个测试必须是可重复执行的，即运行 N 次，会得到 N 次相同的结果
  - 每个测试的结果不应依赖时间，日期，和随机数的输出
- **Self-validating**: 
  - 每个测试都可以自己判断结果来判断测试是否通过
  - 不需要人类去查阅手册来判断结果
- **Thorough and Timely**:
  - 应该尽量覆盖所有使用场景
  - 应该尝试测试驱动开发(TDD)

这就是经典的 F.I.R.S.T 原则。
我们最好时刻注意自己编写的单元测试是否遵守这些原则。

JavaScript 社区里有很多测试框架可以用来编写单元测试，有 `ava`、`mocha`、`jasmine`、`tap` 等。
这些测试框架都有提供 `beforeEach`、`afterEach` API，目的是隔离我们的测试数据，从而满足 **Isolated / Independent** 和 **Repeatable** 原则。


## 编写单元测试

假设我们有以下 Restful API (用了 jwt 来做用户验证):

```js
const router = new Router()
// 根据 token 获取用户信息，必须登录
router.get('/user', jwt, user.getSelf)
// 获取用户列表，无需登录
router.get('/users', user.getList)
// 获取指定用户信息，无需登录
router.get('/users/:userId', user.get)
// 创建新用户(用户注册)，无需登录
router.post('/users', user.create)
// 更新用户信息，必须登录
router.put('/user', jwt, user.update)
```

那么我们应该怎样为这些 Restful API 编写单元测试呢？

最基本流程是：
- 为 app 创建 http 服务器
- 对各个 API 发出请求
- 对响应内容进行断言

幸运的是，社区里已经有相应的工具让我们可以方便管理这个流程，这个工具就是 —— `supertest`。
它提供了非常灵活的 API，足以帮助我们测试 Restful API 了。 
基本用法如下：

```js
const app = require('../app')
const request = require('supertest')(app)

request
  .get('/users')
  .expect(200)
  .end((err, res) => {
    res.body.should.be.an.Array()
  })
```

> 提示
如果你遇到了 `TypeError: app.address is not a function`, 请尝试一下以下方法：
```js
const request = require('supertest').agent(app.listen())
```

现在，我们可以把 `supertest` 和其他测试框架整合起来了，我选择了 `mocha` 作为例子，因为它很经典，当你会用 `mocha` 之后，其他测试框架基本上就难不倒你了。

```js
const co = require('co')
const { ObjectId } = require('mongoose').Types
const config = require('../config')
const UserModel = require('../models/user')
const app = require('../app')
const request = require('supertest')(app)

describe('User API', function () {

  // 为每个单元测试初始化数据
  // 每个单元测试中可以通过 context 来访问相关的数据
  beforeEach(function (done) {
    co(function* () {
      self.user1 = yield UserModel.create({ username: 'user1' })
      self.token = jwt.sign({ _id: self.user1._id }, config.jwtSecret, { expiresIn: 3600 })
      done()
    }).catch(err => {
      console.log('err: ', err)
      done()
    })
  })

  // 正常情况下访问 /user
  it('should get user info when GET /user with token', function (done) {
    const self = this
    request
      .get('/user')
      .set('Authorization', self.token)
      .expect(200)
      .end((err, res) => {
        res.body._id.should.equal(self.user1._id)
        done()
      })
  })

  // 非正常情况下访问 /user
  it('should return 403 when GET /user without token', function (done) {
    request
      .get('/user')
      .expect(403, done)
  })

  // 访问 /users，登录用户和非登录用户都会得到相同的结果，所以不需要区别对待
  it('should return user list when GET /users', function (done) {
    request
      .get('/users')
      .expect(200)
      .end((err, res) => {
        res.body.should.be.an.Array()
        done()
      })
  })

  // 访问 /users/:userId 也不需要区分登录和非登录状态
  it('should return user info when GET /users/:userId', function (done) {
    const self = this
    request
      .get(`/users/${self.user1._id}`)
      .expect(200)
      .end((err, res) => {
        res.body._id.should.equal(self.user1._id)
        done()
      })
  })

  // 访问不存在的用户，我们需要构造一个虚假的用户 id
  it('should return 404 when GET /users/${non-existent}', function (done) {
    request
      .get(`/users/${ObjectId()}`)
      .expect(404, done)
  })

  // 正常情况下的用户注册不会带上 token
  it('should return user info when POST /user', function (done) {
    const username = 'test user'
    request
      .post('/users')
      .send({ username: username })
      .expect(200)
      .end((err, res) => {
        res.body.username.should.equal(username)
        done()
      })
  })

  // 非法情况下的用户注册，带上了 token 的请求要判断为非法请求
  it('should return 400 when POST /user with token', function (done) {
    const username = 'test user 2'
    request
      .post('/users')
      .set('Authorization', this.token)
      .send({ username: username })
      .expect(400, done)
  })

  // 正常情况下更新用户信息，需要带上 token
  it('should return 200 when PUT /user with token', function (done) {
    request
      .put('/user')
      .set('Authorization', this.token)
      .send({ username: 'valid username' })
      .expect(200, done)
  })

  // 非法情况下更新用户信息，如缺少 token
  it('should return 400 when PUT /user without token', function (done) {
    request
      .put('/user')
      .send({ username: 'valid username' })
      .expect(400, done)
  })

})
```

可以看到，为 Restful API 编写单元测试还有一个优点，就是可以轻易区分登录状态和非登录状态。如果要在用户界面中测试这些功能，那么就需要不停地登录和注销，将会是一项累人的工作~
另外，上面的例子中基本都是对返回状态吗进行断言的，你可以按照自己的需要进行断言。

> 提示
你可以选择自己喜欢的断言库，我这里选择了 should.js，原因是好读。
个人认为 should.js 和其他断言库比起来有个缺点，就是不好写。
value.should.xxx.yyy.zzz 这个形式和 assert.equal(value, expected) 相比不太直观。
另外由于 should.js 是通过扩展 Object.prototype 的原型来实现的，但 null 值是一个例外，它不能访问任何属性。
因此 should.js 在 null 上会失效。
一个变通的办法是 `(value === null).should.equal(true)`。

---

```shell
$ npm test

  User api
    ✓ should get user info when GET /user with token
    ✓ should return 403 when GET /user without token
    ✓ should return user list when GET /users
    ✓ should return user info when GET /users/:userId
    ✓ should return 404 when GET /users/${non-existent}
    ✓ should return user info when POST /user
    ✓ should return 400 when POST /user with token
    ✓ should return 200 when PUT /user with token
    ✓ should return 400 when PUT /user without token
```

当我们运行测试时，看到自己编写的测试都通过时，心里都会非常踏实。
而当我们要对项目进行重构时，这些测试用例会帮我们发现重构过程中的问题，减少 Debug 时间，提升重构时的效率。

## 细节

### 如何连接测试数据库

在 Node.js 的环境下，我们可以设置环境变量 `NODE_ENV=test`，然后通过这个环境变量去连接测试数据库，这样测试数据就不会存在于开发环境下的数据库拉！

```js
// config.js
module.exports = {
  development: {},
  production: {},
  test: {}
}

// app.js
const ENV = process.NODE_ENV || 'development'
const config = require('./config')[ENV]
// connect db by config
```


### 如何清空测试数据库

清空数据库这种一次性的工作最好放到 npm scripts 中处理，需要进行清空操作的时候直接运行 `npm run resetDB` 就可以了。
需要注意的是，编写清空数据库脚本时必须判断环境变量 `NODE_ENV`，以免误删 production 环境下的数据。

```js
// resetDB.js
const env = process.NODE_ENV || 'development'
if (env === 'test' || env === 'development') {
  // connect db and delete data
} else {
  throw new Error('You can not run this script in production.')
}

// package.json
{
  "scripts": {
    "resetDB": "node scripts/resetDB.js"
  },
  // ...
}
```


### 何时清空测试环境的数据库

如果是按照上面的原则来生成测试数据的话，测试数据其实可以不用删掉的。
但由于测试数据会占用我们的空间，最好还是把这些测试数据删掉。
那么，清空测试数据库这个操作在测试前执行好，还是测试后执行好？
我个人倾向于测试前删除，因为有时候我们需要进入数据库，查看测试数据的正确性。
如果在测试后清空测试数据库的话，我们就没办法访问到测试数据了。

```js
{
  "scripts": {
    "resetDB": "node scripts/resetDB.js",
    "test": "NODE_ENV=test npm run resetDB && mocha --harmony"
  },
  // ...
}
```


## 参考资料

- https://zh.wikipedia.org/zh-cn/%E5%8D%95%E5%85%83%E6%B5%8B%E8%AF%95
- http://blog.hubstaff.com/why-you-should-write-unit-tests/
- http://www.yinwang.org/blog-cn/2016/09/14/tests
- https://www.zhihu.com/question/28729261/answer/94964928
- http://agileinaflash.blogspot.de/2009/02/first.html
- http://howtodoinjava.com/best-practices/first-principles-for-good-tests/
- https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing
- https://github.com/sindresorhus/awesome-nodejs#testing
- https://github.com/visionmedia/supertest
- https://yq.aliyun.com/articles/57804
