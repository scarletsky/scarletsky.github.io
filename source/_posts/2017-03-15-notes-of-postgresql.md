---
title: PostgreSQL 学习笔记
date: 2017-03-15 21:57:28
category: sql
tags: [sql, postgresql]
---

## 简介

PostgreSQL 是世界上功能最强大的关系型数据库，除了标准的 SQL 的支持外，还有丰富的特性，可以通过扩展来增强数据库的能力。
本文是我学习 PostgreSQL 过程中的一些总结。

## 安装

### macOS

```
brew update
brew install postgresql
```

## 常用命令

- `\?` 显示帮助信息
- `\l` 列出所有数据库，相当于 `show databases`
- `\c <database_name>` 连接指定的数据库，相当于 `use <database_name>`
- `\dt` 列出数据库中所有的表，相当于 `show tables`
- `\ds` 列出数据库中所有序列
- `\d <table_name>` 显示该表的详细信息
- `\password <username>` 修改指定用户的密码
- `\dx` 列出数据库已使用的扩展
- `\x` 开启/关闭展开显示，当 `select` 的输出结果太长时，可以试试这个模式


## 扩展

我们知道，PostgreSQL 有非常丰富的扩展供我们使用，下面列举一些常见的扩展：

### uuid-ossp

该模块提供了一些函数来生成 UUID。如 `uuid_generate_v1`, `uuid_generate_v4` 等。

```sql
CREATE EXTENSION "uuid-ossp";
SELECT uuid_generate_v4();
           uuid_generate_v4
--------------------------------------
 cc230056-ed05-4bcf-a12f-1611728ac199
```


