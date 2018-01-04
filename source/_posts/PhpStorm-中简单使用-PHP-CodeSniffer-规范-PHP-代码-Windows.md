---
title: PhpStorm 中简单使用 PHP_CodeSniffer 规范 PHP 代码(Windows)
date: 2017-04-29 12:58:27
tags:
categories:
- PHP
---

什么是 PHP_CodeSniffer？
-------
`PHP_CodeSniffer` 是 `PEAR` 中的一个用PHP5写的用来检查嗅探 `PHP 代码`是否有`违反`一组`预先设置`好的`编码标准`的`一个包`，它是确保你的`代码简洁`一致的必不可少的`开发工具`，甚至还可以帮助程序员`减少`一些语义`错误`。

什么是 PEAR
-------
`PEAR` 是 `PHP 扩展`与`应用库`(the PHP Extension and Application Repository)的`缩写`。它是一个 `PHP扩展及应用`的一个`代码仓库`，简单地说，`PEAR` 之于 `PHP` 就像是CPAN(Comprehensive Perl Archive Network)之于 `PEAR`。
` PEAR` 的基本目标是发展成为 `PHP 扩展`和`库代码`的`知识库`，而这个项目最有雄心的目标则是试图`定义`一种`标准`，这种`标准`将`帮助`开发者编写可`移植`、`可重用`的`代码`。

那么对于一个开发团队统一的编码风格，有助于他人对代码的理解和维护，对于大项目来说尤其重要。

安装 PEAR
-------
1、下载 [PEAR][1]到 PHP 目录
2、打开 `dos 命令窗口`(win + R 输入 cmd )，进入到 `PHP` 目录下
```
php go-pear.phar
```
![][2]
输入命令，直接`回车`默认 `system` 继续
![][3]
直接回车，出现如下，表示安装`成功`
![][6]

安装 PHP_CodeSniffer
-------
```
pear install PHP_CodeSniffer
```
如果出现报错,修改 `pear` 配置（路径为 `php 目录`）

![][4]
```
pear config-set doc_dir D:\PDT\php_7.0.14
pear config-set cfg_dir D:\PDT\php_7.0.14
pear config-set data_dir D:\PDT\php_7.0.14
pear config-set test_dir D:\PDT\php_7.0.14
pear config-set www_dir D:\PDT\php_7.0.14
pear install PHP_CodeSniffer
```


PhpStorm 配置
-------
打开 `PhpStorm` 的配置框，找到 `Languages & Frameworks -> php-> Code Sniffer`，不同版本的 `PhpStorm` 可能会有出入，直接搜索 `Code Sniffer` 也可以。
点击如下进行编辑：
![][7]
设置 `PHP Code Sniffer path` 为 `phpcs.bat` 的路径。
![][8]
点击 `Validate`，出现如下图表示设置`成功`：
![][9]
打开配置搜索 `Inspections`, 展开 `PHP`，勾选 `PHP Code Sniffer validation`, 选择 `Coding standard` 为 `PSR2` , 点击OK确定
![][10]
接下来，在编码 `PHP` 的时候就会出现`规范提示`:
![][11]


  [1]: http://pear.php.net/go-pear.phar
  [2]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/1.png
  [3]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/2.png
  [4]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/3.png
  [5]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/4.png
  [6]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/5.png
  [7]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/6.png
  [8]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/7.png
  [9]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/8.png
  [10]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/9.png
  [11]: http://olln3wpar.bkt.clouddn.com/image/PHP_CodeSniffer/10.png