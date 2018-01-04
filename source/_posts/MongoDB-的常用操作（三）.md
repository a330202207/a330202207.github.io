---
title: MongoDB 的常用操作（三）
date: 2017-04-29 06:50:53
tags:
- MongoDB
categories:
- NoSQL
---
MongoDB 集合操作
----------------
1.显示创建集合
```
db.createCollection();
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/1.png)

2.查看集合
```
show collections
show tables
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/2.png)

3.隐式创建
```
db.class.insert({‘classId’:1,’className’:”三年二班”});
```
表示往 `class` 集合中插入一个`文档`(班级编号为1，班级名称为三年二班)。
本质是，往一个集合中插入一条数据，如果这个集合不存在，`MongoDB` 会自动帮我们创建该集合
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/3.png)

4.删除集合
```
db.collectionName.drop();
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/4.png)

MongoDB 文档操作
----------------
1.文档插入
`MongoDB` 中数据插入，指的是将创建的`文档`，`插入`到指定的`集合`中
```
db.collectionName.insert(doc);
```
如果`集合不存在`，则`创建`该集合。`doc` 可以是一个`集合`或者一个`集合数组`。
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/5.png)
`_id` 这是 `MongoDB` 自动生成的一个`全球唯一`的`主键`，用于`区分文档`。其包含四部分：`时间戳`、`机器`、`PID(进程号)`、`计数器`

插入多个文档:
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/6.png)

2.文档删除
```
db.collectionName.remove(where,[justOne]);
```
`where` 代表`删除条件`，当 `where` 为`{}`代表`删除所有`文档。
`justOne` 默认情况下为 `false`，代表`匹配`条件的`文档`有多个的时候，`删除所有`的`匹配文档`。如果设置为 `true` 或`1`（设置其他非0的整数也可以），代表仅仅`删除一个`文档。

删除 `school` 数据库 `student` 集合里名字(stu_name)为王五的文档:

```
db.student.remove({"stu_name":"王五"});
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/7.png)

删除 `school` 数据库 `student` 集合里班级编号(class_id)为1的一个集合:
```
db.student.remove({"class_id":1},2);
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/8.png)

虽然`满足`条件的`文档`很多，但是`仅仅删除`了其中`一个文档`。

3.文档修改
```
db.collectionName.update(query,update,upset,multi);
```
`collectionName`：集合名字

`query`：修改的条件，类似 SQL 的 where 语句。
`update`：更新命令，类似 SQL 的 set 语句。
`upset`：可选，默认为 false，如果未发现符合更新条件的数据内容，是否执行插入操作，1或者 true 代表进行插入,0或者 false 代表不插入。
`multi`：可选，MongoDB 默认为 false。是否进行多行更新。1或者 true 进行·多行更新,0或者false代表不更新`。

关于 `update` 格式有`两种`形式

- 直接`赋值`使用 `{set:{}}`
如 `{set:{“name”:”张三”}}`，将名字改为张三
- 进行`算术运算`使用 `{inc:{}}`
如 `{inc:{“age”:3}}`，将 `age` 增加3
如 `{$inc:{“age”:-3}}`，将 `age` 减3

**实例用法：**
1、更新文档中不存在字段
```
# 更新学生编号(stu_id)为2文档，为其添加一个字段年龄(age)为13
db.student.update({"stu_id":2},{$set:{"age":13}});
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/9.png)

2、批量更新文档中不存在字段
```
# 更新学生，将其年龄(age不存在)设置为12
db.student.update({},{$set:{"age":12}},0,1);
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/10.png)

3、批量指定字段更新
```
# 将班级编号为2的所有学生，年龄(age)增加1
db.student.update({"class_id":1},{$inc:{"age":1}},0,1);
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/11.png)

4.文档查询

- 查询集合中的所有文档
```
db.collectionName.find();
```
为了便于查找，先向其中插入一些数据
```
for (var i = 1;i < 20;i ++){
db.student.insert({"stu_id":i,"stu_name":"李" + i});
}
db.student.find();
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/12.png)