从[官网](http://www.postgres.cn/docs/9.4/uuid-ossp.html)上了解到，如果我们只需要生成 UUID (v4)，那么可以考虑使用 `pgcrypto` 模块。

### pgcrypto

该模块提供了一些加密的函数，其中 `gen_random_uuid` 这个函数可以用来生成 UUID。

```sql
CREATE EXTENSION "pgcrypto";
SELECT gen_random_uuid();
           gen_random_uuid
--------------------------------------
 f8ea5410-cac7-4811-bfbd-5e0b8d0a0cc4
```


### hstore

该模块为 PostgreSQL 提供存储 Key/Value 数据的功能，你可以定义 hstore 数据列来存储这样的数据，并可对这些数据进行分组、排序和唯一检索的查询。

```sql
CREATE EXTENSION "hstore";
SELECT 'a=>1, b=>2'::hstore;
       hstore
--------------------
 "a"=>"1", "b"=>"2"
```

对于 hstore 的增删查改操作，我们可以用如下的操作：

- `hstore -> text` 获取 hstore 中指定的 key。
- `hstore || hstore` 合并两个 hstore，常用于更新操作。
- `delete(hstore, text)` 删除 hstore 中指定的 key。

```sql
CREATE TABLE testhstore (id SERIAL, value hstore);
INSERT INTO testhstore (value) VALUES ('name=>smallfish, age=>29'::hstore);
INSERT INTO testhstore (value) VALUES ('name=>nnfish, age=>20'::hstore);
INSERT INTO testhstore (value) VALUES ('name=>aaa, age=>30, addr=>China'::hstore);

-- 查询 key
SELECT id, value->'name' AS name FROM testhstore;
 id |   name
----+-----------
  1 | smallfish
  2 | nnfish
  3 | aaa
(3 rows)
-- 更新 key
UPDATE testhstore SET value = value || ('addr=>Shanghai') WHERE id = 2;
-- 删除 key
UPDATE testhstore SET value = delete(value, 'addr') WHERE id = 3;
```

### PostGIS
PostGIS 在对象关系型数据库PostgreSQL上增加了存储管理空间数据的能力，相当于Oracle的spatial部分。PostGIS最大的特点是符合并且实现了OpenGIS的一些规范，是最著名的开源GIS数据库。

### PostPic
PostPic 用来在数据库内进行图像处理，PostPic 为 SQL 增加了 image 类型，还包含很多相关的函数用来处理图片以及从图片中抽取对应的属性。

## JSON / JSONB

JSON 类型是在 PostgreSQL 9.2 版本中加入的，而 JSONB 则是在 9.4 版本中加入的。
两者的区别在于：JSON 是纯文本，而 JSONB 则是二进制。
由于 JSON 存储的是未经处理过的 JSON 字符串，不需要进行解析，而 JSONB 插入时需要进行解析，因此在插入速度上 JSON 会比 JSONB 快，但在查询时，JSONB 会比 JSON 快。

因此，如果我们的应用需要频繁操作 JSON 的话，用 JSONB 会比较合适；相反，如果只是需要存储的话，用 JSON 会比较合适。

下面是一些常用的 JSON / JSONB 的操作：

- `json -> int` 根据指定索引获取 JSON 数组中的元素
- `json -> text` 获取 JSON 中指定字段的值
- `json #> array` 获取 JSON 中指定路径的值

需要注意的是，上面三种操作都是返回 `JSON(B)` 类型的结果，有些情况下我们需要把结果转换成字符串之后才能进行比较。
这时候就需要在上面几个操作符改成 `->>`, `#>>`，这样返回的结果就是 `text` 类型，可以直接进行比较了。

```sql
SELECT
  pg_typeof('{ "name": "john", "age": 30, "addr": "shanghai" }'::json -> 'age'),
  pg_typeof('{ "name": "john", "age": 30, "addr": "shanghai" }'::json ->> 'age');

 pg_typeof | pg_typeof
-----------+-----------
 json      | text
```

看看下面的例子：

```sql
CREATE TABLE testjson (id SERIAL, value json);
INSERT INTO testjson (value) values ('{ "name": "john", "age": 30, "addr": "shanghai" }');
INSERT INTO testjson (value) values ('{ "name": "lucy", "age": 20, "addr": "guangzhou" }');
INSERT INTO testjson (value) values ('{ "name": "tim", "age": 24, "addr": "beijing", "books": [1,2,3,4,5] }');

-- 根据指定 key 查询
SELECT * FROM testjson WHERE value ->> 'age' > '20';

 id |                                 value
----+-----------------------------------------------------------------------
  1 | { "name": "john", "age": 30, "addr": "shanghai" }
  3 | { "name": "tim", "age": 24, "addr": "beijing", "books": [1,2,3,4,5] }

-- 根据指定路径查询
SELECT * FROM testjson WHERE value #>> '{books, 3}' = '4';
 id |                                 value
----+-----------------------------------------------------------------------
  3 | { "name": "tim", "age": 24, "addr": "beijing", "books": [1,2,3,4,5] }
```

由于 JSON 是纯文本，因此对其进行更新操作只是相当于操作字符串，如：

```sql
UPDATE testjson SET value = replace(value::text, '"john"', '"mark"')::json WHERE id = 1;
```

这种方式相当不友好，因此，如果需要经常操作 JSON 的话，建议使用 JSONB 类型，它提供了多种操作方式。

```sql
-- for JSONB
-- 根据指定字段更新
UPDATE testjsonb SET value = jsonb_set(value, '{name}', 'mark') WHERE id = 1;
-- 根据指定路径更新
UPDATE testjsonb SET value = jsonb_set(value, '{books, 3}', '10') WHERE id = 3;
-- 移除 age 字段
UPDATE testjsonb SET value = value - 'age' WHERE id = 2;
```


## 升级

### macOS

- 停止 PostgreSQL 服务器
    ```
    brew services stop postgresql
    ```
- 升级 PostgreSQL
    ```
    brew update && brew upgrade postgresql
    ```
- 为新版 PostgreSQL 创建数据库
    ```
    initdb /usr/local/var/postgres9.6.2 -E utf8
    ```
- 迁移数据
    ```
    pg_upgrade \
      -d /usr/local/var/postgres \
      -D /usr/local/var/postgres9.6.2 \
      -b /usr/local/Cellar/postgresql/9.5/bin/ \
      -B /usr/local/Cellar/postgresql/9.6.2/bin/ \
      -v
    ```
- 把新数据库移动到 PostgreSQL 默认的位置
    ```
    mv /usr/local/var/postgres /usr/local/var/postgres9.5
    mv /usr/local/var/postgres9.6.2 /usr/local/var/postgres
    ```
- 重启 PostgreSQL 服务器
    ```
    brew services start postgresql
    ```

## 常见坑

### 单引号与双引号
单引号 (`'`) 和双引号 (`"`) 在 PostgreSQL 中有明确的区分，单引号用来引用值 (value)，而双引号用来引用标识符，如字段名，表名等等。
因此，下面两段 SQL 会有不同的结果：

```sql
-- 单引号
SELECT id FROM posts WHERE id = 'ad3eef9a-e081-46a9-9304-4a1d1637e542';
                  id
--------------------------------------
 ad3eef9a-e081-46a9-9304-4a1d1637e542
(1 row)

-- 双引号
SELECT id FROM posts WHERE id = "ad3eef9a-e081-46a9-9304-4a1d1637e542";
ERROR:  column "ad3eef9a-e081-46a9-9304-4a1d1637e542" does not exist
LINE 1: select id from posts where id = "ad3eef9a-e081-46a9-9304-4a1...
```



## 参考资料
http://www.postgres.cn/docs/9.4/index.html
http://postgresguide.com/
http://www.ruanyifeng.com/blog/2013/12/getting_started_with_postgresql.html
https://hashrocket.com/blog/posts/faster-json-generation-with-postgresql
https://www.postgresql.org/docs/9.4/static/uuid-ossp.html
https://www.postgresql.org/docs/9.4/static/pgcrypto.html
http://www.postgresqltutorial.com/postgresql-hstore/
https://www.postgresql.org/docs/9.5/static/sql-do.html
https://keita.blog/2016/01/09/homebrew-and-postgresql-9-5/
http://www.oschina.net/news/28211/postgresql-most-useful-extensions
http://chenxiaoyu.org/2011/02/19/postgresql-key-value-hstore.html
https://segmentfault.com/a/1190000002911580
https://www.citusdata.com/blog/2016/07/14/choosing-nosql-hstore-json-jsonb/
http://blog.lerner.co.il/quoting-postgresql/
https://wiki.postgresql.org/wiki/Things_to_find_out_about_when_moving_from_MySQL_to_PostgreSQL
http://stackoverflow.com/questions/38777535/what-is-the-difference-between-and-in-postgres-sql
