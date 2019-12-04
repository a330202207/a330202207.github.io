---
title: MySQL5.7 主从复制配置
date: 2017-07-16 13:06:09
tags:
categories:
- MySQL
---

一、主从复制原理
----------------
>`MySQL` 主从复制是一个`异步`的`复制`过程，`主库`发送`更新事件`到`从库`，`从库`读取`更新记录`，并`执行`更新`记录`，使得`从库`的内容与主库`保持一致`。每一个`主从复制`的连接，都有`三个线程`。拥有`多个从库`的`主库`为`每一个连接`到`主库`的`从库`创建一个 `Binarylog` 输出线程，`每一个从库`都有它自己的 `I/O` 线程和 `SQL` 线程。
![][8]
**步骤**：
1.`主库`会将`所有`的`更新`记录保存到 `Binarylog` 文件。
2.每当有`从库`连接到`主库`的时候，`主库`都会创建一个 `log dump` 线程发送 `Binarylog` 文件到`从库`。
3.在`从库`里，当复制开始的时候，`从库`就会创建`两个线程`进行处理，一个 `I/O` 线程，一个 `SQL` 线程。
4. `I/O` 线程去请求`主库`的  `Binarylog`文件，并将得到的 `Binarylog` 文件写到 `Relaylog` 文件中。
5. `SQL` 线程会读取 `Relaylog` 文件中的日志，并解析成具体操作，来实现主从的操作一致，而最终数据一致。

二、工具
----------------
 - VMware版本：12.0.0 
 - CentOS版本：7.0
 - MySQL版本: 5.7.18
 - Master 服务器：192.168.78.128
 - Slave 服务器：192.168.78.130

三、准备工作
----------------
1.安装 MySQL5.7 [详见此处][1]
2.如果`从服务器`是`克隆`的`主服务器`，则修改 `auto.cnf` 文件中 `server-uuid` 值，不然后面`主从复制`会报 `1593` 错误。修改完记得重启MySQL
![][2]
3.关闭`主、从`服务器`防火墙`：
```
$ firewall-cmd --state      ##查看防火墙状态
running                     ##防火墙开启

$ systemctl stop firewalld  ##关闭防火墙
$ firewall-cmd --state
not running                 ##防火墙关闭
```

4.修改`主从配置`文件(my.cnf)

192.168.78.128(master)：

```
bind-address=192.168.78.128         #当前服务器地址
log_bin=mysql-bin  
server_id=128
```

192.168.78.130(slave)：

```
bind-address=192.168.78.130         #当前服务器地址
log_bin=mysql-bin  
server_id=130
```
![][3]

重启主从 `MySQL`:

```
$ systemctl restart mysqld
```

`注`: `server_id` 必须唯一


5.`master` 上创建一个测试数据库

```
$ create database test;
$ use test;
$ create table test(id int(11), value varchar(20));
$ insert into test values(1, 'aa'),(2, 'bb'),(3, 'cc');
```

四、步骤
----------------
1.`master`创建授权用户：
192.168.78.128(master)：

```
##创建 test 用户，指定该用户只能在master 192.168.78.130 上
##使用 MyPass1! 密码登录
mysql> create user 'test'@'192.168.78.130' identified by 'MyPass1!';

##为 test 用户赋予 REPLICATION SLAVE 权限。
mysql> grant replication slave on *.* to 'test'@'192.168.78.130';

##查看用户
mysql> select user,host from mysql.user;

##查看 master 状态
mysql> show master status;
```
![][4]
`注`：
这里的 `mysql-bin.000001`和 `Position` 配置 `slave` 的时候需要用到

2.将 `master` 中现有的数据信息`导出`：

```
$ mysqldump -u root -p --all-databases --master-data > all.sql
```

3.将 `all.sql` 发送到 `slave` 服务器 `tmp` 目录下:

```
$ scp all.sql root@192.168.78.130:/tmp
```

4.`slave` 导入 `master` 数据，使 `master-slave` 数据`保持一致`：

192.168.78.130(slave)：

```
$ mysql -uroot -p < all.sql
```

5.使 `slave` 与 `master` 建立连接，从而同步

```
mysql> change master to
    -> master_host='192.168.78.128',
    -> master_user='test',
    -> master_password='MyPass1!',
    -> master_log_file='mysql-bin.000001',
    -> master_log_pos=1244;
    
mysql> start slave;

mysql> show slave status \G
```
![][5]

`注`：

 - `master_log_file` 和 `master_log_pos`值为`主库`上面执行`show master status`得到
 - 如果 `Slave_IO_Running` 和 `Slave_SQL_Running` 都为 `Yes`，说明配置`成功`。 如果
 - 其中一项不为 `Yes`，查看 `Last_IO_Errno` 错误码和`错误信息`，或者查看 `MySQL` 日志信息并查找对应问题。

五、主从配置检验
----------------

`master` 插入一条数据，`slave`查看是否成功
192.168.78.128(master)：
![][6]

192.168.78.130(slave)：
![][7]

六、思考
----------------
主从复制同步`延迟`如何解决？

  [1]: http://upupjie.com/2017/04/10/CentOS-7-%E5%AE%89%E8%A3%85-LNMP-%E7%8E%AF%E5%A2%83%EF%BC%88PHP7-MySQL5-7-Nginx1-10%EF%BC%89/
  [2]: https://ned.oss-cn-beijing.aliyuncs.com/image/Mysql/jpg/1.png
  [3]: https://ned.oss-cn-beijing.aliyuncs.com/image/Mysql/jpg/2.png
  [4]: https://ned.oss-cn-beijing.aliyuncs.com/image/Mysql/jpg/3.png
  [5]: https://ned.oss-cn-beijing.aliyuncs.com/image/Mysql/jpg/5.png
  [6]: https://ned.oss-cn-beijing.aliyuncs.com/image/Mysql/jpg/6.png
  [7]: https://ned.oss-cn-beijing.aliyuncs.com/image/Mysql/jpg/7.png
  [8]: https://ned.oss-cn-beijing.aliyuncs.com/image/Mysql/jpg00.png
