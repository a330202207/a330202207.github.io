---
title: MongoDB的索引（四）
date: 2017-04-30 09:57:56
tags:
- MongoDB
categories:
- NoSQL
---

MongoDB 索引分类
----------------
1、_id 索引
`_id 索引`是绝大多数集合`默认建立`的`索引`，对于每个插入的数据，`MongoDB` 都会`自动生成`一条`唯一`的 `_id`字段

2、单键索引
1.	`单键索引`是最`普通的`索引
2.	与`_id索引`不同，`单键索引`不会自动创建 如：一条记录，形式为：`{ "stu_id" : 1, "stu_name" : "李1"}`
3.	可以重复创建，若创建已经存在的索引，则会直接返回成功


3、多键索引
多键索引与单键索引创建形式相同，区别在于字段的值：

 1. 单键索引：值为一个单一的值，如字符串，数字或日期
 2. 多键索引：值具有多个记录，如数组
 
4、复合索引
查询多个条件时,建立复合索引
例如`{stu_id:1,stu_name:-1}`这样一条数据，要按照`学生id`与`学生名字`的值进行查询，就需要创建复合索引。
```
# 1升序;-1降序
db.student.createIndex({stu_id:1,stu_name:-1}) 

# 使用复合索引查询
db.student.find({stu_id:1,stu_name:"李1"})
```

5、过期索引（TTL索引）
在`一段时间`后会`过期`的`索引`，在`索引过期`后，相应的`数据`会被`删除`，适合存储在`一段时间`之后会`失效`的`数据`，比如用户的登录信息、存储的日志等
```
# 创建过期索引，time-字段，expireAfterSeconds-在多少秒后过期，单位：秒
db.collection.createIndex({time:1},{expireAfterSeconds:10}) 

# time索引30秒后失效
db.student.createIndex({time:1},{expireAfterSeconds:15})

# new Date()自动获取当前时间，ISODate
db.student.insert({time:new Date()}) 
db.student find() 

```
过30秒后再查找，刚才的数据就已经不存在了
![][1]
过期索引的限制：

 1. 存储在过期索引字段的值`必须`是指定的`时间类型`，必须是 `ISODate` 或者 `ISODate` 数组，`不能`使用`时间戳`，否则不能`自动删除`
 2. 如果指定了 `ISODate` 数组，则按照`最小`的`时间`进行删除
 3. 过期索引`不能`是`复合索引`。因为`不能`指定`两个`过期时间
 4. 删除时间是不精确的。删除过程是由 `MongoDB` 的后台进程每60s跑一次的，而且删除也需要一定时间，所以存在误差
 
6、全文索引（文本索引）
```
# 每个集合只允许创建一个全文索引
db.articles.createIndex({key: "text"});

# 多个字段上创建全文索引
db.articles.createIndex({key1:"text",key2:"text"});

# 所有字段建立全文索引
db.articles.createIndex({"$**":"text"});

# 全文索引的查找
db.articles.find({$text:{$search:"coffee"}});

# 空格代表或操作，aa或bb或cc
db.articles.find({$text:{$search:"aa bb cc"}});

# -号为非操作，即不包含cc的
db.articles.find({$text:{$search:"aa bb -cc"}}); 

# 加双引号可以提供与关系操作
db.articles.find({$text:{$search: "\"aa\" \"bb\" \"cc\""}}); 

# 全文索引相似度查询，score 返回字段越高相关度越高
db.articles.find({$text:{$search:"aa bb"}},{score:{$meta:"textScore"}});

# 对`查询`出的`结果`根据`得分`进行`排序`
db.articles.find({$text:{$search:"aa bb"}},{score:{$meta:"textScore"}}).sort({score:{$meta:"textScore"}})
```
全文索引的限制：

 - 每次查询，只能指定一个`$text`查询
 - `$text`查询不能出现在`$nor`查询(非或)中
 - 查询中如果包含了 `$text`, `hint` 不再起作用（hint:强制指定索引）
 - `MongoDB` 全文索引还`不支持中文`

