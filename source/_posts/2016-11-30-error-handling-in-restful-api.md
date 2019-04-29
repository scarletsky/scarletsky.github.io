title: Restful API 中的错误处理
date: 2016-11-30 17:02:18
tags: [restful]
---

## 简介
随着移动开发和前端开发的崛起，越来越多的 Web 后端应用都倾向于实现 Restful API。
Restful API 是一个简单易用的前后端分离方案，它只需要对客户端请求进行处理，然后返回结果即可， 无需考虑页面渲染，一定程度上减轻了后端开发人员的负担。
然而，正是由于 Restful API 不需要考虑页面渲染，导致它不能在页面上展示错误信息。
那就意着当出现错误的时候，它只能通过返回一个错误的响应，来告诉用户和开发者相应的错误信息，提示他们接下来应该怎么办。
本文将讨论 Restful API 中的错误处理方案。


## 设计错误信息
当 Restful API 需要抛出错误的时候，我们要考虑的是：这个错误应该包含哪些信息。
我们先看看 Github, Google, Facebook, Twitter, Twilio 的错误信息是怎样的。

Github (use http status)

```json
{
  "message": "Validation Failed",
  "errors": [
    {
      "resource": "Issue",
      "field": "title",
      "code": "missing_field"
    }
  ]
}
```

Google (use http status)

```json
{
  "error": {
    "errors": [
      {
        "domain": "global",
        "reason": "insufficientFilePermissions",
        "message": "The user does not have sufficient permissions for file {fileId}."
      }
    ],
    "code": 403,
    "message": "The user does not have sufficient permissions for file {fileId}."
  }
}
```


Facebook (use http status)

```json
{
  "error": {
    "message": "Message describing the error", 
    "type": "OAuthException",
    "code": 190,
    "error_subcode": 460,
    "error_user_title": "A title",
    "error_user_msg": "A message",
    "fbtrace_id": "EJplcsCHuLu"
  }
}
```

Twitter (use http status)

```json
{
  "errors": [
    {
      "message": "Sorry, that page does not exist",
      "code": 34
    }
  ]
}
```

Twilio (use http status)

```json
{
  "code": 21211,
  "message": "The 'To' number 5551234567 is not a valid phone number.",
  "more_info": "https://www.twilio.com/docs/errors/21211",
  "status": 400
}
```

观察这些结构可以发现它们都有一些共同的地方：

- 都利用了 Http 状态码
- 有些返回了业务错误码
- 都提供了给用户看的错误提示信息
- 有些提供了给开发者看的错误信息

### Http 状态码
在 Restful API 中利用 Http 状态码来表明错误类型再合适不过了，因为 Http 状态码定义了很多抽象的错误类型。
虽然 Http 状态码定义了非常多的错误类型，但实际应用中，我们常用的状态码并不多，通常都是下面这几方面：

- API 正常工作 (200, 201)
- 客户端错误 (400, 401, 403, 404)
- 服务端错误 (500, 503)


### 业务错误码
很多时候，我们根据业务类型来自定义错误码。
这些业务错误码与 Http 状态码并不重叠，这时候我们可以返回业务错误码，用来提示用户/开发者错误类型。


### 给用户看的错误信息
当出现错误的时候，我们需要提示用户如何处理这种情况，通常这种错误信息都是必须的。
可以看到上面几个例子中都有返回给用户看的错误信息。


### 给开发者看的错误信息
若我们的 API 需要开放给第三方开发者，那么我们就需要考虑返回一些给开发者看的错误信息。


## 设计错误类型
我们刚才提到过，可以利用 Http 状态码来为错误类型进行分类。
通常我们所说的分类通常是对客户端错误进行分类， 即 4xx 类型的错误。

而这些错误类型中，我们最常用的是：

- 400 Bad Request
  由于包含语法错误，当前请求无法被服务器理解。除非进行修改，否则客户端不应该重复提交这个请求。
  通常在请求参数不合法或格式错误的时候可以返回这个状态码。

- 401 Unauthorized
  当前请求需要用户验证。
  通常在没有登录的状态下访问一些受保护的 API 时会用到这个状态码。

- 403 Forbidden
  服务器已经理解请求，但是拒绝执行它。与401响应不同的是，身份验证并不能提供任何帮助。
  通常在没有权限操作资源时(如修改/删除一个不属于该用户的资源时)会用到这个状态码。

- 404 Not Found
  请求失败，请求所希望得到的资源未被在服务器上发现。
  通常在找不到资源时返回这个状态码。

尽管我们可以通过 Http 状态码来表示错误的类型，
但在实际应用中，如果仅仅使用 Http 状态码的话，我们的代码中就遍布 Http 状态码：

```js
// Node.js
if (!res.body.title) {
  res.statusCode = 400
}

if (!user) {
  res.statusCode = 401
}

if (!post) {
  res.statusCode = 404
}
```

上面的实现方式在小项目中还可以接受，当项目变大、需求变多的时候，维护起来就变得很麻烦了。
为了提高错误的可读性和可维护性，我们需要对各种错误进行分类。
我个人习惯把错误分成以下几种类型：

