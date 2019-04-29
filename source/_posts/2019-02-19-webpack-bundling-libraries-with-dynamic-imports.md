---
title: Webpack 打包含动态加载的类库
date: 2019-02-19 12:11:01
categories: [javascript, webpack]
tags: [webpack]
---


## 前言
在编写库的时候，我们有时候会希望按需加载某些依赖，例如如果代码的运行环境不支持某些功能的话，就加载相关的 Polyfill 。
webpack 作为当前最流行的打包工具，早已支持动态加载的功能了。
本文将讨论一种用 webpack 打包含动态加载的类库的方法。
**注意，本文是写给类库作者看的，如果读者写的是应用，那就没有必要往下看了。**


## 示例类库

```js
// my-lib.js
class MyLib {
    loadDeps() {
        return new Promise((resolve, reject) => {
            if (global.TextDecoder === undefined) {
                return Promise.all([
                    import('./deps/text-encoding'),
                    import('./deps/other-dep.js'),
                    import('./deps/another-dep.js'),
                ]).then(resolve).catch(reject);
            } else {
                resolve();
            }
        });
    }

    initialize() {
        this.decoder = new TextDecoder();
        // ...
    }
}
```

这个类库中有个 `loadDeps` 方法，会根据运行环境检测 `TextDecoder` 是否存在，如果不存在就会去加载依赖，这些依赖是在本地目录中。
当我们把类库发布到 npm 之后，我们就会这么使用它：

```js
// app.js
import MyLib from 'my-lib';
class App {
    constructor() {
        this.myLib = new MyLib();
    }
}

const app = new App();
app.myLib.loadDeps().then(() => {
    app.myLib.initialize();
    console.log(app.myLib.decoder);
});
```

这样，我们的类库无论在什么环境都能使用到 `TextDecoder` 了。

**但，真的会这么顺利吗？**


## 问题

当我们发布到 npm 仓库之前，我们会先用 Webpack 构建项目，这时候 webpack 就会分析文件中的依赖路径，把依赖文件生成到 `output.path` 中，同时 `import()`方法就已经变成 webpack 内部的方法了，依赖的路径也变成 `output.publicPath` 相关的了。

假设我们的 webpack 配置是这样的：

```js
// webpack.config.js
module.exports = {
    output: {
        filename: 'index.js',
        chunkFilename: '[name].js',
        path: 'dist/',
        publicPath: '/',
        libraryTarget: 'umd'
    },
    // ...
};
```

然后输出了这些文件：

```
// webpack output
Built at: 02/19/2019 5:08:41 PM
  Asset      Size  Chunks             Chunk Names
   1.js  79 bytes       1  [emitted]
   2.js  79 bytes       2  [emitted]
   3.js  79 bytes       3  [emitted]
index.js   144 KiB       0  [emitted]  main
Entrypoint main = index.js
```

那么当应用层调用 `app.myLib.loadDeps()` 的时候会加载依赖的路径会是怎样的呢？
猜对了，浏览器会尝试加载这些路径：

```
/1.js
/2.js
/3.js
```

但是结果呢？当然是 `404` 了。因为网站的根目录没有这些文件呀，除非你把这些文件复制到网站根目录。
同样道理，如果 `publicPath` 是相对路径的话(假设是 `''`)，那么请求依赖的路径就会相对于当前的 URL 了，即如果当前 URL 是 `/topics/:uid`，那请求的路径就会是 `/topics/:uid/1.js`。除非当前目录下有这些依赖，不然结果还是 `404`。
当然，我们还可以修改服务端配置，把 `/1.js` 和 `**/1.js` 都指向 `/path/to/project/node_modules/my-lib/dist/1.js`。
对应用层来说，为了一个库而改服务器配置就显得太麻烦了。
对一个类库来说，它应该管理好自己的依赖，不应该让应用层甚至是服务器端来配合。


## 解决方案

我们需要解决这个路径问题，既要保证结果正确，又要方便开发。
很明显，这个问题是因为 webpack 打包的时候处理了 `import()` 导致的，如果我们不在编译时处理，而是运行时处理，这不就可以达到目的了吗？

先来准备一下 `dist/` 的目录结构， 修改 webpack 的配置：

```js
const CopyWebpackPlugin = require('copy-webpack-plugin');

module.exports = {
    output: { ... },
    plugins: [
       new CopyWebpackPlugin([{
           from: 'src/deps/',
           to: '.'
       }]) 
    ]
}
```

`CopyWebpackPlugin` 会把 `src/deps/` 中的文件复制到 `dist/` 目录中，这时候 `dist/` 就会变成：

```
dist
├── 1.js
├── 2.js
├── 3.js
├── text-encoding.js
├── other-dep.js
├── another-dep.js
└── index.js
```

下一步就是让 webpack 不解析 `import()` 了，这里有两种做法：

### 方案一：由应用层处理

这个方案很容易实现，只需要让应用层来调用 `import()` 就可以了。

