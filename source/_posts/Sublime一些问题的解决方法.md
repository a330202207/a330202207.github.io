---
title: Sublime一些问题的解决方法
date: 2017-02-20 06:26:10
tags:
- Sublime
---
##一、如何关闭Sublime的自动更新 ##

打开`Sublime`，找到`Preferences - Settings`(配置文件)
![][1]
打开后类似这样
![][2]

在最后的花括号（`}`）前添加一句：

    "update_check":false,

## 二、关闭记住上次打开的文件 ##


`同样的位置`(配置文件)中添加

    "hot_exit": false,
    "remember_open_files": false

## 三、中文文件名显示方框 ##
`老地方`(配置文件)添加
```
"dpi_scale":1.0
```
## 四、在GBK编码下的中文乱码问题 ##

 1.安装`Package Control`
 菜单 `View - Show Console` 或者 `Ctrl` + `~` 快捷键，调出 console，输入下面代码，按回车

```
import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())
```

2.关闭`Sublime`，重新打开

3.`Ctrl` + `Shift`+ `p`打开命令行模式，输入`Install Package`关键字，然后点击自动出现的下拉菜单里的第一项：`Package Control: Install Package`

4.左下角会有`=`号来回动，稍等一会，会再次在命令行下弹出一个下拉菜单。输入`ConvertToUTF8`或者`GBK Encoding Support`，选择匹配项。中文字符就可以正常显示了。

 

  [1]: https://ned.oss-cn-beijing.aliyuncs.com/sublime-1.png
  [2]: https://ned.oss-cn-beijing.aliyuncs.com/sublime-2.png