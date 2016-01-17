---
title: ExpressJS 源码学习 —— Router 篇
date: 2015-08-27 00:36:54
categories: javascript
---

# 概述

Router 分为三个部分，分别是：Layer, Route, Router。


# Layer

## 构造函数

```js
function Layer(path, options, fn) {
  if (!(this instanceof Layer)) {
    return new Layer(path, options, fn);
  }

  debug('new %s', path);
  // Route 中会传一个空的对象进来
  var opts = options || {};

  this.handle = fn;
  this.name = fn.name || '<anonymous>';
  this.params = undefined;
  // 不会受到传进来的 path 的影响
  // 这个值会在 layer.match(path) 中修改
  this.path = undefined;
  
  // 定义 this.keys
  // pathRegexp(path, keys, options) 会把路径转换成正则表达式，express 通过这个库来解析路由
  // 转换后，this.regexp 为正则表达式，如果 path 中带有 express 路由风格的参数(/:foo/:bar)，则 this.keys 会变成 [{name: 'foo'...}, {name: 'bar'...}]，如果没有匹配的话，则 this.keys 就会是空数组
  // 如果想知道更多这个库的信息，请到 https://github.com/pillarjs/path-to-regexp
  this.regexp = pathRegexp(path, this.keys = [], opts);

  // router.route 中会把 opts.end 设置成 true
  if (path === '/' && opts.end === false) {
    this.regexp.fast_slash = true;
  }
}
```

## Layer.prototype.handle_error 处理错误


```js
/**
 * Handle the error for the layer.
 *
 * @param {Error} error
 * @param {Request} req
 * @param {Response} res
 * @param {function} next
 * @api private
 */
 
// route.dispatch 中调用这个方法
Layer.prototype.handle_error = function handle_error(error, req, res, next) {
  var fn = this.handle;

  // fn 必须要有 4 个参数
  if (fn.length !== 4) {
    // not a standard error handler
    return next(error);
  }

  try {
    fn(error, req, res, next);
  } catch (err) {
    next(err);
  }
};
```

## Layer.prototype.handle_request 处理请求


```js
/**
 * Handle the request for the layer.
 *
 * @param {Request} req
 * @param {Response} res
 * @param {function} next
 * @api private
 */

// route.dispatch 中会调用这个方法
Layer.prototype.handle_request = function handle(req, res, next) {
  var fn = this.handle;

  // fn 如果参数多余三个，那么它就不是一个标准的处理器
  if (fn.length > 3) {
    // not a standard request handler
    return next();
  }

  try {
    fn(req, res, next);
  } catch (err) {
    next(err);
  }
};
```


## Layer.prototype.match 检查是否匹配指定路径，同时会把获取 URL 中变量的实际值

```js

/**
 * Check if this route matches `path`, if so
 * populate `.params`.
 *
 * @param {String} path
 * @return {Boolean}
 * @api private
 */

Layer.prototype.match = function match(path) {
  // 检查路径
  if (path == null) {
    // no path, nothing matches
    this.params = undefined;
    this.path = undefined;
    return false;
  }

  if (this.regexp.fast_slash) {
    // fast path non-ending match for / (everything matches)
    this.params = {};
    this.path = '';
    return true;
  }

  // 匹配 path
  var m = this.regexp.exec(path);

  if (!m) {
    this.params = undefined;
    this.path = undefined;
    return false;
  }

  // store values
  this.params = {};
  this.path = m[0];

  // keys 是在生成 this.regexp 中生成的，简单来说就是当路径有参数的时候就有值，否则就是空数组
  var keys = this.keys;
  // 看前三行，params 为空对象
  var params = this.params;

  // m[0] 为路径本身，m[1] 以后的才是路径中的参数
  for (var i = 1; i < m.length; i++) {
    // keys 为 [{name: 'xxx'...}, {name: 'yyy'...}] 形式的数组，name 为参数的名字
    // 例如 /users/:userId 的 keys 为 [{name: 'userId'...}] 
    var key = keys[i - 1];
    var prop = key.name;
    
    // 相当于 decodeURIComponent(m[i])
    var val = decode_param(m[i]);

    // 把路径中匹配到的参数到保存到 params
    if (val !== undefined || !(hasOwnProperty.call(params, prop))) {
      params[prop] = val;
    }
  }

  return true;
};
```

