---
title: CentOS下三种PHP拓展安装方法
date: 2017-04-08 23:15:06
tags:
- MongoDB
- yum
categories:
- Linux
---

> `CentOS` 下，PHP有多种方式来安装拓展， 主要有 `包管理式`的 `yum 安装`、`pecl 安装`， 以及`源码编译安装`。
> `包管理式`的安装卸载尤为方便，而`源码编译式`的安装则方便参数调优。
> 一般的搭建本机开发环境推荐`包管理式`的安装，节约时间。而`线上部署`环境则推荐`编译安装`， 方便调优。 
> 本文以 `MongoDB` 扩展`安装举例。

工具
----------------
 - PHP版本      ： 7.0.17
 - Nginx        ： 1.10.2
 - VMware版本号 ： 12.0.0 
 - CentOS版本   ： 7.0


一、yum 安装
----------------
`yum 方式`安装能自动安装拓展的.so动态库，并配置好 `php.ini`
注：
 - 请确保自己 `yum 源` 里面有对应扩展
 - 安装完成后重启服务器 `Nginx` 或者 `Apache`
 - 浏览器访问 `index.php` 文件,输出 `phpinfo` 信息，如果有 `MongoDB` 信息，则安装成功

```
[root@localhost ~]yum search mongodb|grep php        # 搜索 yum 源里面 MongoDB 拓展
[root@localhost ~]yum -y install php70w-pecl-mongo   # 安装 PHP 对应版本的 MongoDB 扩展
[root@localhost ~]systemctl restart nginx            # 重新启动 Nginx
```
![][1]
![][2]


二、pecl 安装
----------------
官方文档：http://php.net/manual/zh/mongodb.installation.pecl.php

```
[root@localhost ~]# pecl install mongodb
-bash: pecl: 未找到命令
```
直接输入 `pecl install mongodb` 会报错，说明 `pecl` 我们没有安装，安装 `pecl`
```
[root@localhost ~]# yum -y install php70w-pear
[root@localhost ~]# pecl install mongodb
configure: error: Cannot find OpenSSL's <evp.h>
ERROR: `/var/tmp/mongodb/configure --with-php-config=/usr/bin/php-config' failed
```
到这一步又会`报错`，需要我们安装 `openssl `，安装完成后继续执行上次`未执行成功`的命令
![][7]

```
[root@localhost ~]# yum -y install openssl openssl-devel
[root@localhost ~]# pecl install mongodb
[root@localhost ~]# systemctl restart nginx             # 重新启动 Nginx
```
安装完成后在 `PHP` 配置文件 `php.ini` 里面加载 `MongoDB` 扩展
![][5]
 - 安装完成后重启服务器 `Nginx` 或者 `Apache`
 - 浏览器访问 `index.php` 文件,输出 `phpinfo` 信息，如果有 `MongoDB` 信息，则安装成功


![][6]


三、源码编译安装
----------------
源码编译包下载列表：https://pecl.php.net/packages.php
Mongodb包下载地址：https://pecl.php.net/package/mongodb
```
[root@localhost ~]# wget http://pecl.php.net/get/mongodb-1.2.8.tgz  #下载源码包
[root@localhost ~]# tar zxf mongodb-1.2.8.tgz  #解压
[root@localhost ~]# cd mongodb-1.2.8
# 可能是 /usr/local/php/bin/phpize 找到自己的 phpize 文件，php-config 同理
[root@localhost mongodb-1.2.8]# /usr/bin/phpize    
Configuring for:
PHP Api Version:         20151012
Zend Module Api No:      20151012
Zend Extension Api No:   320151012
[root@localhost mongodb-1.2.8]# ./configure --with-php-config=/usr/bin/php-config
configure: error: Cannot find OpenSSL's <evp.h>
```
到了这步`又是熟悉的味道又是熟悉的感觉`，需要我们安装 `openssl `，安装完成后继续执行上次`未执行成功`的命令
![][3]

```
[root@localhost mongodb-1.2.8]# yum -y install openssl openssl-devel
[root@localhost mongodb-1.2.8]# ./configure --with-php-config=/usr/bin/php-config
# 确保自己安装了 gcc gcc++ 如果没有安装 yum -y install gcc gcc++
[root@localhost mongodb-1.2.8]# make && make install # 编译
```
说明：`php-config` 是一个简单的命令行脚本用于`获取`所安装的 `PHP 配置`的信息。

在编译扩展时，如果安装有多个 PHP 版本，可以在配置时用 `--with-php-config` 选项来指定使用哪一个版本编译，该选项指定了相对应的 `php-config` 脚本的路径。

`编译成功`如下图
![][4]
此时在 `PHP` 配置文件 `php.ini` 里面加载 `MongoDB` 扩展
![][5]

 - 重启服务器 `Nginx` 或者 `Apache`
 - 浏览器访问 `index.php` 文件,输出 `phpinfo` 信息，如果有 `MongoDB` 信息，则安装成功


 ```
 [root@localhost mongodb-1.2.8]# systemctl restart nginx   # 重新启动 Nginx
 ```
![][6]


**总结：**
 - `pecl 安装`和`源码编译安装`区别就是：方便参数调优。
 - 在选择 `Mongo 扩展`的时候，官方提供了两种：`mongo` 和 `mongodb`

第一种： https://pecl.php.net/package/mongo
第二种： https://pecl.php.net/package/mongodb
第一种官方提示：`This package has been superseded, but is still maintained for bugs and security fixes`，已经废弃了，不过 `bug` 和 `security` 方面的问题还会继续修复，不支持 `PHP7` 

**建议：**
 - PHP 版本为 5.x 建议使用 `mongo` 扩展
 - PHP 版本为 7.x 建议使用 `mongodb` 扩展
PHP5.x 可以使用 `mongodb` 扩展。但是 PHP7.x  不可以使用 `mongo` 扩展。

**写在最后：**
如果是自己学习的话还是推荐 `yum 安装`，因为在你安装过程中会出现`缺少各种依赖`的`报错`。

  [1]: https://ned.oss-cn-beijing.aliyuncs.com/php_extension_1.png
  [2]: https://ned.oss-cn-beijing.aliyuncs.com/php_extension_2.png
  [3]: https://ned.oss-cn-beijing.aliyuncs.com/php_extension_4.png
  [4]: https://ned.oss-cn-beijing.aliyuncs.com/php_extension_7.png
  [5]: https://ned.oss-cn-beijing.aliyuncs.com/php_extension_5.png
  [6]: https://ned.oss-cn-beijing.aliyuncs.com/php_extension_6.png
  [7]: https://ned.oss-cn-beijing.aliyuncs.com/php_extension_3_1.png