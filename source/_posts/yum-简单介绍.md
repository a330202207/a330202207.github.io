---
title: yum 简单介绍
date: 2017-03-21 23:17:33
tags:
- yum
categories:
- Linux
---
yum 简单介绍
----------------
`yum`（全称为 Yellow dog Updater, Modified）是一个在 `Fedora`和  `RedHat` 以及 `CentOS` 中的 `Shell` 前端软件包管理器。基于 `RPM` 包管理，能够从`指定`的服务器自动下载 `RPM` 包并且安装，可以`自动处理依赖`性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装

`yum` 主要是更方便的添加、删除、更新RPM包，`自动解决`软件包之间的`依赖关系`，`方便系统更新`及`软件管理`。`yum` 通过软件仓库（repository）进行软件的下载、安装等，软件仓库可以是一个 HTTP 或 FTP 站点，也可以是一个本软件池，资源仓库也可以是多个，在 /etc/yum.conf 文件中进行相关配置即可。在yum的资源库中，会包括 `RPM` 的头信息（header），头信息中包括了软件的功能描述、依赖关系等。通过分析这些信息，`yum` 计算出依赖关系并进行相关的升级、安装、删除等操作

yum 命令
----------------

命令格式：
```
yum [options] COMMAND
```

常用选项(options)列表：

```
-h, --help         #显示帮助信息
-t, --tolerant     #容错
-C, --cacheonly    #完全从系统缓存中运行，不更新缓存
-c [config file], --config=[config file]      #本地配置文件
-R [minutes], --randomwait=[minutes]          #命令最大等待时间
-d [debug level], --debuglevel=[debug level]  #设置调试级别
-e [error level], --errorlevel=[error level]  #设置错误等级
-q, --quiet     #退出运行
-v, --verbose   #详细模式
-y, --assumeyes #对所有交互提问都回答 yes
```
命令(COMMAND)列表：

```
check         #检测 rpmdb 是否有问题
check-update  #检查可更新的包
clean         #清除缓存的数据
deplist       #显示包的依赖关系
distribution-synchronization  #将已安装的包同步到最新的可用版本
downgrade     #降级一个包
erase         #删除包
groupinfo     #显示包组的详细信息
groupinstall  #安装指定的包组
grouplist     #显示可用包组信息
groupremove   #从系统删除已安装的包组
help          #删除帮助信息
history       #显示或使用交互历史
info          #显示包或包组的详细信息
install       #安装包
list          #显示可安装或可更新的包
makecache     #生成元数据缓存
provides      #搜索特定包文件名
reinstall     #重新安装包
repolist      #显示已配置的资源库
resolvedep    #指事实上依赖
search        #搜索包
shell         #进入yum的shell提示符
update        #更新系统中的包
upgrade       #升级系统中的包
version       #显示机器可用源的版本
```
`注`：以上可用命令和选项由于 `yum 版本`的不同可能会有所有`不同`

yum 使用示例
----------------
1、安装方式：

 - 单独安装
 - 包组安装

```
# 安装软件包 foo
yum install foo
# 安装 Web server 软件包组
yum groupinsall "Web server"
```
`注`：`groupinsall` 是一种快捷安装方式，他会将包组中所需的软件包一次性全部安装。如，上例中的 "Web server" 包组可能会包含： httpd、 crypto-utils 等软件包

2、更新、升级
对于已安装的程序，可以进行升级操作，有以下几种升级方式：

```
# 检查可用更新
yum check-update 
# 全部更机关报
yum update
# 更新 foo 软件包
yum update foo
# 或
yum upgrade foo
# 升级 Web server 软件包组
yum groupupdate "Web server"
```
更新安装包时，可以使用update或upgrade，二者区别如下：
 - yum update 是更新下载源里面的 metadata，包括这个源有什么包、每个包什么版本之类的
 - yum upgrade 会根据update后的元信息对软件包进行升级

3、删除
删除时，可以删除单个软件包或软件包组：

```
# 删除软件 foo
yum remove foo
# 删除 Web server 软件包组
yum groupremove "Web server"
```
4、查找
通过 `search` 命令可以查找软件包，而 `info` 命令可以用来显示`软件包信息`

```
# 查找名称包含 foo 的软件包
yum search foo
# 显示名为 foo 的软件包信息
yum info foo
# 显示软件包 foo 的依赖关系
yum deplist foo
# 显示软件包组 Web server 的信息
yum groupinfo "Web server"
# 显示已安装的软件包 
yum list installed
```

yum 的配置
----------------
`yum` 的配置文件分为`main` 和`repository` ：
 1. `main`:定义了`全局配置选项`，该文件只有一个。通常位于 `/etc/yum.conf`
 2. `repository`:定义了`源服务器`的具体配置，可能是一或多个。通常位于 `/etc/yum.repo.d` 目录
 
可以通过以下命令查看yum的配置：

```
cat /etc/yum.conf
```

主要配置项如下：

```
[main]
# yum 的缓存目录，用于存储下载的 RPM 包和数据库
cachedir=/var/cache/yum/$basearch/$releasever

# 安装完成后是否保留软件包，0为不保留（默认为0），1为保留
keepcache=0
   
# Debug 信息输出等级，范围为0-10，默认为2
debuglevel=2

# yum 日志文件位置，用户通过该文件查询做过的更新   
logfile=/var/log/yum.log
   
# 是否只安装和系统架构匹配的软件包。可选项为：1､0，默认为1
# 设置为 1 时不会将 i686 的软件包安装在适合i386的系统中
exactarch=1

# update 设置，是否允许更新陈旧的 RPM 包，相当于 upgrade
obsoletes=1
   
# 是否进行 GPG(GNU Private Guard) 校验，以确定 RPM 包的来源是有效和安全
# 当在这个选项设置在[main]部分，则对每个 repository 都有效
plugins=1

# 是否启用插件，默认1为允许，0表示不允许
gpgcheck=1
   
# 排除某些软件在升级名单之外，可以用通配符，各个项目用空格隔开
exclude=*.i?86 kernel kernel-xen kernel-debug

# 可同时安装多少程序包   
installonly_limit=5

# Bug 追踪路径   
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=16&ref=http://bugs.centos.org/bug_report_page.php?category=yum

# 当前发行版版本号   
distroverpkg=centos-release
```

相关链接
[CentOS更改yum源与更新系统][1]


  [1]: https://www.upupjie.com/2017/03/07/CentOS%E6%9B%B4%E6%94%B9yum%E6%BA%90%E4%B8%8E%E6%9B%B4%E6%96%B0%E7%B3%BB%E7%BB%9F/