## decode_param 解析路径中的参数


```js

/**
 * Decode param value.
 *
 * @param {string} val
 * @return {string}
 * @private
 */

// 只在 layer.match 中使用
function decode_param(val) {
  if (typeof val !== 'string' || val.length === 0) {
    return val;
  }

  try {
    return decodeURIComponent(val);
  } catch (err) {
    if (err instanceof URIError) {
      err.message = 'Failed to decode param \'' + val + '\'';
      err.status = err.statusCode = 400;
    }

    throw err;
  }
}
```

-----


# Route


## 构造函数

``` js 
/**
 * Initialize `Route` with the given `path`,
 *
 * @param {String} path
 * @public
 */
function Route(path) {
  this.path = path;
  // 在 route.VERB 中会修改这个属性，用于保存 layer 
  this.stack = [];

  debug('new %s', path);

  // route handlers for various http methods
  // Route.prototype[method] 中会通过 this.methods[method] = true 修改这个属性
  this.methods = {};
}
```

## `Route.prototype._handles_method` 判断该路由是否能处理指定的 HTTP 方法

``` js
/**
 * Determine if the route handles a given method.
 * @private
 */
Route.prototype._handles_method = function _handles_method(method) {
  // 调用过 route.all(...) 时，this.methods._all 会变成 true
  if (this.methods._all) {
    return true;
  }

  var name = method.toLowerCase();

  if (name === 'head' && !this.methods['head']) {
    name = 'get';
  }

  return Boolean(this.methods[name]);
};
```


## Route.prototype._options 返回该路由支持的 HTTP 方法

```js
/**
 * @return {Array} supported HTTP methods
 * @private
 */

Route.prototype._options = function _options() {
  var methods = Object.keys(this.methods);

  // append automatic head
  if (this.methods.get && !this.methods.head) {
    methods.push('head');
  }

  for (var i = 0; i < methods.length; i++) {
    // make upper case
    methods[i] = methods[i].toUpperCase();
  }

  return methods;
};
```

## Route.prototype.dispatch 分发 req, res

```js
/**
 * dispatch req, res into this route
 * @private
 */

// Router.route 中会在创建 Layer 的时候调用这个方法，作为 Layer 的第三个参数
Route.prototype.dispatch = function dispatch(req, res, done) {
  // 每次调用 next() 都会增加这个下标
  var idx = 0;
  var stack = this.stack;
  // 当该路由没有处理函数(handler)时，stack 是一个空数组
  if (stack.length === 0) {
    return done();
  }

  var method = req.method.toLowerCase();
  // 对 head 方法进行特殊处理
  if (method === 'head' && !this.methods['head']) {
    method = 'get';
  }

  // 为请求对象 req 添加 route 属性，指向该 route 实例
  req.route = this;

  next();

  // 定义 next 方法
  function next(err) {
    if (err && err === 'route') {
      return done();
    }

    // stack 中保存的是 layer 的列表
    var layer = stack[idx++];
    if (!layer) {
      return done(err);
    }

    // route.all 会把 layer.method 设置为 undefined
    if (layer.method && layer.method !== method) {
      return next(err);
    }

    if (err) {
      layer.handle_error(err, req, res, next);
    } else {
      layer.handle_request(req, res, next);
    }
  }
};
```

## Route.prototype[method] && Route.prototype.all 为所有 HTTP 方法添加处理(handler)方法