6、地理位置索引
将一些点的位置存储在 `MongoDB` 中，创建索引后，可以按照位置来查找其他点。

地理位置索引分为两类：

 - `2d 索引`，用于存储和查找`平面上的点`
 - `2dsphere 索引`，用于存储和查找`球面上的点`

1.2d索引:

`2d 索引`的`取值范围`以及表示方法:`经纬度[经度,纬度]`,经纬度取值范围 经度`[-180,180]`，纬度`[-90,90]`

```
# 创建 2d 索引
db.collection.createIndex({<location field>:”2d”});

# 查询距离某个点最近的点 ,默认返回最近的100个点
db.collection.find({w:{$near:[x,y]}});

# $maxDistance:x 限制返回的最远距离
db.collection.find({w:{$near:[x,y],$maxDistance:z}});

# 查询矩形中的点
db.collection.find({w:{$geoWithin:{$box:[[0,0],[3,3]]}}});

# 查询圆中的点
db.collection.find({w:{$geoWithin:{$center:[[0,0],5]}}});

# 查询多边形中的点
db.collection.find({w:{$geoWithin:{$polygon:[[0,0],[0,1],[2,5],[6,1]]}}});

# geoNear查询，minDistance对2D索引无效，对2Dsphere有效，num:返回数量
db.runCommand({geoNear:"collectionName",near:[x,y],minDistance:10,num:1});

```

2.2dsphere 索引:

`2dsphere` 索引位置表示方式是用 `GeoJSON` 对象来存储的，存储`GeoJSON`数据的话，在文档中使用 `type` 字段来指定 `GeoJSON` 对象类型以及 `coordinates` 对象来指定对象的坐标：

```
{ type: "<GeoJSON type>" , coordinates: <coordinates> }
```
`MongoDB` 默认使用 `WGS84` 基准作为 `GeoJSON `默认的坐标参考系统。

`GeoJSON` 对象类型：
```
# 点（Point）
{ type: "Point", coordinates: [ 40, 5 ] }

# 线（LineString），coordinate 成员必须是两个或多个坐标的数组
{ type: "LineString", coordinates: [ [ 40, 5 ], [ 41, 6 ] ] }

# 单环多边形（ Polygon ），第一个和最后一个坐标必须相同以关闭这个多边形 
{
  type: "Polygon",
  coordinates: [ [ [ 0 , 0 ] , [ 3 , 6 ] , [ 6 , 1 ] , [ 0 , 0  ] ] ]
}

# 多环多边形（ Polygon ）
# 第一个描述的环必须是外环，外环不能自交叉，所有内环必须全部包含在外环中，内环之间不能交叉或覆盖。内环不能共边
{
  type : "Polygon",
  coordinates : [
     [ [ 0 , 0 ] , [ 3 , 6 ] , [ 6 , 1 ] , [ 0 , 0 ] ],
     [ [ 2 , 2 ] , [ 3 , 3 ] , [ 4 , 2 ] , [ 2 , 2 ] ]
  ]
}

# 多点（ MultiPoint ）
{
  type: "MultiPoint",
  coordinates: [
     [ -73.9580, 40.8003 ],
     [ -73.9498, 40.7968 ],
     [ -73.9737, 40.7648 ],
     [ -73.9814, 40.7681 ]
  ]
}

# 多线（ MultiLineString ）
{
  type: "MultiLineString",
  coordinates: [
     [ [ -73.96943, 40.78519 ], [ -73.96082, 40.78095 ] ],
     [ [ -73.96415, 40.79229 ], [ -73.95544, 40.78854 ] ],
     [ [ -73.97162, 40.78205 ], [ -73.96374, 40.77715 ] ],
     [ [ -73.97880, 40.77247 ], [ -73.97036, 40.76811 ] ]
  ]
}

# 多个多边形（ MultiPolygon ）
{
  type: "MultiPolygon",
  coordinates: [
     [ [ [ -73.958, 40.8003 ], [ -73.9498, 40.7968 ], [ -73.9737, 40.7648 ], [ -73.9814, 40.7681 ], [ -73.958, 40.8003 ] ] ],
     [ [ [ -73.958, 40.8003 ], [ -73.9498, 40.7968 ], [ -73.9737, 40.7648 ], [ -73.958, 40.8003 ] ] ]
  ]
}

# 几何集合（GeometryCollection）
{
  type: "GeometryCollection",
  geometries: [
     {
       type: "MultiPoint",
       coordinates: [
          [ -73.9580, 40.8003 ],
          [ -73.9498, 40.7968 ],
          [ -73.9737, 40.7648 ],
          [ -73.9814, 40.7681 ]
       ]
     },
     {
       type: "MultiLineString",
       coordinates: [
          [ [ -73.96943, 40.78519 ], [ -73.96082, 40.78095 ] ],
          [ [ -73.96415, 40.79229 ], [ -73.95544, 40.78854 ] ],
          [ [ -73.97162, 40.78205 ], [ -73.96374, 40.77715 ] ],
          [ [ -73.97880, 40.77247 ], [ -73.97036, 40.76811 ] ]
       ]
     }
  ]
}
```
创建 `2dsphere` 索引：
```
# 创建2dsphere索引
db.collection.createIndex({<location field>:”2dsphere”});

# 插入一个坐标为[-73.97,40.77]的点
db.places.insert(
   {
      loc : { type: "Point", coordinates: [ -73.97, 40.77 ] },
      name: "Central Park",
      category : "Parks"
   }
)
```