- 格式错误 (FORMAT_INVALID)
- 数据不存在 (DATA_NOT_FOUND)
- 数据已存在 (DATA_EXISTED)
- 数据无效 (DATA_INVALID)
- 登录错误 (LOGIN_REQUIRED)
- 权限不足 (PERMISSION_DENIED)

错误分类之后，我们抛错误的时候就变得更加直观了：

```js
if (!res.body.title) {
  throw new Error(ERROR.FORMAT_INVALID)
}

if (!user) {
  throw new Error(ERROR.LOGIN_REQUIRED)
}

if (!post) {
  throw new Error(ERROR.DATA_NOT_FOUND)
}

if (post.creator.id !== user.id) {
  throw new Error(ERROR.PERMISSION_DENIED)
}
```

这种形式比上面的写死状态码的方式方便很多，而且维护起来也更加简单。
但有一个问题，就是不能根据错误类型来返回指定的错误信息。


## 自定义错误类型
要实现根据错误类型来返回指定的错误信息，我们可以通过自定义错误的方式来实现。
假设我们自定义错误的结构如下：

```json
{
  "type": "",
  "code": 0,
  "message": "",
  "detail": ""
}
```

我们需要做到如下几点：

- 根据错误类型来自动设置 `type`, `code`, `message`
- `detail` 为可选项，用来描述该错误的具体原因


```js
const ERROR = {
  FORMAT_INVALID: 'FORMAT_INVALID',
  DATA_NOT_FOUND: 'DATA_NOT_FOUND',
  DATA_EXISTED: 'DATA_EXISTED',
  DATA_INVALID: 'DATA_INVALID',
  LOGIN_REQUIRED: 'LOGIN_REQUIRED',
  PERMISSION_DENIED: 'PERMISSION_DENIED'
}

const ERROR_MAP = {
  FORMAT_INVALID: {
    code: 1,
    message: 'The request format is invalid'
  },
  DATA_NOT_FOUND: {
    code: 2,
    message: 'The data is not found in database'
  },
  DATA_EXISTED: {
    code: 3,
    message: 'The data has exist in database'
  },
  DATA_INVALID: {
    code: 4,
    message: 'The data is invalid'
  },
  LOGIN_REQUIRED: {
    code 5,
    message: 'Please login first'
  },
  PERMISSION_DENIED: {
    code: 6,
    message: 'You have no permission to operate'
  }
}

class CError extends Error {
  constructor(type, detail) {
    super()
    Error.captureStackTrace(this, this.constructor)

    let error = ERROR_MAP[type]
    if (!error) {
      error = {
        code: 999,
        message: 'Unknow error type'
      }
    }

    this.name = 'CError'
    this.type = error.code !== 999 ? type : 'UNDEFINED'
    this.code = error.code
    this.message = error.message
    this.detail = detail
  }
}
```

自定义好错误之后，我们调用起来就更加简单了：

```js
// in controller
if (!user) {
  throw new CError(ERROR.LOGIN_REQUIRED, 'You should login first')
}

if (!req.body.title) {
  throw new CError(ERROR.FORMAT_INVALID, 'Title is required')
}

if (!post) {
  throw new CError(ERROR.DATA_NOT_FOUND, 'The post you required is not found')
}
```

最后，还剩下一个问题，根据错误类型来设置状态码，然后返回错误信息给客户端。

## 捕获错误信息
在 Controller 中抛出自定义错误后，我们需要捕获该错误，才能返回给客户端。
假设我们使用 koa 2 作为 web 框架来开发 restful api，那么我们要做的是添加错误处理的中间件:

```js
module.exports = async function errorHandler (ctx, next) {
  try {
    await next()
  } catch (err) {

    let status

    switch (err.type) {
      case ERROR.FORMAT_INVALID:
      case ERROR.DATA_EXISTED:
      case ERROR.DATA_INVALID:
        status = 400
        break
      case ERROR.LOGIN_REQUIRED:
        status = 401
      case ERROR.PERMISSION_DENIED:
        status = 403
      case ERROR.DATA_NOT_FOUND:
        status = 404
        break
      default:
        status = 500
    }

    ctx.status = status
    ctx.body = err
  }
}

// in app.js
app.use(errorHandler)
app.use(router.routes())
```

通过这种方式，我们就能优雅地处理 Restful API 中的错误信息了。


## 参考资料
https://zh.wikipedia.org/zh-hans/HTTP%E7%8A%B6%E6%80%81%E7%A0%81
https://www.loggly.com/blog/node-js-error-handling/
http://blog.restcase.com/rest-api-error-codes-101/
https://apigee.com/about/blg/technology/restful-api-design-what-about-errors
http://stackoverflow.com/questions/942951/rest-api-error-return-good-practices
http://goldbergyoni.com/checklist-best-practices-of-node-js-error-handling/
http://blogs.mulesoft.com/dev/api-dev/api-best-practices-response-handling/
https://developers.facebook.com/docs/graph-api/using-graph-api/#errors
https://developers.google.com/drive/v3/web/handle-errors
https://developer.github.com/v3/#client-errors
https://dev.twitter.com/overview/api/response-codes
https://www.twilio.com/docs/api/errors