```js
/**
 * Add a handler for all HTTP verbs to this route.
 *
 * Behaves just like middleware and can respond or call `next`
 * to continue processing.
 *
 * You can use multiple `.all` call to add multiple handlers.
 *
 *   function check_something(req, res, next){
 *     next();
 *   };
 *
 *   function validate_user(req, res, next){
 *     next();
 *   };
 *
 *   route
 *   .all(validate_user)
 *   .all(check_something)
 *   .get(function(req, res, next){
 *     res.send('hello world');
 *   });
 *
 * @param {function} handler
 * @return {Route} for chaining
 * @api public
 */

// route.all(...) 的实现
Route.prototype.all = function all() {
  // 提取所有中间件， flatten 会把多维数组变成一维数组，即 [fn1, [fn2, fn3]] => [fn1, fn2, fn3]
  var handles = flatten(slice.call(arguments));

  for (var i = 0; i < handles.length; i++) {
    var handle = handles[i];

    // 检查 handle 的类型，必须为 function
    if (typeof handle !== 'function') {
      var type = toString.call(handle);
      var msg = 'Route.all() requires callback functions but got a ' + type;
      throw new TypeError(msg);
    }

    // 新建 layer
    // Layer(path, options, fn)
    // path 无论传什么，layer.path 都会是 undefined
    var layer = Layer('/', {}, handle);
    // 注意，Route.prototype[method] 中这是是 layer.method = method;
    layer.method = undefined;

	// route._handles_method 中会检查这个值
    this.methods._all = true;
    // 保存 layer
    this.stack.push(layer);
  }

  return this;
};

// 基本和 Route.prototype.all 一样
methods.forEach(function(method){
  Route.prototype[method] = function(){
    var handles = flatten(slice.call(arguments));

    for (var i = 0; i < handles.length; i++) {
      var handle = handles[i];

      if (typeof handle !== 'function') {
        var type = toString.call(handle);
        var msg = 'Route.' + method + '() requires callback functions but got a ' + type;
        throw new Error(msg);
      }

      debug('%s %s', method, this.path);

      var layer = Layer('/', {}, handle);
      // 和 .all 中不一样
      layer.method = method;

  	  // route._handles_method 中会检查这个值, route.dispatch 中也会用到，但作用只是检查 head 方法而已
      this.methods[method] = true;
      this.stack.push(layer);
    }

    return this;
  };
});
```

-----

# Router

## 构造函数

```js
var proto = module.exports = function(options) {
  var opts = options || {};

  // 初始化时会调用 router.handle
  function router(req, res, next) {
    router.handle(req, res, next);
  }

  // mixin Router class functions
  // 把 router 的原型对象指向 proto
  router.__proto__ = proto;

  // 初始化属性
  router.params = {};
  router._params = [];
  
  // 下面几个属性会传进在 router.route 中传进 layer 的 options 中
  router.caseSensitive = opts.caseSensitive;
  router.mergeParams = opts.mergeParams;
  router.strict = opts.strict;
  
  // stack 用于保存 route(其实是特殊的 layer)
  // 调用 router.route 或者 router.VERB 的时候会修改这个值
  router.stack = [];

  return router;
};
```

## Router.handle 把 req，res 分发到 router 中


