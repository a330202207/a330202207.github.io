---
title: MongoDB 简单介绍以及安装(一)
date: 2017-03-30 23:08:15
tags:
- MongoDB
categories:
- NoSQL
---

MongoDB 介绍
----------------
`MongoDB` 是一种 `NoSQL` 数据库，它在`数据存储`的形态上和 `MySQL` 这类`关系数据库`有本质区别。`MongoDB` 存储的基本对象是 `Document`，所以我们把它称为一种`文档数据库`，而文档的`集合`则组成了 `Collection`。与 `SQL` 的概念类比，`Collection` 对应于 `Table` 而 `Document` 对应于 `Row`。`Document` 使用一种 `BSON（Binary JSON`）结构来表达，`JSON` 大家都熟悉，像下面这样。
![此处输入图片的描述][1]

MongoDB 应用场景
----------------

 - 游戏场景，使用 MongoDB 存储游戏用户信息，用户的装备、积分等直接以内嵌文档的形式存储，方便查询、更新
 - 物流场景，使用 MongoDB 存储订单信息，订单状态在运送过程中会不断更新，以 MongoDB 内嵌数组的形式来存储，一次查询就能将订单所有的变更读取出来
 - 社交场景，使用 MongoDB 存储存储用户信息，以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能
 - 物联网场景，使用 MongoDB 存储所有接入的智能设备信息，以及设备汇报的日志信息，并对这些信息进行多维度的分析
 - 视频直播，使用 MongoDB 存储用户信息、礼物信息等

MongoDB 安装
----------------
`MongoDB` 的安装方式比较简单，由于`源码安装`比较麻烦，我们的本意只是为了学习 `MongoDB` 而 `yum` 种傻瓜式安装是为了更方便现在学习，本文以 yum 方式安装。

工具：

 - VMware版本号：12.0.0 

 - CentOS版本：7.0

`注`： 3.4 版本 `MongoDB` 不再为 `32` 位平台（Linux 和 Windows）提供商业支持，本文安装版本为3.4
 

查看自己 `Linux` 版本：

```
uname –a
```
`x86_64` 表示 `64` 位机器
`i686` 表示 `32` 位机器

 - 整个 `MongoDB`（社区版）包含如下软件
```

# 包含mongod守护程序和关联的配置和init脚本
mongodb-org-server	

# 包含mongos守护程序
mongodb-org-mongos	

# 包含mongo shell，它是一个连接mongodb的命令行客户端，允许用户直接输入nosql语法管理数据库
mongodb-org-shell	

# 包含以下工具的MongoDB：数据导入、导出、备份、恢复等等
mongodb-org-tools	
```


 - 创建 yum 源文件

```
vim /etc/yum.repos.d/mongodb-org-3.4.repo
```

 - 将下面内容复制到源文件中

```
[mongodb-org-3.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```

![此处输入图片的描述][2]
 
 - 启动 yum 命令开始安装

```
yum install -y mongodb-org
```

 - 如果使用 `SELinux`，则必须配置 `SELinux`，以允许在基于 `Red Hat Linux` 的系统（Red Hat Enterprise Linux 或 CentOS Linux）上启动 `MongoDB`

```
vim /etc/selinux/config
```

将 `SELINUX` 值设置为 `disabled`
![此处输入图片的描述][3]

 - 启动 Mongodb (Mongodb 服务端的默认连接端口是 `27017`)

```
# Centos6 启动
$ service mongod start

# Centos7 启动
$ systemctl start mongod
```

 - 查看是否启动
```
netstat -tlnup|grep mongod
```
 - 查看是否占用 `27017` 端口号

```
netstat -tlnup|grep 27017
```

 - 其它控制命令

```
# 停止 Mongodb 服务
$ service mongod stop

# 重启 Mongodb
$ service mongod restart

```

 - 设置开机启动

```
chkconfig mongod on
```

 - 找到 MongoDB 客户端

```
find / -name mongo
```
![此处输入图片的描述][4]

 - 连接客户端

```
/usr/bin/mongo
```
输入测试命令 `show dbs` 查看当前数据库有哪些
![此处输入图片的描述][5]

 - 停止 MongoDB 服务器
可以使用 `Ctrl + c` 或者输入 `exit` 退出 `MongoDB` 界面。

`注`：进入 `MongoDB` 界面会出现`警告`
```
Server has startup warnings: 
2017-03-30T06:40:26.039+0800 I CONTROL  [initandlisten] 
2017-03-30T06:40:26.039+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2017-03-30T06:40:26.039+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2017-03-30T06:40:26.039+0800 I CONTROL  [initandlisten] 
2017-03-30T06:40:26.040+0800 I CONTROL  [initandlisten] 
2017-03-30T06:40:26.040+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2017-03-30T06:40:26.040+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2017-03-30T06:40:26.040+0800 I CONTROL  [initandlisten] 
2017-03-30T06:40:26.040+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2017-03-30T06:40:26.040+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2017-03-30T06:40:26.040+0800 I CONTROL  [initandlisten] 
```
这是因为没有配置 `MongoDB` 的`安全功能`，如`授权`和`身份验证`。当然只是为了学习的话，可以忽略它，但是生产环境`必须需要配置`。
  [1]: http://olln3wpar.bkt.clouddn.com/MongoDB_JOSN.png
  [2]: http://olln3wpar.bkt.clouddn.com/MongoDB_1.png
  [3]: http://olln3wpar.bkt.clouddn.com/MongoDB_2.png
  [4]: http://olln3wpar.bkt.clouddn.com/MongoDB_3.png
  [5]: http://olln3wpar.bkt.clouddn.com/MongoDB_4.png

 