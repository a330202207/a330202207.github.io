---
title: Redis集群搭建与简单使用（四）
date: 2017-03-08 06:43:42
tags:
- Redis
categories:
- NoSQL
---
工具
----------------
 - VMware版本号：12.0.0 
 - CentOS版本：7.0
 - 三台虚拟机(IP)：192.168.1.8、192.168.1.9、192.168.1.10    
![][1]

安装 Redis 
----------------
下载，解压，编译:
```
$ cd /usr/local/
$ wget http://download.redis.io/releases/redis-3.2.6.tar.gz
$ tar xzf redis-3.2.6.tar.gz
$ cd redis-3.2.6
$ make
```

创建 Redis 节点
----------------
首先在 `192.168.1.8` 机器上 `/usr/local/redis-3.2.6` 目录下创建 `redis_cluster` 目录
```
$ mkdir /usr/local/redis-3.2.6/redis_cluster　
```
在 `redis_cluster` 目录下，创建名为`7000`、`7001`、`7002`的目录，并将 `redis.conf` 拷贝到这三个目录中
```
$ mkdir 7000 7001 7002
$ cp /usr/local/redis-3.2.6/redis.conf /usr/local/redis-3.2.6/redis_cluster/7000
$ cp /usr/local/redis-3.2.6/redis.conf /usr/local/redis-3.2.6/redis_cluster/7001
$ cp /usr/local/redis-3.2.6/redis.conf /usr/local/redis-3.2.6/redis_cluster/7002
```
分别修改这`三个配置文件`，修改如下内容
```
port                  7000                        //端口7000,7002,7003        
bind                  本机ip                      //默认ip为127.0.0.1，需要改为其他节点机器可访问的ip，否则创建集群时无法访问对应的端口，无法创建集群
daemonize             yes                         //redis后台运行
pidfile               /var/run/redis_7000.pid     //pidfile文件对应7000，7001，7002
cluster-enabled       yes                         //开启集群，把注释#去掉
cluster-config-file   nodes_7000.conf             //集群的配置，配置文件首次启动自动生成 7000，7001，7002
cluster-node-timeout  15000                       //请求超时，默认15秒，可自行设置
appendonly            yes                         //aof日志开启，有需要就开启，它会每次写操作都记录一条日志　
```
接着在另外两台机器上(`192.168.1.9`、`192.168.1.10`)重复以上三步，只是把目录改为`7003、7004、7005、7006、7007、7008`对应的`配置文件`也按照这个`规则修改`即可

启动各个节点
----------------

```
##第一台机器上执行
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7000/redis.conf
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7001/redis.conf
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7002/redis.conf
 
##第二台机器上执行
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7003/redis.conf
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7004/redis.conf
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7005/redis.conf 

##第三台机器上执行
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7006/redis.conf
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7007/redis.conf
$ /usr/local/redis-3.2.6/src/redis-server /usr/local/redis-3.2.6/redis_cluster/7008/redis.conf 
```

检查各 Redis 启动情况
----------------

```
##第一台机器
$ ps -ef | grep redis           //redis是否启动成功
$ netstat -tnlp | grep redis    //监听redis端口
```
![][2]
`注`：确保每个节点没有配置错误，并且启动起来


关闭防火墙
----------------

```
$ firewall-cmd --state  ##查看防火墙状态
running
```
`running` 说明防火墙是打开状态
```
$ systemctl stop firewalld  ##关闭防火墙
$ firewall-cmd --state
not running
```
`注`: `CentOS 7` 关闭防火墙与 `CentOS 6` 有所不同

安装 Ruby
----------------

```
$ yum -y install ruby ruby-devel rubygems rpm-build
$ gem install redis
```
`注`:`创建集群`时需要安装 `Ruby` 运行`redis-trib.rb`