```js
/**
 * Dispatch a req, res into the router.
 * @private
 */
 
// 无返回值
proto.handle = function handle(req, res, out) {
  var self = this;

  debug('dispatching %s %s', req.method, req.url);

  // 用于找查询字符串的下标，如果 url 中没有查询字符串，则 search = 0，下一行就用到了
  var search = 1 + req.url.indexOf('?');
  // 不含查询字符串的 url 的长度
  var pathlength = search ? search - 1 : req.url.length;
  // 判断 url 是否为完全合格域名，不以 '/' 开头的，同时也必须包含 '://' 。
  // 如果满足条件，就返回 '://' 的下标 + 1， 否则为 0 
  var fqdn = req.url[0] !== '/' && 1 + req.url.substr(0, pathlength).indexOf('://');
  // 如果是完全合格域名，就返回该域名。
  // 实现上是直接找 URI 中 host 后面的斜杠，即 https://www.baidu.com/ <-- 这斜杠
  var protohost = fqdn ? req.url.substr(0, req.url.indexOf('/', 2 + fqdn)) : '';
  var idx = 0;
  var removed = '';
  var slashAdded = false;
  var paramcalled = {}; 

  // store options for OPTIONS request
  // only used if OPTIONS request
  // 只用于 http 请求方法中的 options 方法
  var options = [];

  // middleware and routes
  // 获取 self.stack 的引用，这里面保存的都是 layer，即中间件和 route
  var stack = self.stack;

  // manage inter-router variables
  var parentParams = req.params;
  var parentUrl = req.baseUrl || '';
  // restore(fn, obj)，用于重新保存除 fn, obj 以外的其他参数，返回一个函数
  // 这里就是重新保存 baseUrl, next, params
  // done 现在是一个函数，重新保存上面提到的属性之后，会调用 out 函数
  var done = restore(out, req, 'baseUrl', 'next', 'params');

  // setup next layer
  req.next = next;

  // for options requests, respond with a default if nothing else responds
  // 为 options 请求方法返回一个默认的响应
  // wrap(old, fn) 函数只是简单包装一下，fn 的第一个参数会变成 old，其他参数照旧
  // sendOptionsResponse 里面会调用 res.send，这样下面的代码就不会执行了。
  if (req.method === 'OPTIONS') {
    done = wrap(done, function(old, err) {
      if (err || options.length === 0) return old(err);
      sendOptionsResponse(res, options, old);
    });
  }

  // setup basic req values
  req.baseUrl = parentUrl;
  req.originalUrl = req.originalUrl || req.url;

  next();

  function next(err) {
    var layerError = err === 'route'
      ? null
      : err;

    // remove added slash
    // slashAdded 默认为 false
    if (slashAdded) {
      req.url = req.url.substr(1);
      slashAdded = false;
    }

    // restore altered req.url
    if (removed.length !== 0) {
      req.baseUrl = parentUrl;
      req.url = protohost + removed + req.url.substr(protohost.length);
      removed = '';
    }

    // no more matching layers
    if (idx >= stack.length) {
      setImmediate(done, layerError);
      return;
    }

    // get pathname of request
    // 获取请求中的路径，即
    // protocol :// hostname[:port] / path / [;parameters][?query]#fragment
    // 中的 path 部分
    var path = getPathname(req);

    if (path == null) {
      return done(layerError);
    }

    // find next matching layer
    var layer;
    var match;
    var route;

    // 我们知道，router.stack 中保存了很多 layer，这些 layer 都是对应某个 url 的
    // 这里就是要通过 while 循环来找到和 path 对应的 layer
    while (match !== true && idx < stack.length) {
      layer = stack[idx++];
      // 这里封装了 layer.match(path); 为它添加了 try...catch
      // layer.path 就是匹配的值了
      match = matchLayer(layer, path);
      // 还记得吗？router.route 中新建 layer 时会给 layer 添加 route 属性
      route = layer.route;

      //  match 应该都是 boolean 的，不知道哪里会出错
      if (typeof match !== 'boolean') {
        // hold on to layerError
        layerError = layerError || match;
      }

      // 如果路径不匹配，就不会执行下面的代码了，继续下一轮循环
      if (match !== true) {
        continue;
      }

      // 没有 route ，就没有处理函数(handler)
      // 估计这句是为了兼容 router.VERB(...) 的
      if (!route) {
        // process non-route handlers normally
        continue;
      }

      // 处理 layerError
      if (layerError) {
        // routes do not match with a pending error
        match = false;
        continue;
      }

      var method = req.method;
      // 用来判断该 route 是否有处理 method 的方法，返回 boolean
      var has_method = route._handles_method(method);

      // build up automatic options response
      if (!has_method && method === 'OPTIONS') {
        // options 原本是一个空数组
        // route._options() 返回的是该 route 支持的方法
        // appendMethods 可以理解成 options = [].concat(route._options())
        // options 数组里面不会有重复的方法
        appendMethods(options, route._options());
      }

      // don't even bother matching route
      // 注意 HEAD 方法是不会执行下面的代码的
      if (!has_method && method !== 'HEAD') {
        match = false;
        continue;
      }
    }

    // no match
    // 路径不匹配就直接 return ，不在执行下面的代码了
    if (match !== true) {
      return done(layerError);
    }

    // store route for dispatch on change
    // 为 req 添加 route 属性
    if (route) {
      req.route = route;
    }

    // Capture one-time layer values
    // self.mergeParams 是在 Router 的构造函数中通过 opts 传进来的，默认值为 false
    // 默认情况下会以 layer.params 优先
    req.params = self.mergeParams
      ? mergeParams(layer.params, parentParams)
      : layer.params;
    var layerPath = layer.path;

    // this should be done for the layer
    self.process_params(layer, paramcalled, req, res, function (err) {
      if (err) {
        return next(layerError || err);
      }

      // 把 req，res，next 分发到后续的中间件，或者是我们的业务逻辑
      if (route) {
        return layer.handle_request(req, res, next);
      }

      trim_prefix(layer, layerError, layerPath, path);
    });
  }

  // 确保 path 是以 / 开头的
  // 确保 req.baseUrl 不是以 / 结尾
  function trim_prefix(layer, layerError, layerPath, path) {
    var c = path[layerPath.length];
    if (c && '/' !== c && '.' !== c) return next(layerError);

     // Trim off the part of the url that matches the route
     // middleware (.use stuff) needs to have the path stripped
    if (layerPath.length !== 0) {
      debug('trim prefix (%s) from url %s', layerPath, req.url);
      removed = layerPath;
      req.url = protohost + req.url.substr(protohost.length + removed.length);

      // Ensure leading slash
      if (!fqdn && req.url[0] !== '/') {
        req.url = '/' + req.url;
        slashAdded = true;
      }

      // Setup base URL (no trailing slash)
      req.baseUrl = parentUrl + (removed[removed.length - 1] === '/'
        ? removed.substring(0, removed.length - 1)
        : removed);
    }

    debug('%s %s : %s', layer.name, layerPath, req.originalUrl);

    if (layerError) {
      layer.handle_error(layerError, req, res, next);
    } else {
      layer.handle_request(req, res, next);
    }
  }
};

```