- 查询集合中的第一个文档
```
db.collectionName.findOne();
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/13.png)

**条件查询：**

| 项目        | 价格   |  数量  |
| --------   | -----:  | :----:  |
| 等于(=)      | / |   查找名字为李四的所有学生：db.student.find({“name”:”李四”});     |
| 等于(!=)      | \$ne |   查找名字不为李四的所有学生：db.student.find({“name”:{'$ne:"李四"}'}});     |
| 大于(>)      | \$gt |   查找所有年龄大于5的学生：db.student.find({"age":{$gt:5}});     |
| 小于(<)      | \$lt |   查找所有年龄小于15的学生：db.student.find({"age":{$lt:15}});     |
| 大于等于(>=)        |   \$gte   |   查找所有年龄大于或等于5的学生：db.student.find({"age":{$gte:5}});   |
| 小于等于(>=)        |    \ &#124; $lte    |  查找所有年龄小于或等于15的学生：db.student.find({"age":{$lte:15}});  |
| 与(and)        |    /    |  查找所有年龄小于或等于15且班级编号为1的所有学生：db.student.find({"age":{$lte:15},"class_id":1});  |
| 或(or)        |    {$or:[{条件1},{条件2}]}    |  查找所有年龄小于或等于15或者班级为1的所有学生：db.student.find({$or:[{"age":{$lte:15}},{"class_id":1}]});  |
| 非或(nor)        |    {$nor:[{条件1},{条件2}]}    |  条件1不能满足和条件2不能满足  |
| 集合运算符(in)        |    \$in	    |  查找年龄12或者13或者14的学生：db.student.find({"age":{$in:[12,13,14]}});  |
| 集合运算符(all)        |    \$all    |  查找爱好包含football,和basketBall的所有学生：db.student.find({"hobby":{$all:["football","basketBall"]}});  |
| 是否存在(exists)        |    $exists    |  查询存在hooby的所有学生：db.student.find({hobby:{$exists:1}});{$exists:0}代表不存在  |


5.统计、排序、分页

- 统计
```
db.collectionName.count();
db.collectionName.find().count();
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/14.png)

- 排序
```
# key升序排列
db.collectionName.find().sort({key:1});
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/15.png)

```
# key降序排列
db.collectionName.find().sort({key:-1});
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/16.png)

- 分页
```
db.collectionName.find().skip(n).limit(m);
```
从 `collectionName` 中第 `n` 个文档开始读取，共读取 `m` 个文档，需要注意的是 `MongoDB` 中 `n` 是从 0 开始的。可以类比 `MySQL` 中 `limit` 方法。`n` 相当于 `limit` 中的第一个方法，`m` 相当于 `limit` 中第二个参数,当使用分页时候，若使用 `count`，需要注意 `MongoDB` 统计数量默认是忽略分页的
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/17.png)
给 `count` 传递一个参数 `count(flag)`。`flag`为 `0`（默认情况为0）`忽视分页`，`flag` 为`1`，`不忽视分页`。
从 `student` 中取出6~12条数据
```
db.student.find().skip(6).limit(6);
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/18.png)
从集合 student 中取出第一条数据
```
db.student.find().skip(0).limit(1);
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/19.png)

6.投影查询
在 MongoDB 中，每次查询都是将所有的键都显示出来了，怎样只让部分键显示呢？这就要用到投影查询。
比如，我只让集合 student 查询结果中 stu_name 显示。
```
db.student.find({},{_id:0,stu_id:0});
```
![](http://olln3wpar.bkt.clouddn.com/image/MongoDB%283%29/20.png)
在 `MongoDB` 中，`find` 函数里面的`第二个参数`是用来`控制`让哪些键`显示`或者`不显示`。`0`是`不显示`，`1`为`显示`。`默认`情况下为`显示`。
















