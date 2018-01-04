---
title: Linux(CentOS)下安装Redis（一）
date: 2017-02-13 06:19:19
tags:
- Redis
- yum
categories:
- NoSQL
---
整理一下最近学习Redis的心得
----------------

 - VMware版本号：11.1.2 
 - CentOS版本：6.6
 


----------


下载redis
-------

    wget http://download.redis.io/releases/redis-3.0.0.tar.gz
    


----------

编译源程序
-----

    tar zxvf redis-3.0.0.tar.gz 
    cd redis-3.0.0 
    make
    cd src
    make install
   


----------


创建redis目录，移动文件，为了便于管理
---------------------

```
mkdir -p /usr/local/redis/bin 
mkdir -p /usr/local/redis/etc
mv /src/redis-3.0.0/redis.conf /usr/local/redis/etc
cd /src/redis-3.0.0/src
mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-server /usr/local/redis/bin
```


----------


启动redis服务（redis服务端的默认连接端口是`6379`）
-------------------------------

```
/usr/local/redis/bin/redis-server
/usr/local/redis/etc/redis.conf
```
默认情况下，redis不是在后台运行的，我们需要把开启的redis后台运行

```
vi /usr/local/redis/etc/redis.conf 
```
将`daemonize`的值改为`yes`
 
查看是否启动

```
ps -ef | grep redis
```

查看是否占用6379端口号

```
netstat -tunpl | grep 6379
```

再启动redis

```
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
```


----------

客户端连接
-----

```
 /usr/local/redis/bin/redis-cli
```
退出客户端

```
exit
```
或者

```
quit
```
再或者`Ctrl`+`C`


----------

停止redis
-------

```
/usr/local/redis/bin/redis-cli shutdown
```
或者

```
pkill redis-server
```


----------
## redis的一些配置 ##
`daemonize`如果需要在后台运行，把该项改为yes
`pidfile`配置多个pid的地质，默认在/var/ren/redis.pid
`bind`绑定ip,设置后只接受来自该ip的请求
`port`监听端口，默认为6379 
`timeout`设置客户端连接时的超时时间，单位为秒 
`loglevel`分为4级，debug、verbose、notice、warning
`logfile`配置log文件地址 databases 设置数据库的个数，默认使用的数据库为0 
`save`设置redis进行数据库镜像的频率 
`rdbcompression`在进行镜像备份时，是否进行压缩
`Dbfilename`镜像备份文件的文件名
`Dir`数据库镜像备份的文件放置路径 
`Slaveof`设置数据库为其他数据库的从数据库
`Masteauth`主数据库连接需要的密码验证 
`Requirepass`设置登陆时需要的使用的密码 
`Maxclients`限制同时连接的客户数量
`Maxmemory`设置redis能够使用的最大内存 
`Appendonly`开启append only模式 
`Appendfsync`设置对appendonly.aof文件同步的频率
`vm-enabled`是否开启虚拟内存支持
`vm-swap-file`设置虚拟内存的交换文件路径
`vm-max-memory`设置redis使用的最大物理内存大小
`vm-page-size`设置虚拟内存的页大小
`vm-pages`设置交换文件的总的page数量
`vm-max-threads`设置VMIO同时使用的线程数量
`Glueoutputbuf`设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启
`hash-max-zipmap-entries`设置hash的临界值
`Activerehashing`重置hash，默认为开启


----------


**相关链接：**
[Linux下PHP安装Redis扩展（二）][1]
[Redis主从配置（三）][2]
[Redis集群搭建与简单使用（四）][3]
[Redis持久化（五）][4]


  [1]: https://segmentfault.com/a/1190000008420258
  [2]: https://segmentfault.com/a/1190000008469182
  [3]: https://segmentfault.com/a/1190000008448919
  [4]: https://segmentfault.com/a/1190000008639459