## Router.param 获取指定参数的值


```js
/**
 * Map the given param placeholder `name`(s) to the given callback.
 *
 * Parameter mapping is used to provide pre-conditions to routes
 * which use normalized placeholders. For example a _:user_id_ parameter
 * could automatically load a user's information from the database without
 * any additional code,
 *
 * The callback uses the same signature as middleware, the only difference
 * being that the value of the placeholder is passed, in this case the _id_
 * of the user. Once the `next()` function is invoked, just like middleware
 * it will continue on to execute the route, or subsequent parameter functions.
 *
 * Just like in middleware, you must either respond to the request or call next
 * to avoid stalling the request.
 *
 *  app.param('user_id', function(req, res, next, id){
 *    User.find(id, function(err, user){
 *      if (err) {
 *        return next(err);
 *      } else if (!user) {
 *        return next(new Error('failed to load user'));
 *      }
 *      req.user = user;
 *      next();
 *    });
 *  });
 *
 * @param {String} name
 * @param {Function} fn
 * @return {app} for chaining
 * @public
 */

// 这个方法并不会调用 fn，实际调用 fn 的是 router.handle
proto.param = function param(name, fn) {
  // param logic
  if (typeof name === 'function') {
    deprecate('router.param(fn): Refactor to use path params');
    this._params.push(name);
    return;
  }

  // apply param functions
  var params = this._params;
  var len = params.length;
  var ret;

  // 兼容旧写法，旧的写法支持参数以 : 开头，新的写法不需要以 : 开头
  if (name[0] === ':') {
    deprecate('router.param(' + JSON.stringify(name) + ', fn): Use router.param(' + JSON.stringify(name.substr(1)) + ', fn) instead');
    name = name.substr(1);
  }

  for (var i = 0; i < len; ++i) {
    if (ret = params[i](name, fn)) {
      fn = ret;
    }
  }

  // ensure we end up with a
  // middleware function
  if ('function' != typeof fn) {
    throw new Error('invalid param() call for ' + name + ', got ' + fn);
  }

  (this.params[name] = this.params[name] || []).push(fn);
  return this;
};

```


## Router.process_params 为 layer 处理参数

