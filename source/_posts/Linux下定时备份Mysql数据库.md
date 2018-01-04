---
title: Linux下定时备份MySQL数据库
date: 2017-02-18 06:25:18
tags:
categories:
- MySQL

---
## 首先连接数据库，查看数据库是否连接成功##

    mysql -u 用户名 -h 脚本中地址 -p密码
    
## 创建脚本server_mysql_bak.sh ##

    #!/bin/sh
    # Database info
    DB_HOST="127.0.0.1"
    DB_NAME="db_test"
    DB_USER="root"
    DB_PASS="root"
    
    # Others vars
    BCK_DIR="/data/backup/data/"
    DATE=`date +%F`
    
    # TODO
    mysqldump --opt -h$DB_HOST -u$DB_USER -p$DB_PASS $DB_NAME | gzip > $BCK_DIR/$DB_NAME-$DATE.gz



## 计划任务 ##

    //编辑用户目前的crontab任务列表
    crontab -e
    //分钟 小时 日 月 天 执行目录的脚本(代码)
    00 03 * * * sh /data/shell/server_mysql_bak.sh

每天`凌晨3点`开始执行(/data/shell/)目录下这个脚本
    
## 重启脚本 ##    

    service crond restart

    