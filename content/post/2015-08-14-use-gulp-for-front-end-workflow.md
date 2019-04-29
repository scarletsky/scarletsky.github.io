---
title: 利用 Gulp 处理前端工作流程
date: 2015-08-14 16:57:33
categories: [javascript]
tags: [gulp]
---

# 基本用法

```
// gulpfile.js
gulp.task('foo', function() {
    gulp.src(glob)
        .pipe(...)
        .pipe(...)
        .pipe(gulp.dest(...))
        .pipe(...)
        .pipe(gulp.dest(...))
});

// shell
$ gulp foo
```


# 基本 API

+ `gulp.src(glob[, options])`

    - 根据 `glob` 匹配文件，返回 `stream`，可以通过 `.pipe()` 方法传递给后续的插件。

+ `gulp.dest(path[, options])`

    - 一般用法 `.pipe(gulp.dest(path))`，把 `pipe` 中的内容按照指定的 `path` 写成文件，会自动创建不存在的文件夹。

    - 注意，可以通过 `.pipe` 多次指定输出的地方，具体请看 [这里](https://github.com/gulpjs/gulp/blob/master/docs/API.md#gulpdestpath-options)

+ `gulp.task(name[, deps], fn)`

    - 定义名为 `name` 的任务，定义之后就可以在命令行中使用 `gulp xxx` 来执行任务。

    - `deps` 里面的任务全部完成后才会执行 `fn`

    - `deps` 里面的任务都是并行执行的，如果需要顺序执行，需要特殊写法。具体看 [这里](https://github.com/gulpjs/gulp/blob/master/docs/API.md#return-a-promise)

+ `gulp.watch(glob[, opts, cb])`

    - 监听文件变化

    - 根据[这个帖子](http://stackoverflow.com/questions/22391527/gulps-gulp-watch-not-triggered-for-new-or-deleted-files)，`gulp.watch` 不会监听新文件（目录），所以一般你会需要 [gulp-watch](https://github.com/floatdrop/gulp-watch)


# 常用命令 （自定义）
``` bash
# for development mode
gulp server

# run test
gulp test

# for production mode
gulp build
```

## `gulp server` 流程

+ 把 `less`, `sass`, 之类的文件编译成 CSS，常用插件：

    - [gulp-less](https://github.com/plus3network/gulp-less)

    - [gulp-sass](https://github.com/dlmanning/gulp-sass)

+ 创建 Web Server (with Live Reload)，常用：

    - [gulp-connect](https://github.com/AveVlad/gulp-connect)，用来创建 Web Server，其实还有其他选择的，但多数都是利用 [connect](https://github.com/senchalabs/connect) 来创建 Web Server 的。

    - [node-proxy-middle](https://github.com/andrewrk/node-proxy-middleware)，用来代理请求，可以把 `/api/xxx` 发送到指定的地址。(常用于 SPA 开发)

    - [connect-modrewrite](https://github.com/tinganho/connect-modrewrite)，匹配资源，如果不匹配就可以重定向到指定地址。(常用于 SPA 开发)

    - [connect-history-api-fallback](https://github.com/bripkens/connect-history-api-fallback)，作用同上，也用于匹配资源，但用起来简单很多。(常用于 SPA 开发)

+ 监听文件变化，常用插件：

    - [gulp-watch](https://github.com/floatdrop/gulp-watch)

### 示例代码
```js
gulp.task('clean:css', function () {
    del.sync('app/styles/*.css');
});

gulp.task('less', ['clean:css'], function () {
    var stream = gulp
            .src('app/styles/main.less')
            .pipe(less())
            .pipe(gulp.dest('app/styles/'));
    return stream;
});

gulp.task('connect', function () {
    connect.server({
        root: './app',
        port: 9000,
        livereload: true,
        middleware: function (connect, o) {
            return [
                (function () {
                    var url = require('url');
                    var proxy = require('proxy-middleware');
                    var options = url.parse('http://localhost:3000/api');
                    options.route = '/api';
                    return proxy(options);
                })(),
                modRewrite([
                    '!\\.html|\\.js|\\.css|\\.swf|\\.jp(e?)g|\\.png|\\.gif|\\.eot|\\.woff|\\.ttf|\\.svg$ /index.html'
                ])
            ];
        }
    });
});

gulp.task('watch', function () {
    gulp
        .src('app/styles/**/*.less', {read: false})
        .pipe(watch('app/styles/**/*.less', function () {
            return gulp
                .src('app/styles/main.less')
                .pipe(less())
                .pipe(gulp.dest('app/styles/'))
                .pipe(connect.reload());
        }));

    gulp
        .src(['app/scripts/**/*.js', 'app/**/*.html'])
        .pipe(watch(['app/scripts/**/*.js', 'app/**/*.html']))
        .pipe(plumber())
        .pipe(connect.reload());
});

gulp.task('server', ['less', 'connect', 'watch']);
```

## `gulp build` 流程

+ 清理 `dist/` 文件夹

    - [del](https://github.com/sindresorhus/del)，根据 `glob` 来删除文件/目录

+ 压缩文件

    - [gulp-htmlmin](https://github.com/jonschlinkert/gulp-htmlmin)，压缩 `html` 文件

    - [gulp-minify-html](https://github.com/murphydanger/gulp-minify-html)，同上

    - [gulp-cssmin](https://github.com/chilijung/gulp-cssmin)，压缩 `css` 文件

    - [gulp-minify-css](https://github.com/murphydanger/gulp-minify-css)，同上，封装了 [clean-css](https://github.com/jakubpawlowicz/clean-css)，star 比上面的多

    - [gulp-uglify](https://github.com/terinjokes/gulp-uglify)，混淆 `JavaScript` 代码

    - [gulp-usemin](https://github.com/zont/gulp-usemin)，合并指定 `block` 中的文件

    - [gulp-rev](https://github.com/sindresorhus/gulp-rev)，给静态文件加上版本号，如 `app.js` -> `app-d41d8cd98f.js`

+ 复制其他文件到 `dist/`

    - `gulp.src(...).pipe(gulp.dest(...))`

### 实例代码
```js
gulp.task('clean:build', function () {
    del.sync('dist/', {force: true});
});

gulp.task('minify', ['clean:build', 'less'], function () {
    gulp
        .src('app/views/**/*.html')
        .pipe(htmlmin({collapseWhitespace: true}))
        .pipe(gulp.dest('dist/views'));

    gulp
        .src('app/index.html')
        .pipe(usemin({
            js: [uglify(), rev()],
            css: [minifyCss(), 'concat', rev()]
        }))
        .pipe(gulp.dest('dist/'));
});

gulp.task('copyfonts', function () {
    gulp
        .src('app/styles/fonts/*')
        .pipe(gulp.dest('dist/fonts/'));
});

gulp.task('build', ['clean:build', 'minify', 'copyfonts']);
```

# 参考资料
[英文文档](https://github.com/gulpjs/gulp/blob/master/docs/API.md)
[中文文档](http://www.gulpjs.com.cn/docs/api/)