```js
/**
 * Process any parameters for the layer.
 * @private
 */
 // 这个方法里面定义了两个函数，param(err) 和 paramCallback(err)
 // 先执行 param()，如果执行过程报错，就会调用 param(err) 来处理错误
 // 如果没有报错，就会调用 paramCallback()
 // 如果执行过程中报错，就会调用 paramCallback(err) 来处理错误
 // 其实 paramCallback(err) 内部也是调用 param(err) 来处理错误的
 //
 // 这个方法应该是用来处理 router.param(name, fn) 的
 // 因为 router.param 并没有处理参数，而是把 fn 添加到 router.params[name] 中
 // 实际上调用 fn 的是下面这个方法
proto.process_params = function process_params(layer, called, req, res, done) {
  var params = this.params;

  // captured parameters from the layer, keys and values
  // 初始化 layer 的时候会通过 pathRegexp() 来定义 layer.keys
  // 这里的 keys 是 url 中的参数
  // 如 /:foo/:bar/baz 的 keys 就是 [{name: 'foo'...}, {name: 'bar'...}]
  var keys = layer.keys;

  // fast track
  // 没有 keys 的话就不需要处理参数了，直接调用回调函数
  if (!keys || keys.length === 0) {
    return done();
  }

  // 用于遍历 keys 的下标
  var i = 0;
  // key.name，即 url 中的参数名
  var name;
  // 每次调用 param() 都会把 paramIndex 清零
  var paramIndex = 0;
  var key;
  // url 中对应参数的实际值
  var paramVal;
  var paramCallbacks;
  // 下面的 paramCalled 的结构是在 param(err) 函数结束前定义的，形式为:
  // { error: null, match: val, value: val }
  var paramCalled;

  // process params in order
  // param callbacks can be async
  function param(err) {
    if (err) {
      return done(err);
    }

    if (i >= keys.length ) {
      return done();
    }

    paramIndex = 0;
    key = keys[i++];

    if (!key) {
      return done();
    }

    name = key.name;
    paramVal = req.params[name];
    // params 即 this.params
    // 翻了一下代码，只有 router.param(name, fn) 会修改这个值
    // 难道这行是为了处理 router.param 的？
    // 另外， params[name] 会返回一个数组，里面包含 fn
    paramCallbacks = params[name];
    // called 是在 router.handle 传进来的，默认为 {}
    // 该值会在本函数递归调用时修改
    paramCalled = called[name];

    if (paramVal === undefined || !paramCallbacks) {
      return param();
    }

    // param previously called with same value or error occurred
    if (paramCalled && (paramCalled.match === paramVal
      || (paramCalled.error && paramCalled.error !== 'route'))) {
      // restore value
      req.params[name] = paramCalled.value;

      // next param
      return param(paramCalled.error);
    }

    called[name] = paramCalled = {
      error: null,
      match: paramVal,
      value: paramVal
    };

    paramCallback();
  }

  // single param callbacks
  function paramCallback(err) {
    // paramCallbacks 是通过 this.params[name] 来得到的，是一个保存了 fn 的数组
    var fn = paramCallbacks[paramIndex++];

    // store updated value
    paramCalled.value = req.params[key.name];

    if (err) {
      // store error
      paramCalled.error = err;
      param(err);
      return;
    }

    if (!fn) return param();

    try {
      fn(req, res, paramCallback, paramVal, key.name);
    } catch (e) {
      paramCallback(e);
    }
  }

  param();
};
```


## Router.use 加载特定中间件

``` js
/**
 * Use the given middleware function, with optional path, defaulting to "/".
 *
 * Use (like `.all`) will run for any http METHOD, but it will not add
 * handlers for those methods so OPTIONS requests will not consider `.use`
 * functions even if they could respond.
 *
 * The other difference is that _route_ path is stripped and not visible
 * to the handler function. The main effect of this feature is that mounted
 * handlers can operate without any code changes regardless of the "prefix"
 * pathname.
 *
 * @public
 */

// 中间件默认挂载在 / 下
proto.use = function use(fn) {
  var offset = 0;
  var path = '/';

  // default path to '/'
  // disambiguate router.use([fn])
  if (typeof fn !== 'function') {
    var arg = fn;

    while (Array.isArray(arg) && arg.length !== 0) {
      arg = arg[0];
    }

    // first arg is the path
    // 处理 router.use('/foo', bar)
    // 注意这里只是设置了偏移值
    if (typeof arg !== 'function') {
      offset = 1;
      path = fn;
    }
  }

  // 获取 .use 中所有函数
  // 如 router.use('/test', fn1, fn2, fn3); 
  // callbacks = [fn1, fn2, fn3]
  var callbacks = flatten(slice.call(arguments, offset));

  if (callbacks.length === 0) {
    throw new TypeError('Router.use() requires middleware functions');
  }

  // 遍历 callbacks，每个 callback 都会成为一个 layer，并按顺序进入 Router.stack
  for (var i = 0; i < callbacks.length; i++) {
    var fn = callbacks[i];

    if (typeof fn !== 'function') {
      throw new TypeError('Router.use() requires middleware function but got a ' + gettype(fn));
    }

    // add the middleware
    debug('use %s %s', path, fn.name || '<anonymous>');

    var layer = new Layer(path, {
      sensitive: this.caseSensitive,
      strict: false,
      end: false
    }, fn);

    layer.route = undefined;
    
    this.stack.push(layer);
  }

  return this;
};
```

