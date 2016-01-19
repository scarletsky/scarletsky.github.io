---
title: JavaScript 利用 Blob 进行分片上传
date: 2015-01-27 17:28:00
categories: javascript
---

# 基本流程

- 用户选择文件

- 判断文件大小

  - 如果文件小于限定文件大小，则直接上传

  - 否则进行分片上传

-----

# 分片流程

- 给定限定大小。

- 利用 `Blob` 对象的 `slice` 方法把文件分成 N 份。

- 遍历 N 次，利用 `FormData` 创建需要提交的数据，上传数据。

-----

# 服务端处理流程

- 接收数据

- 判断数据总份数

  - 如果总份数为 1，则直接保存成文件，文件名不需要改变。保存完成后，直接返回信息给客户端。

  - 否则保存成文件时，文件名后缀名需要带上 .partX 来表示该文件为第几部分。

- 判断已上传的数据是否等于总份数。

  - 如果相等，则合并文件。

  - 合并完成后，删除带有 .partX 的文件。

-----

# 代码实例

## 客户端代码 - html
```html
<input type="file" id="upload"/>
```

## 客户端代码 - JavaScript

```js
function uploadFile (url, blob, callback) {
    var perFileSize = 2097152; // 2 * 1024 * 1024
    var blobParts = Math.ceil(blob.size / perFileSize);

    for (var i = 0; i < blobParts; i++) {
        (function (i) {
            var fd = new FormData();
            var _blob = blob.slice(i * perFileSize, (i + 1) * perFileSize);

            fd.append('_blob', _blob);
            fd.append('filename', blob.name);
            fd.append('index', i + 1);
            fd.append('total', blobParts);

            $.ajax({
                type: 'POST',
                url: url,
                data: fd,
                processData: false,
                contentType: false
            }).done(function (res) {
                console.log('upload status: ');
                console.log('this is ' + (i + 1) + 'part, total ' + blobParts + ' part(s).');

                if (res.statusCode === 200) {
                    callback(null, res);
                }
            }).fail(function (err) {
                callback(err, null);
            });
        })(i)
    }
}

$('#upload').on('change', function (e) {
    var file = e.target.files[0];
    uploadFile('/upload', file, function (err, res) {
        if (err) { return console.log(err); }
        console.log('upload successfully!');
    });
});
```

## 服务端代码 - NodeJS
```js
// Expressjs
var multipart = require('connect-multiparty');
var multipartMiddleware = multipart();

var uploadDir = './upload/';

// 合并文件
function mergeFiles(fileName, fileParts) {
    var buffers = [];

    // 获取各个部分的路径
    var filePartsPaths = fileParts.map(function(name) {
        return path.join(uploadDir, name);
    });

    // 获取各个 part 的 buffer，并保存到 buffers 中
    filePartsPaths.forEach(function(path) {
        var buffer = fs.readFileSync(path);
        buffers.push(buffer);
    });

    // 合并文件
    var concatBuffer = Buffer.concat(buffers);
    var concatFilePath = path.join(uploadDir, fileName);
    fs.writeFileSync(concatFilePath, concatBuffer);

    // 删除各个 part 的文件
    filePartsPaths.forEach(function(path) {
        fs.unlinkSync(path);
    });
}


function upload (req, res) {
    if (req.method === 'POST') {
        var data = req.body;
        var _blobPath = req.files._blob.path;
        var fileName = data.filename;
        var filePath;
        var total = parseInt(data.total);

        // 处理文件路径
        if (total === 1) {
            filePath = path.join(uploadDir, fileName);
        } else {
            var fileNameWithPart = fileName + '.part' + data.index;
            filePath = path.join(uploadDir, fileNameWithPart);
        }

        // 读取上传的数据，保存到指定路径
        var tmpBuffer = fs.readFileSync(_blobPath);
        fs.writeFileSync(filePath, tmpBuffer);

        // 判断是否上传完成
        if (total === 1) {
            res.send(200);
        } else {
            // 获取指定目录下的所有文件名
            var filesInDir = fs.readdirSync(uploadDir);
            // 找出指定文件名中带有 .part 的文件
            var fileParts = filesInDir.filter(function(name) {
                return name.substring(0, fileName.length + 5) === (fileName + '.part');
            });

            // 判断是否需要合并文件
            if (fileParts.length === total) {
                mergeFiles(fileName, fileParts);
                res.send(200);
            } else {
                res.send(204);
            }
        }

    } else {
        res.send(405);
    }
}

app.post('/upload', multipartMiddleware, upload);

```
