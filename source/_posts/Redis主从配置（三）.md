---
title: Redis主从配置（三）
date: 2017-03-07 06:43:42
tags:
- Redis
categories:
- NoSQL
---
工具
----------------
 - VMware版本号：12.0.0 
 - CentOS版本：7.0
 - 两台虚拟机(IP)：192.168.29.18、192.168.29.19

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

修改配置文件(redis.conf)
----------------

```
##192.168.29.18(主)
port         8000                        //端口        
bind         192.168.18 127.0.0.1        //redis 在 server 上所有有效的网络接口上监听客户端连接，多个IP用空格隔开
daemonize    yes                         //redis后台运行
pidfile      /var/run/redis_8000.pid
requirepass  root                        //设置认证密码

##192.168.29.19(从)
port        8001                         //端口        
bind        192.168.19 127.0.0.1         //redis 在 server 上所有有效的网络接口上监听客户端连接，多个IP用空格隔开
daemonize   yes                          //redis后台运行
pidfile     /var/run/redis_8001.pid
slaveof     192.168.29.19 8001           //slaveof 主机ip 端口号
masterauth  root                         //主机认证密码
```

关闭防火墙
----------------
```
$ firewall-cmd --state      ##查看防火墙状态
running                     ##防火墙开启

$ systemctl stop firewalld  ##关闭防火墙
$ firewall-cmd --state
not running                 ##防火墙关闭
```
`注`: `CentOS 7` 关闭防火墙与 `CentOS 6` 有所不同

启动 Redis
----------------

```
$ /usr/local/redis-3.2.8/src/redis-server /usr/local/redis-3.2.8/redis.conf
```

检查各 Redis 启动情况
----------------
```
##192.168.29.18(主)
$ ps -ef | grep redis           //redis是否启动成功
$ netstat -tnlp | grep redis    //监听redis端口
```
![图片描述][1]

客户端连接-测试同步
----------------

```
##主 -p 端口号 -a 主机验证密码 -h 默认为127.0.0.1
$ /usr/local/redis-3.2.8/src/redis-cli -p 8000 -a root  

##从
$ /usr/local/redis-3.2.8/src/redis-cli -p 8001            
```
`注:`
1、因为 `redis.conf` 文件中`bind`参数为：`192.168.29.19 127.0.0.1`
所以这里不用添加参数：`/usr/local/redis-3.2.8/src/redis-cli -h 192.168.29.19 -p 8000 -a root`
2、从机`redis.conf` 文件中`masterauth`参数`已配置过`验证密码，所以不用添加参数 `-a`

查看连接状态
```
##主
127.0.0.1:8000> info Replication 
```

![图片描述][2]
```
127.0.0.1:8001> info Replication 
```
![图片描述][3]

在`主机`上执行命令

```
127.0.0.1:8000> set name redis
```

在`从机`上查看

```
127.0.0.1:8001> get name
```
![图片描述][4]

说明`主从配置成功`
PS：计算机不存在`玄学` /捂脸


----------
**相关链接：**

[Linux下PHP安装Redis扩展（二）][5]
[Redis集群搭建与简单使用（四）][6]


  [1]: /img/bVJGHd
  [2]: /img/bVJGSR
  [3]: /img/bVJGTN
  [4]: /img/bVJGWL
  [5]: https://segmentfault.com/a/1190000008420258
  [6]: https://segmentfault.com/a/1190000008448919