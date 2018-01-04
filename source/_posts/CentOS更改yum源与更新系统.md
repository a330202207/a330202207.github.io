---
title: CentOS更改yum源与更新系统
date: 2017-03-06 06:43:42
tags:
- yum
categories:
- Linux
---
>  - VMware版本号：12.0.0  
>  - CentOS版本：7.0

一、更换之前确保自己安装wget
-------

``` 
[root@localhost ~]# yum list wget
```
没有安装：
```
[root@localhost ~]# yum -y install wget
```
二、首先备份`/etc/yum.repos.d/CentOS-Base.repo`
-------
```
[root@localhost ~]# cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

三、进入`yum源`配置文件所在文件夹
-------

```
[root@localhost yum.repos.d]# cd /etc/yum.repos.d/
```

四、下载163的`yum源`配置文件，放入`/etc/yum.repos.d/`(操作前做好相应备份)
-------

```
[root@localhost yum.repos.d]# wget http://mirrors.163.com/.help/CentOS6-Base-163.repo
```

五、运行yum makecache生成缓存
-------

```
[root@localhost yum.repos.d]# yum makecache
```

六、运行yum makecache生成缓存
-------

```
[root@localhost yum.repos.d]# yum -y update
```




