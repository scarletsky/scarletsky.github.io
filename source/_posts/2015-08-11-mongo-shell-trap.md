---
title: Mongo Shell 使用过程中遇到的坑
date: 2015-08-11 12:17:47
categories: mongodb
tags: mongodb
---

# 编写 js 脚本

+ js 脚本中不能包含 `use` 关键字，需要用 `$ mongo mydb xxx.js` 这样的方式来指定数据库

+ 不能用 `console.log`，需要用 `print` 代替

+ 调用 `ObjectId()` 时需要确保传进去的值为字符串，否则会出现 `Error: invalid object id: length`

+ 使用 `$set` 来设置数字时需要注意，mongo shell 默认会把所有数字类型的值转换成浮点型，如果你需要插入整形，你需要用 `NumberInt()` 或 `NumberLong()` 来代替。

``` javascript
$ mongo mydb test.js

// test.js
var users = db.users.find({});
users.forEach(function(u) {

    // not console.log(u._id) !!
    // use print instead of console!
    print(u._id);

    db.users.update({
        _id: ObjectId('' + u._id) // make sure the params is string
    }, {
        $set: {
            money: NumberInt(100) // it will be int, not float.
        }
    });
});
```


# 参考资料

http://stackoverflow.com/questions/8218484/mongodb-inserts-float-when-trying-to-insert-integer

http://docs.mongodb.org/manual/core/shell-types/
