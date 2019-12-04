---
title: MongoDB 的基本命令（二）
date: 2017-04-28 23:08:15
tags:
- MongoDB
categories:
- NoSQL
---
MongoDB 的基本命令
----------------

1.db.help()
查看命令提示
![][1]

2.db
查看当前所在数据库的名字
![][2]

3.use
`use` + `数据库名`，表示`切换`或是`创建`数据库。和 `MySQL` 中 `use` 的唯一不同点，就是当数据库`不存在`的时候，`MongoDB` 中的 `use` 可以`创建`数据库
![][3]

4.show dbs
显示数据库，需要注意的是 `show dbs`，只会显示有数据的库，没有数据的库，是不会显示的。下图中，`test` 数据库中，开始没有数据，所以不会显示，当我插入一条数据库后，`show dbs` 就会显示它
![][4]

5.db.dropDatabase()
`删除`当前数据
![][5]

6.db.stats()
显示当前 `db` 状态
![][6]

7.db.version()
显示当前 `db` 版本
![][7]

8.db.getMongo()
查看当前 `db` 的链接机器地址
![][8]

9.MongoDB 对支持 JavaScript 
因为 `MongoDB` 支持 `JavaScript`，所以可以直接在命令提示中输入 `js` 代码
![][9]

10.清空屏幕
`cls` 命令

MongoDB 的数据结构类型
----------------
一个`数据库`(Database)中可以包含多个`集合`(colection)，每个`集合`中，可以包含多个`文档`。可以类比，SQL 数据库，一个`数据库`中包含多个`表`，每个`表`中包含多个`记录`
| SQL术语/概念       | MongoDB术语/概念 |
| :--------          | :-----           | 
|Database(数据库)    | Database(数据库) |
|Table(数据库表)     | Collection(集合) |
|Row(数据记录行)     | Document(文档)   |
|Column(数据字段)    | Field(域)        |
|Index(索引)         | Index(索引)      |
|Table  joins(表连接)| 不支持           |
|primary key(主键)   |primary key(MongoDB自动将_id字段设置为主键)|
![][10]
![][11]

`BSON` 类型：

 - null
 `null` 用于表示`空值`或者`不存在`的字段。`{“tmpkey”:null}`
 - 布尔
 `布尔类型`有两个值 `true` 和 `false`。`{“tmpkey”:true}`
 - 32位整数
 类型不可用。`JavaScript` 仅支持`64位`浮点数，所以`32位`整数会被`自动转换`
 - 64位整数
 不支持这个类型。`shell` 会使用一个特殊的内嵌文档来显示`64位`整数
 - 64位浮点数
 `shell` 中的数字都是这种类型。下面的表示都是浮点数:`{“tmpkey”:5.12}`
 - 字符串
 `UTF-8` 字符串都可表示为字符串类型的数据:`{“tmpkey”:“abf”}`
 - 符号
 不支持这种类型。`shell` 将数据库里的`符号类型`转换成`字符串`
 - 对象 id
 `对象 id `是文档的12字节的唯一 ID,`{“tmpkey”:ObjectId()}`
 - 日期
 `日期类型存储`的是从`标准纪元`开始的毫秒数。不存储时区:`{“tmpkey”:new Date()}`
 - 正则表达式
 文档中可以包含`正则表达式`，采用 `JavaScript` 的正则表达式语法:`{“tmpkey”:/\w/i}`
 - 代码
 文档中还可以包含 `JavaScript` 代码：`{“tmpkey”:function() { /* …… */ }} `
 - 二进制数据
 `二进制数据`可以由任意字节的串组成。不过 `shell` 中无法使用
 - 最大值
 `BSON` 包括一个`特殊类型`，表示可能的`最大值`。`shell` 中没有这个类型 
 - 最小值
 `BSON` 包括一个`特殊类型`，表示可能的`最小值`。`shell` 中没有这个类型
 - 未定义
 文档中也可以使用`未定义类型`:`{“tmpkey”:undefined}`
 - 数组
 值的`集合`或者`列表`可以表示成`数组`:`{“tmpkey”:[“a”, “b”, “c”]}`
 - 内嵌文档
 文档可以包含别的文档，也可以作为值`嵌入`到`父文档`中，数据可以组织得更自然些，不用非得存成扁平结构的:`{“tmpkey”:{“color”:“yello”}}`


**相关文档：https://docs.mongodb.com/manual/reference/bson-types/**

  [1]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/1.png
  [2]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/2.png
  [3]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/3.png
  [4]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/4.png
  [5]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/5.png
  [6]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/6.png
  [7]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/7.png
  [8]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/8.png
  [9]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/9.png
  [10]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/10.png
  [11]: https://ned.oss-cn-beijing.aliyuncs.com/image/MongoDB%282%29/11.png

  