创建集群
----------------
`Redis` 官方提供了 `redis-trib.rb` 这个工具，就在解压目录的 `src` 目录中
```
$ /usr/local/redis-3.2.6/src/redis-trib.rb  create  --replicas  1  192.168.1.8:7000 192.168.1.8:7001  192.168.1.8:7002 192.168.1.9:7006  192.168.1.9:7004  192.168.1.9:7005 192.168.1.10:7006 192.168.1.10:7007 192.168.1.10:7008
```
其中，前三个 `ip:port` 为第一台机器的节点，中间三个为第二台机器，最后三个为第三台机器
![][3]
 输入 `yes`，然后出现如下内容，说明`安装成功`
![][4]

集群验证
----------------
在第一台机器上连接集群的`7000节点`，在另外一台连接`7004节点`，连接方式为：
```
##加参数 -C 可连接到集群，因为 redis.conf 将 bind 改为了ip地址，所以 -h 参数不可以省略，-p 参数为端口号
$ /usr/local/redis-3.2.6/src/redis-cli -h 192.168.1.8 -c -p 7000  
```
在`7004节点`执行命令：
```
192.168.1.9:7004> set name redis
```
然后在另两台`7000、7007端口`，查看 `key` 为 `name` 的内容
![][5]

```
192.168.1.8:7000> get name
```
![][6]

```
192.168.1.10:7007> get name
```
![][7]

说明`集群运作正常`

总结
----------------
`redis cluster`在设计的时候，就考虑到了去`中心化`、`去中间件`，也就是说，集群中的`每个节点`都是`平等关系`，都是`对等的`，`每个节点`都`保存`各自的`数据`和整个集群的状态。`每个节点`都和`其他所有节点`连接，而且这些连接`保持活跃`，这样就保证了我们`只需要连接`集群中的`任意一个节点`，就可以获取到`其他节点`的`数据`。

`Redis` 集群没有并使用传统的`一致性哈希来`分配数据，而是采用另外一种叫做`哈希槽 `(hash slot)的方式来分配的。`redis cluster` 默认分配了 `16384` 个 `slot`，当我们 `set `一个 `key` 时，会用`CRC16算法`来取模得到所属的 `slot`，然后将这个 `key` 分到`哈希槽区间的节点`上，具体算法就是：`CRC16(key)` % `16384`。所以我们在测试的时候看到 `set` 和 `get` 的时候，直接跳转到了`7000端口`的节点。

`Redis` 集群会把数据存在一个 `master` 节点，然后在这个 `master` 和其对应的 `salve` 之间进行数据同步。当读取数据时，也根据`一致性哈希算法`到对应的 `master` 节点获取数据。只有当一个 `master` 挂掉之后，才会启动一个对应的 `salve` 节点，充当 `master` 。

需要注意的是：必须要`3个`或`以上`的`主节点`，否则在`创建集群`时会`失败`，并且当`存活`的`主节点数`小于`总节点数`的`一半`时，整个`集群`就`无法提供服务`了。


----------


**相关文档：**
中文：http://www.redis.cn/topics/cluster-tutorial.html
英文：https://redis.io/topics/cluster-tutorial

----------
**相关链接：**
[Linux下PHP安装Redis扩展（二）][8]
[Redis主从配置（三）][9]
[Redis持久化（五）][10]


  [1]: https://ned.oss-cn-beijing.aliyuncs.com/2017-03-07-21-34-1.png
  [2]: https://ned.oss-cn-beijing.aliyuncs.com/2017-03-07-21-34-2.png
  [3]: https://ned.oss-cn-beijing.aliyuncs.com/2017-03-07-21-34-3.png
  [4]: https://ned.oss-cn-beijing.aliyuncs.com/2017-03-07-21-34-4.png
  [5]: https://ned.oss-cn-beijing.aliyuncs.com/2017-03-07-21-30-5.png
  [6]: https://ned.oss-cn-beijing.aliyuncs.com/2017-03-07-21-30-7.png
  [7]: https://ned.oss-cn-beijing.aliyuncs.com/2017-03-07-21-30-8.png
  [8]: https://segmentfault.com/a/1190000008420258
  [9]: https://segmentfault.com/a/1190000008469182
  [10]: https://segmentfault.com/a/1190000008639459