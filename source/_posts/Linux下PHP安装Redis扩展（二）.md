---
title: Linux下PHP安装Redis扩展（二）
date: 2017-02-23 06:26:43
tags:
- Redis
- yum
categories:
- NoSQL
---
> `PECL库`是一个`PHP扩展`，提供一个目录的所有已知的扩展和托管设备下载PHP扩展，PHP很多扩展都可以在这里面找到。

一、PHP Redis下载
----------------
下载地址：http://pecl.php.net/package/redis 

```
[root@localhost ~]# wget http://pecl.php.net/get/redis-3.1.0.tgz
```

二、解压安装并进入Redis目录
----------------
```
[root@localhost ~]# tar zxf redis-3.1.0.tgz 
[root@localhost ~]# cd redis-3.1.0

``` 
三、在Redis文件夹下，生成configure配置文件
----------------
```
[root@localhost redis-3.1.0]# /usr/local/php/bin/phpize
Configuring for:
PHP Api Version:         20160303
Zend Module Api No:      20160303
Zend Extension Api No:   320160303
[root@localhost redis-3.1.0]# ./configure --with-php-config=/usr/local/php/bin/php-config
[root@localhost redis-3.1.0]# make && make install

```
![][1]
![][2]
![][3]

`redis.so`扩展存放在`/usr/local/php/lib/php/extensions/no-debug-non-zts-20160303/`目录下。

四、在PHP配置文件php.ini里面加载Redis扩展 
----------------
```
extension=redis.so
```
![][4]

五、重启服务器(Apache或者Nginx)
----------------

```
[root@localhost redis-3.1.0]# service httpd restart
```

或者

```
[root@localhost redis-3.1.0]# service nginx start
```
六、测试
----------------
浏览器访问`index.php`文件,输出`phpinfo`信息，如果有Redis信息，则`安装成功`
![][5]

七、其他
----------------
`windows`下安装`Redis扩展`就更加简单了，找到对应的版本，下载`dll文件`，放到`PHP目录`下面的`ext`，修改PHP的配置文件`php.ini`，加载`extension=php_redis.dll`，重启`Apache`或者`Nginx`，查看`phpinfo`是否有`Redis`，如果有就`安装成功`。
----------
**相关链接：**
[Redis主从配置（三）][6]
[Redis集群搭建与简单使用（四）][7]
[Redis持久化（五）][8]


  [1]: https://ned.oss-cn-beijing.aliyuncs.com/1_articlex.png
  [2]: https://ned.oss-cn-beijing.aliyuncs.com/2_articlex.png
  [3]: https://ned.oss-cn-beijing.aliyuncs.com/3_articlex.png
  [4]: https://ned.oss-cn-beijing.aliyuncs.com/4_articlex.png
  [5]: https://ned.oss-cn-beijing.aliyuncs.com/5_articlex.png
  [6]: https://segmentfault.com/a/1190000008469182
  [7]: https://segmentfault.com/a/1190000008448919
  [8]: https://segmentfault.com/a/1190000008639459