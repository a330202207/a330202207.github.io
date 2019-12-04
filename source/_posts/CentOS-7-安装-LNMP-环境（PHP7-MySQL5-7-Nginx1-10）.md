---
title: CentOS 7 安装 LNMP 环境（PHP7 + MySQL5.7 + Nginx1.10）
date: 2017-04-10 05:54:08
tags:
- yum
categories:
- Linux
---

工具
----------------
 - VMware版本号 ： 12.0.0 
 - CentOS版本   ： 7.0
一、修改 yum 源
----------------
```
[root@localhost ~]# rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
[root@localhost ~]# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
[root@localhost ~]# rpm -Uvh  http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
```
webtatic：https://webtatic.com
MySQL:https://dev.mysql.com/downloads/repo/yum/

二、安装 Nginx、MySQL、PHP
----------------
```
[root@localhost ~]# yum -y install nginx
[root@localhost ~]# yum -y install mysql-community-server
[root@localhost ~]# yum -y install php70w-devel php70w.x86_64 php70w-cli.x86_64 php70w-common.x86_64 php70w-gd.x86_64 php70w-ldap.x86_64 php70w-mbstring.x86_64 php70w-mcrypt.x86_64  php70w-pdo.x86_64   php70w-mysqlnd  php70w-fpm php70w-opcache php70w-pecl-redis php70w-pecl-mongo
```
三、配置
----------------
1、配置 MySQL
`MySQL` 安装完成之后，在 `/var/log/mysqld.log `文件中给 `root` 生成了一个`默认密码`。通过下面的方式找到`root 默认密码`，然后登录 `MySQL` 进行修改：
```
[root@localhost ~]# systemctl start mysqld    # 启动 MySQL
[root@localhost ~]# grep 'temporary password' /var/log/mysqld.log  # 查找默认密码
2017-04-10T02:58:16.806931Z 1 [Note] A temporary password is generated for root@localhost: iacFXpWt-6gJ
```
登录 `MySQL`:
```
[root@localhost ~]# mysql -uroot -p'iacFXpWt-6gJ'  
```
修改`root 默认密码`:
```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyPass1!';
```
或者：
```
mysql> set password for 'root'@'localhost'=password('123abc'); 
```
`注`：`MySQL5.7` 默认安装了`密码安全检查插件（validate_password）`，默认密码检查策略要求密码必须包含：大小写字母、数字和特殊符号，并且长度不能少于8位。
否则会提示ERROR 1819 (HY000): `Your password does not satisfy the current policy requirements`错误
![][3]
详见 MySQL 官网密码策略详细说明：https://dev.mysql.com/doc/refman/5.7/en/validate-password-options-variables.html#sysvar_validate_password_policy
配置默认编码为 `utf8`：
修改 `/etc/my.cnf` 配置文件，在 `[mysqld]` 下添加编码配置，配置完成后重启：
```
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
[root@localhost ~]# systemctl restart mysqld    # 重启 MySQL
```
![][6]
设置开机启动：
```
[root@localhost ~]# systemctl enable mysqld
```
默认配置文件路径：
配置文件：`/etc/my.cnf`
日志文件：`/var/log//var/log/mysqld.log`
服务启动脚本：`/usr/lib/systemd/system/mysqld.service`
socket 文件：`/var/run/mysqld/mysqld.pid`

2、配置 Nginx
安装完成以后查看自己`防火墙`是否开启，如果已开启，我们需要`修改`防火墙`配置`，开启 `Nginx` 外网端口访问。
```
[root@localhost ~]# systemctl status firewalld
```
![][1]
如果显示 `active (running)`，则需要调整防火墙规则的配置。

修改 `/etc/firewalld/zones/public.xml`文件，在zone一节中增加
保存后重新加载 `firewalld` 服务：
```
[root@localhost ~]# vim /etc/firewalld/zones/public.xml
<zone>
    ...
    <service name="nginx"/>
<zone>
[root@localhost ~]# systemctl reload firewalld
```
![][2]
修改 `Nginx` 配置：
```
[root@localhost ~]# vim /etc/nginx/nginx.conf
```
在 `server {}` 里添加：
```
location / {
    #定义首页索引文件的名称
    index index.php index.html index.htm;   
}

# PHP 脚本请求全部转发到 FastCGI处理. 使用FastCGI默认配置.
location ~ .php$ {
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```
![][4]
配置完成重启 `Nginx`：
```
[root@localhost ~]# systemctl start nginx    # 启动 Nginx
```
`注`：本文只是简单配置 `Nginx`，具体更多配置请自行百度。
设置开机启动:
```
[root@localhost ~]# systemctl enable nginx
```

3、设置开机启动 `php-fpm`：
```
[root@localhost ~]# systemctl enable php-fpm
[root@localhost ~]# systemctl start php-fpm    # 启动 php-fpm
```

四、测试：

 - 在 `/usr/share/nginx/html` 文件下创建php文件，输出 `phpinfo` 信息
 - 浏览器访问 `http://<内网IP地址>/phpinfo.php`，如果看到 PHP 信息，说明`安装成功`
 ![][5]
 
  [1]: https://ned.oss-cn-beijing.aliyuncs.com/LNMP_1.png
  [2]: https://ned.oss-cn-beijing.aliyuncs.com/LNMP_2.png
  [3]: https://ned.oss-cn-beijing.aliyuncs.com/LNMP_11.png
  [4]: https://ned.oss-cn-beijing.aliyuncs.com/LNMP_12.png
  [5]: https://ned.oss-cn-beijing.aliyuncs.com/LNMP_13.png
  [6]: https://ned.oss-cn-beijing.aliyuncs.com/LNMP_15.png