---
title: 「译」 MapReduce in MongoDB
date: 2016-06-12 10:04:53
tags: [mongodb]
---

在这篇文章里面，我们会演示如何在 MongoDB 中使用 MapReduce 操作。
我们会用 `dummy-json` 这个包来生成一些虚假的数据，然后用 `Mongojs`

如果想要快速看到结果，可以到 [这里](http://code.runnable.com/U1fmE30iHWMIGY_r/mapreduce-in-mongodb-for-node-js) 里看看。

## 什么是 MongoDB ?

MongoDB 是一个 NoSQL 数据库，不像 MySQL 、MSSQL 和 Oracle DB 那样，MongoDB 使用集合(collections) 来代替表(tables)。同时，它用集合中的文档(documents)来代替表中的行(rows)。还有最好的一点是，所有文档都保存成 JSON 格式！你可以到[这里](http://try.mongodb.org/)学更多关于 MongoDB 的知识。

你可以从 [这里](http://www.mongodb.org/downloads) 下载安装 MongoDB。

如果以前没用过 MongoDB，那么你可以记住下面这些命令:

| Command             | Result
|---------------------|--------------------- |
| `mongod`            | 启动 MongoDB 服务 |
| `mongo`             | 进入 MongoDB Shell |
| `show dbs`          | 显示所有数据库列表 |
| `use <db name>`     | 进入指定的数据库 |
| `show collections`  | 进入数据库之后，显示该数据库中所有的集合 |
| `db.collectionName.find()`    | 显示该集合中所有文档 |
| `db.collectionName.findOne()` | 显示该集合中第一个文档 |
| `db.collectionName.find().pretty()` | 显示漂亮的 JSON 格式 |
| `db.collectionName.insert({key: value})` | 插入一条新的记录 |
| `db.collectionName.update({ condition: value}, {$set: {key: value}}, {upsert: true})` | 会更新指定的文档，设置指定的值。如果 `upsert` 为 `true`，当没有找到匹配的文档时，会创建一条新的记录 |
| `db.collectionName.remove({})` | 移除集合中的所有文档 |
| `db.collectionName.remove({key: value})` | 移除集合中匹配到的文档 |


## 什么是 MapReduce ?

弄清楚 MapReduce 是如何运作的是非常重要的，如果对 MapReduce 过程不了解的话，你在运行 MapReduce 时很可能得不到你想要的结果。

从 mongodb.org 上的解析：

> Map-reduce 是一种数据处理范例，用于将大量的数据变成有用的聚合结果。 对于 map-reduce 操作，MongoDB 提供了 mapReduce 的数据库命令。

在这非常简单的术语里面，mapReduce 命令接受两个基本的输入：mapper 函数和 reducer 函数。

Mapper 是一个匹配数据的过程，它会在集合中查询我们想要处理的字段，然后根据我们指定的 key 去分组，再把这些 key-value 对交给 reducer 函数，由它来处理这些匹配到的数据。

我们来看看下面这些数据：

```json
[
  { name: foo, price: 9 },
  { name: foo, price: 12 },
  { name: bar, price: 8 },
  { name: baz, price: 3 },
  { name: baz, price: 5 }
]
```

我们想要计算出相同名字下的所需要的价钱。我们将会用这个数据通过 Mapper 和 Reducer 去获得结果。

当我们让 Mapper 去处理上面的数据时，会生成如下的结果：

| Key     | Value     |
|---------|-----------|
| foo     | [9,12]    |
| bar     | [8]       |
| baz     | [3,5]     |

看到了吗？它用相同的 key 去分组数据。在我们的例子中，是用 name 分组。这些结果会发送到 Reducer 中。

现在，在 reducer 中，我们会得到上面表格中的第一行数据，然后迭代这些数据然后把它们加起来，这就是第一行数据的总和。然后 reducer 会对第二行数据做同样的事情，直到所有行被处理完。

最终的输出结果如下：

| Name     | Total     |
|----------|-----------|
| foo      | 21        |
| bar      | 8         |
| baz      | 8         |

现在你明白为什么 Mapper 会叫 Mapper 了吧 ! (因为它会创建一份数据的映射)
也明白了为什么 Reducer 会叫 Reducer 了吧 ! (因为它会把 Mapper 生成的数据归纳成一个简单的形式)

如果你运行一些例子，你就会知道它是怎么工作的拉。你也可以从[官方文档](https://docs.mongodb.com/manual/core/map-reduce/) 中了解更多细节。


## 创建一个项目

正如上文所说，我们可以在 mongo shell 中直接查询和看到输出结果。但是，为了让教程更加丰富，我们会构建一个 Nodejs 项目，在里面运行我们之前的任务。

### Mongojs

我们会用 `mongojs` 去实现我们的 MapReduce。你可以用同样的代码跑在 mongo shell 里面，会看到同样的结果。

### Dummy-json

我们会用 `dummy-json` 去创建一些虚假的数据。你可以在 [这里](https://github.com/webroo/dummy-json) 找到更多的信息。然后我们会在这些虚假数据上面运行 MapReduce 命令，生成一些有意义的结果。

我们开始吧！

首先，你要安装 Nodejs，你可以看看 [这里](http://thejackalofjavascript.com/hello-node/)。然后你要创建一个叫 mongoDBMapReduce 的目录。我们将会创建 `package.json` 文件来保存项目的详细信息。

运行 `npm init` 然后填入你喜欢的东西，创建完 `package.json` 后，我们要添加项目的依赖。
运行 `npm i mongojs dummy-json --save-dev` ，然后等几分钟之后，我们项目的依赖就安装好了。

## 生成虚假数据

下一步，我们要用 `dummy-json` 模块来生成虚假数据。
在项目的根目录创建一个名叫 `dataGen.js` 的文件，我们会把数据生成的逻辑保存到一个独立的文件里面。如果以后需要添加更多的数据，你可以运行这个文件。

把下面的内容复制到 `dataGen.js` 里面：

```js
var mongojs = require('mongojs');
var db = mongojs('mapReduceDB', ['sourceData']);
var fs = require('fs');
var dummyjson = require('dummy-json');
 
var helpers = {
  gender: function() {
    return ""+ Math.random() > 0.5 ? 'male' : 'female';
  },
  dob : function() {
    var start = new Date(1900, 0, 1),
        end = new Date();
        return new Date(start.getTime() + Math.random() * (end.getTime() - start.getTime()));
    },
  hobbies : function () {
    var hobbysList = []; 
    hobbysList[0] = [];
    hobbysList[0][0] = ["Acrobatics", "Meditation", "Music"];
    hobbysList[0][1] = ["Acrobatics", "Photography", "Papier-Mache"];
    hobbysList[0][2] = [ "Papier-Mache"];
    return hobbysList[0][Math.floor(Math.random() * hobbysList[0].length)];
  }
};
 
console.log("Begin Parsing >>");
 
var template = fs.readFileSync('schema.hbs', {encoding: 'utf8'});
var result = dummyjson.parse(template, {helpers: helpers});
 
console.log("Begin Database Insert >>");
 
db.sourceData.remove(function (argument) {
    console.log("DB Cleanup Completd");
});
 
db.sourceData.insert(JSON.parse(result), function (err, docs) {
    console.log("DB Insert Completed");
});
```

**第1-4行**，我们引入了所有依赖。
**第2行**，我们创建了一个叫 `mapReduceDB` 的数据库。在数据库里面，创建了一个叫 `sourceData` 的集合。

**第6-23行**，是 Handlebar 的 helper。你可以到 `dummy-json` 中了解更多信息。

**第27-28行**，我们读取了 `schema.hbs` 文件 (我们接着会创建这个文件)，然后把它解析成 JSON。

**第32行**，在插入新数据之前，我们要先把旧数据清除掉。如果你想保留旧数据，把这部分注释掉就好了。

**第36行**，把生成的数据插入数据库。

接着，我们要在项目根目录创建一个叫 `schema.hbs` 的文件。这里面会包括 JSON 文档的结构。把下面的内容复制到文件里面：

```hbs
[
    {{#repeat 9999}}
    {
      "id": {{index}},
      "name": "{{firstName}} {{lastName}}",
      "email": "{{email}}",
      "work": "{{company}}",
      "dob" : "{{dob}}",
      "age": {{number 1 99}},
      "gender" : "{{gender}}",
      "salary" : {{number 999 99999}},
      "hobbies" : "{{hobbies}}"
    }
    {{/repeat}}
]
```

注意 **第2行**，我们会生成 9999 个文档。

打开一个新的终端，运行 `mongod`，启动 MongoDB 服务。然后回到原来的终端，运行 `node dataGen.js`。

如果一切正常，会显示如下结果：

```
$ node dataGen.js
Begin Parsing >>
Begin Database Insert >>
DB Cleanup Completed
DB Insert Completed
```

然后按 ctrl + c 杀掉 Node 程序。要验证是否插入成功，我们可以打开一个新的终端，运行 `mongo` 命令进入 mongo shell。

```
> use mapReduceDB
> db.sourceData.findOne()
{
    "id": 0,
    "name": "Leanne Flinn",
    "email": "leanne.flinn@unilogic.com",
    "work": "Unilogic",
    "dob": "Sun Mar 14 1909 12:45:53 GTM+0530 (LST)",
    "age": 27,
    "gender": "male",
    "salary": 16660,
    "hobbies": "Acrobatics,Photography,Papier-Mache",
    "_id": Object("57579f702fa6c7651e504fe2")
}
> db.sourceData.count()
9999
```

## 有意义的数据

现在我们有 9999 个虚假用户的数据，让我们试着把数据变得有意义

### 例子1：计算男女数量

首先，在项目根目录创建一个 `example1.js` 的文件，我们要进行 MapReduce 操作，去计算男女的数量。

#### Mapper 的逻辑

我们只需要让 Mapper 以性别作为 key，把值作为 1。因为一个用户不是男就是女。所以，Mapper 的输出会是下面这样：

| Key     | Value     |
|---------|-----------|
| Male    | [1,1,1...]     |
| Female  | [1,1,1,1,1...] |

#### Reducer 的逻辑

在 Reducer 中，我们会获得上面两行数据，我们要做的是把每一行中的值求和，表示该性别的总数。最终的输出结果如下：

| Key     | Value     |
|---------|-----------|
| Male    | 5031      |
| Female  | 4968      |

#### 代码

好了，现在我们可以写代码去实现了。在 `example1.js` 中，我们要先引入所需要的依赖。

```js
var mongojs = require('mongojs');
var db = mongojs('mapReduceDB', ['sourceData', 'example1_results']);
```

注意 **第2行**，第一个参数是数据库的名字，第二个参数表示集合的数组。`example1_results` 集合用来保存结果。

接下来，我们加上 mapper 和 reducer 函数：

```js
var mapper = function () {
    emit(this.gender, 1);
};
 
var reducer = function(gender, count){
    return Array.sum(count);
};
```

在**第2行**中， `this` 表示当前的文档，因此 `this.gender` 会作为 mapper 的 key，它的值要么是 `male`，要么是 `female`。而 `emit()` 将会把数据发送到一个临时保存数据的地方，作为 mapper 的结果。

在**第5行**中，我们简单地把每个性别的所有值加起来。

最后，加上执行逻辑：

```js
db.sourceData.mapReduce(
    mapper,
    reducer,
    {
        out : "example1_results"
    }
 );
 
 db.example1_results.find(function (err, docs) {
    if(err) console.log(err);
    console.log(docs);
 });
```

在**第5行**中，我们设置了输出的集合名。
在**第9行**中，我们会从 `example1_results` 集合取得结果并显示它。

我们可以在终端运行试试：

```
$ node example1.js
[ { _id: 'female', value: 4968 }, { _id: 'male': value: 5031 } ]
```

我的数量可能和你的不一样，但男女总数应该是 9999 !

#### Mongo Shell 代码

如果你想在 mongo shell 中运行上面的例子，你可以粘贴下面这些代码到终端里面：

```js
mapper = function () {
    emit(this.gender, 1);
};
 
reducer = function(gender, count){
    return Array.sum(count);
};
 
db.sourceData.mapReduce(
    mapper,
    reducer,
    {
        out : "example1_results"
    }
 );
 
 db.example1_results.find()
```

然后你就会看到一样的结果，很简单吧！

### 例子2：获取每个性别中最老和最年轻的人

在项目根目录创建一个 `example2.js` 的文件。在这里，我们要把所有用户根据性别分组，然后分别找每个性别中最老和最年轻的用户。这个例子比前面的稍微复杂一点。

#### Mapper 的逻辑

在 mapper 中，我们要以性别作为 key，然后以 object 作为 value。这个 object 要包含用户的年龄和名字。年龄是用来做计算用的，而名字只是用来显示给人看的。

| Key     | Value     |
|---------|-----------|
| Male    | [{age: 9, name: 'John'}, ...] |
| Female  | [{age: 19, name: 'Rita'}, ...] |

#### Reducer 的逻辑

我们的 reducer 会比前一个例子要复杂一点。我们要检查所有和性别相关的年龄，找到年龄最大和最小的用户。最终的输出结果是这样的：

| Key     | Value     |
|---------|-----------|
| Male    | {min: {name: 'harry', age: 1}, max: {name: 'Alex', age: 99} } |
| Female  | {min: {name: 'Loli', age: 10}, max: {name: 'Mary', age: 98} } |

#### 代码

现在打开 `example2.js`，粘贴下面的内容进去：

```js
var mongojs = require('mongojs');
var db = mongojs('mapReduceDB', ['sourceData', 'example2_results']);
 
 
var mapper = function () {
    var x = {age : this.age, name : this.name};
    emit(this.gender, {min : x , max : x});
};
 
 
var reducer = function(key, values){
    var res = values[0];
    for (var i = 1; i < values.length; i++) {
        if(values[i].min.age < res.min.age)
            res.min = {name : values[i].min.name, age : values[i].min.age};
        if (values[i].max.age > res.max.age) 
           res.max = {name : values[i].max.name, age : values[i].max.age};
    };
    return res;
};
 
 
db.sourceData.mapReduce(
    mapper,
    reducer,
    {
        out : "example2_results"
    }
 );
 
 db.example2_results.find(function (err, docs) {
    if(err) console.log(err);
    console.log(JSON.stringify(docs));
 });
```
在**第6行**，我们构建了一个 object，把它作为 value 发送。
在**第13-18行**，我们迭代了所有 object，检查当前的 object 的年龄是否大于或小于前一个 object 的年龄，如果是，就会更新 `res.max` 或者 `res.min`。
在第**第27行**，我们把结果输出到 `example2_results` 中。

我们可以运行一下这个例子：

```
$ node example2.js
[ { _id: 'female', value: { min: [Object], max: [Object] } },
  { _id: 'male', value: { min: [Object], max: [Object] } } ]
```


### 例子3：计算每种兴趣爱好的人数

在我们最后的例子中，我们会看看有多少用户有相同的兴趣爱好。我们在项目根目录创建一个叫 `example3.js` 的文件。用户数据长这样子：

```json
{
    "id": 0,
    "name": "Leanne Flinn",
    "email": "leanne.flinn@unilogic.com",
    "work": "Unilogic",
    "dob": "Sun Mar 14 1909 12:45:53 GTM+0530 (LST)",
    "age": 27,
    "gender": "male",
    "salary": 16660,
    "hobbies": "Acrobatics,Photography,Papier-Mache",
    "_id": Object("57579f702fa6c7651e504fe2")
}
```

如你所见，每个用户的兴趣爱好列表都用逗号分隔。我们会找出有多少用户有表演杂技的爱好等等。

#### Mapper 的逻辑

在这个场景下，我们的 mapper 会复杂一点。我们要为每个用户的兴趣爱好发送一个新的 key-value 对。这样，每个用户的每个兴趣爱好都会触发一次计算。最终我们会得到如下的结果：

| Key     | Value     |
|---------|-----------|
| Acrobatics   | [1,1,1,1,1,1,….] |
| Meditation   | [1,1,1,1,1,1,….] |
| Music        | [1,1,1,1,1,1,….] |
| Photography  | [1,1,1,1,1,1,….] |
| Papier-Mache | [1,1,1,1,1,1,….] |

#### Reducer 的逻辑

在这里，我们只要简单地为每种兴趣爱好求和就好了。最终我们会得到下面的结果：

| Key     | Value     |
|---------|-----------|
| Acrobatics   | 6641 |
| Meditation   | 3338 |
| Music        | 3338 |
| Photography  | 3303 |
| Papier-Mache | 6661 |

#### 代码

```js
var mongojs = require('mongojs');
var db = mongojs('mapReduceDB', ['sourceData', 'example3_results']);
 
 
var mapper = function () {
     var hobbys = this.hobbies.split(',');
      for (i in hobbys) {
        emit(hobbys[i], 1);
    }
};
 
var reducer = function (key, values) {
    var count = 0;
    for (index in values) {
        count += values[index];
    }
 
    return count;
};
 
 
db.sourceData.mapReduce(
    mapper,
    reducer,
    {
        out : "example3_results"
    }
 );
 
 db.example3_results.find(function (err, docs) {
    if(err) console.log(err);
    console.log(docs);
 });
```

注意**第7-9行**，我们迭代了每个兴趣爱好，然后发送了一次记数。
**第13-18行**可以用 `Array.sum(values)` 来代替，这样是另外一种做相同事情的方式。最终我们得到的结果：

```
$ node example3.js
[ { _id: 'Acrobatics', value: 6641 },
  { _id: 'Meditation', value: 3338 },
  { _id: 'Music', value: 3338 },
  { _id: 'Photography', value: 6661 },
  { _id: 'Papier-Mache', value: 3303 } ]
```

这就是 MongoDB 中运行 MapReduce 的方法了。但要记住，有时候一个简单的查询就能完成你想要的事情的。

## 参考资料
[MapReduce in MongoDB](http://thejackalofjavascript.com/mapreduce-in-mongodb/)