## Router.route 为 router 创建新的路由
```js
/**
 * Create a new Route for the given path.
 *
 * Each route contains a separate middleware stack and VERB handlers.
 *
 * See the Route api documentation for details on adding handlers
 * and middleware to routes.
 *
 * @param {String} path
 * @return {Route}
 * @public
 */

proto.route = function route(path) {
  // 创建新的路由
  var route = new Route(path);

  // 创建新的 layer
  var layer = new Layer(path, {
    sensitive: this.caseSensitive,
    strict: this.strict,
    end: true
    
    // 为这个特殊的 layer 提供分发 req，res 的能力
  }, route.dispatch.bind(route));

  layer.route = route;

  // 其实 route 也是一个 layer
  this.stack.push(layer);
  return route;
};
```

## Router.VERB 为 router 提供 VERB 方法

```js
// create Router#VERB functions
// 提供方便的语法来创建 route
methods.concat('all').forEach(function(method){
  proto[method] = function(path){
    // 只是简单的调用上面提到的 Router.route
    var route = this.route(path)
    route[method].apply(route, slice.call(arguments, 1));
    return this;
  };
});
```


## helpers 只有 Router 中才用到的 helpers，属于私有 API
```js
// append methods to a list of methods
// 为给定列表添加不重复的元素
function appendMethods(list, addition) {
  for (var i = 0; i < addition.length; i++) {
    var method = addition[i];
    if (list.indexOf(method) === -1) {
      list.push(method);
    }
  }
}

// get pathname of request
// 获取请求中的 pathname
function getPathname(req) {
  try {
    return parseUrl(req).pathname;
  } catch (err) {
    return undefined;
  }
}

// get type for error message
function gettype(obj) {
  var type = typeof obj;

  if (type !== 'object') {
    return type;
  }

  // inspect [[Class]] for objects
  return toString.call(obj)
    .replace(objectRegExp, '$1');
}

/**
 * Match path to a layer.
 *
 * @param {Layer} layer
 * @param {string} path
 * @private
 */
// 判断给定的 layer 是否与路径匹配
function matchLayer(layer, path) {
  try {
    return layer.match(path);
  } catch (err) {
    return err;
  }
}

// merge params with parent params
function mergeParams(params, parent) {
  if (typeof parent !== 'object' || !parent) {
    return params;
  }

  // make copy of parent for base
  var obj = mixin({}, parent);

  // simple non-numeric merging
  if (!(0 in params) || !(0 in parent)) {
    return mixin(obj, params);
  }

  var i = 0;
  var o = 0;

  // determine numeric gaps
  while (i in params) {
    i++;
  }

  while (o in parent) {
    o++;
  }

  // offset numeric indices in params before merge
  for (i--; i >= 0; i--) {
    params[i + o] = params[i];

    // create holes for the merge when necessary
    if (i < o) {
      delete params[i];
    }
  }

  return mixin(obj, params);
}

// restore obj props after function
function restore(fn, obj) {
  var props = new Array(arguments.length - 2);
  var vals = new Array(arguments.length - 2);

  for (var i = 0; i < props.length; i++) {
    props[i] = arguments[i + 2];
    vals[i] = obj[props[i]];
  }

  return function(err){
    // restore vals
    for (var i = 0; i < props.length; i++) {
      obj[props[i]] = vals[i];
    }

    return fn.apply(this, arguments);
  };
}

// send an OPTIONS response
function sendOptionsResponse(res, options, next) {
  try {
    var body = options.join(',');
    res.set('Allow', body);
    res.send(body);
  } catch (err) {
    next(err);
  }
}

// wrap a function
function wrap(old, fn) {
  return function proxy() {
    var args = new Array(arguments.length + 1);

    args[0] = old;
    for (var i = 0, len = arguments.length; i < len; i++) {
      args[i + 1] = arguments[i];
    }

    fn.apply(this, args);
  };
}
```