2dsphere 查询类型：

 - `$near （GeoJSON点，2dsphere索引）`
```
# 查询坐标[-73.9667, 40.78],最近1000米，最远5000米的记录
db.places.find(
   {
     location:
       { $near :
          {
            $geometry: { type: "Point",  coordinates: [ -73.9667, 40.78 ] },
            $minDistance: 1000,
            $maxDistance: 5000
          }
       }
   }
)
```
 - `$nearSphere （GeoJSON点，2dsphere索引）`
```
# 查询坐标[-73.9667, 40.78]最少0.0004弧度距离的记录
db.legacyPlaces.find(
   { location : { $nearSphere : [ -73.9667, 40.78 ], $minDistance: 0.0004 } }
)
```
 - `$geoWithin:{$geometry:...}` 
 - `$geoWithin:{$centerSphere:...}`
```
# 查询坐标[-88, 30]，半径10/3963.2范围内的
db.places.find( {
  loc: { $geoWithin: { $centerSphere: [ [ -88, 30 ], 10/3963.2 ] } }
} )
```
 - `$geoIntersects`  
```
# 查询多边形内坐标[ 0, 0 ], [ 3, 6 ], [ 6, 1 ], [ 0, 0 ]相交的记录
db.places.find(
   {
     loc: {
       $geoIntersects: {
          $geometry: {
             type: "Polygon" ,
             coordinates: [
               [ [ 0, 0 ], [ 3, 6 ], [ 6, 1 ], [ 0, 0 ] ]
             ]
          }
       }
     }
   }
)
```
索引的操作
----------------
```
# 查询索引
db.collection.getIndexes();

# 创建索引 1:正序；-1:倒序
db.collection.createIndex({indexValue:1/-1);

# 指定索引别名
db.collection.createIndex({indexValue},{name:});

# 指定唯一索引
db.collection.createIndex({indexValue},{unique:true/false});

# 指定索引是否稀疏
#   例如，我们为一个collection的x字段指定了索引，
#   但这个collection中可以# 插入如{y:1,z:1}这种不存在x字段的数据，
#   如果索引为不稀疏的，MongoDB 依然会为这个数据创建索引，
#   如果在创建索引时指定为稀疏索引，那么就可以避免这件事情发生了
db.collection.createIndex({indexValue},{sparse:true/false});

# 删除某个索引
db.collection.dropIndex({indexValue:1});

# 删除集合下所有索引
db.collection.dropIndexes();
```


  [1]: http://olln3wpar.bkt.clouddn.com/image/MongoDB%284%29/1.png




  