```js
// my-lib.js
class MyLib {
    constructor(options) {
        this.onLoadDeps = options.onLoadDeps || null;
    }

    loadDeps() {
        return new Promise((resolve, reject) => {
            if (global.TextDecoder === undefined) {
                if (this.onLoadDeps) {
                   this.onLoadDeps().then(resolve).catch(reject);
                } else {
                    Promise.all([
                        import('./deps/text-encoding'),
                        import('./deps/other-dep.js'),
                        import('./deps/another-dep.js'),
                    ]).then(resolve).catch(reject);
                }
            } else {
                resolve();
            }
        });
    }
}

// app.js
import MyLib from 'my-lib';
class App {
    constructor() {
        this.myLib = new MyLib({
            onLoadDeps: () => Promise.all([
                import('my-lib/dist/text-encoding'),
                import('my-lib/dist/other-dep.js'),
                import('my-lib/dist/another-dep.js'),
            ]);
        });
    }
}
```

这种做法非常简单，而且在开发环境下也不需要任何处理就能正常运行了。
不足之处是如果有多个项目都引用了这个类库的话，那么当类库添加了新的依赖时，所有引用了这个类库的项目都要改动源码。这会是一个比较繁琐的事情。
对于这个解决方案，其实还有一个变种，相对来说更加方便：

```js
// my-lib.js
class MyLib {
    constructor(options) {
        this.importPrefix = options.importPrefix || './deps';
    }

    loadDeps() {
        return new Promise((resolve, reject) => {
            if (global.TextDecoder === undefined) {
                return Promise.all([
                    import(this.importPrefix + '/text-encoding'),
                    import(this.importPrefix + '/other-dep.js'),
                    import(this.importPrefix + '/another-dep.js'),
                ]).then(resolve).catch(reject);
            } else {
                resolve();
            }
        });
    }
}

// app.js
import MyLib from 'my-lib';
class App {
    constructor() {
        this.myLib = new MyLib({ importPrefix: 'my-lib/dist' });
    }
}
```

**注意：这个变种方案有可能会出现 Critical dependency: the request of a dependency is an expression 的报错。**


### 方案二：由类库处理

这个方案稍微复杂一点点。
想要 webpack 不处理 `import()`，那么就不能让 webpack 去解析含有 `import()` 的文件，即需要把含有加载依赖的部分分离到另一个文件中。

```js
// runtime.js
module.exports = {
    onLoadDeps: function() {
        return Promise.all([
            import('my-lib/dist/text-encoding'),
            import('my-lib/dist/other-dep.js'),
            import('my-lib/dist/another-dep.js'),
        ]);
    }
}
```

**注意：因为 webpack 不会解析这个文件，loader 就不会处理这个文件，所以这个文件里面最好使用 Node.js 原生支持的语法。**

```js
// my-lib.js
import RUNTIME from './runtime';

class MyLib {
    loadDeps() {
        return new Promise((resolve, reject) => {
           if (global.TextDecoder === undefined) {
               RUNTIME.onLoadDeps().then(resolve).catch(reject);
           } else {
               resolve();
           }
        });
    }
}
```

然后修改 webpack 配置：

```js
module.exports = {
    output: { ... },
    module: {
        noParse: /src\/runtime/,
    },
    plugins: [ ... ] 
}
```

这样，webpack 在处理 `my-lib.js` 的时候会把 `runtime.js` 加载进来，但不会去解析它。所以会得到以下的结果：

```js
// dist/index.js
/******/ ([
/* 0 */
/***/ (function(module, exports) {

module.exports = {
  onLoadDeps: function onLoadDeps() {
    return Promise.all([import('my-lib/dist/text-encoding'), import('my-lib/dist/other-dep.js'), import('my-lib/dist/another-dep.js')]);
  }
};

/***/ }),
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

// ...

var MyLib =
/*#__PURE__*/
function () {
  function MyLib() {}

  var _proto = MyLib.prototype;

  _proto.loadDeps = function loadDeps() {
    var _this = this;

    return new Promise(function (resolve, reject) {
      if (global.TextDecoder === undefined) {
        _runtime__WEBPACK_IMPORTED_MODULE_0___default.a.onLoadDeps().then(resolve).catch(reject);
      } else {
        resolve();
      }
    });
  };

  _proto.initialize = function initialize() {
    this.decoder = new TextDecoder(); // ...
  };

  return MyLib;
}();

// ...
```

如果应用层引用了这个类库，那么 webpack 打包应用的时候就会处理类库中的 `import()`，这样就和应用层平时的动态加载一样了，上面的问题也就解决了。
最后剩下一个问题，那就是在开发环境下，我们也需要测试 `runtime.js`，但这时候它是 `import('my-lib/dist/xxx')` 的，这个肯定会报 `Error: Cannot find module` 的错误。
这时候可以像方案一那样，用 `import(importPrefix + '/text-encoding')` 的方式来解决，也可以利用 `NormalModuleReplacementPlugin` 来解决。

```js
// webpack.dev.js
module.exports = {
    // ...
    plugins: [
        new webpack.NormalModuleReplacementPlugin(/my-lib\/dist\/(.*)/, function (resource) {
            resource.request = resource.request.replace(/my-lib\/dist/, '../src/deps')
        }),
    ]
}
```

这个插件可以改变重定向资源，上面的配置是把 `my-lib/dist/*` 里面的资源都重定向到 `../src/deps`。
更多详细的用法，可以参考下官方文档 [NormalModuleReplacementPlugin](https://webpack.js.org/plugins/normal-module-replacement-plugin/)。
**注意：这个插件最好只用在开发环境中。**

这个方案虽然有些繁琐，但最大的优点是可以把依赖管理都交给类库自己处理，不需要应用层干预。就算类库日后改变打包位置（不再是 `dist/`） 也无需让应用层知道。

完。
