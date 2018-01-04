---
title: PHP中的自动加载
date: 2018-01-02 16:59:31
tags:
categories:
- PHP
---


所谓的`自动加载`意思就是我们在 `new` 一个 `class` 的时候必须先 `include` 或者 `require` 的类文件，如果没有 `include` 或者 `require`，则会报错。那这样我们就必须在文件头部写上许多 `include` 或 `require` 文件，非常麻烦，
为了使得没有 `include` 或者 `require` 类的时候也正常 `new` 一个类，所以有了 `自动加载` 的概念，也就是说 `new` 一个类之前不用事先包含类文件也可以正常 `new`，这样我们的文件头部就不用包含许多 `include(require)`。
如何才能`自动加载`呢？ PHP 5 版本更新了`自动加载`需要的一个`魔术方法`—— `__autoload($class_name)`
`注`： PHP 7.2.0版本已经废弃。

一、自动加载的原理以及 __autoload 的使用
----------------------------------------------

`自动加载`的原理，就是在我们 `new` 一个 `class` 的时候，PHP系统如果找不到你这个类，就会去自动调用本文件中的 `__autoload($class_name)` 方法，我们 `new` 的这个 `class_name` 就成为这个方法的参数。所以我们就可以在这个方法中根据我们需要 `new class_name` 的各种判断和划分就去 `require` 对应的路径类文件，从而实现自动加载。

我们先一步步来，看下__autoload()的自动调用，看个例子： 

```
// index.php
$db = new DB();
```
如果我们不手动导入 `DB` 类，程序可能会报错，说找不到这个类：
```
Fatal error: Class 'DB' not found in D:\www\test\index.php on line 2
```
那么，我们现在加入 `__autoload()` 这个方法再看看：
```
// index.php
$db = new DB();

function __autoload($className)
{
    echo $className;
    exit();
}
```
根据上面`自动加载`机制的描述，会输出：`DB`， 也就是我们需要 `new` 的类的类名。所以，这个时候我们就可以在 `__autoload()` 方法里，根据需要去加载类库文件了。

```
// index.php
$db = new DB();

function __autoload($className)
{
    require $className . '.php';
}
```

```
// DB.php
class DB
{
    public function __construct()
    {
        echo 'Hello DB';
    }
}
```
这样子我们就很轻松的将我们需要 `new` 的 `class` 全部加载进来，我们就可以轻松的 new N个 class，比如：
```
function __autoload($className)
{
    require $className . '.php';
}

$db = new DB();
$info = new Info();
$gender = new Gender();
$name = new Name();

// 静态方法直接调用的
Height::test();
```

二、spl_autoload_register 的使用
----------------------------------------------
小的项目，用 `__autoload()` 就能实现基本的自动加载了。但是如果一个项目过大，或者需要不同的`自动加载`来加载不同路径的文件，这个时候 `__autoload` 就不能满足了，原因是一个项目中仅能有一个这样的 `__autoload()` 函数，因为 PHP 不允许函数重名，也就是说你不能声明2个 `__autoload()` 函数文件，否则会报致命错误，所以 `spl_autoload_register()` 函数诞生了，并且取而代之它。它执行效率更高，更灵活
`spl_autoload_register` 可以很好地处理需要多个加载器的情况，这种情况下 `spl_autoload_register` 会按顺序依次调用之前注册过的加载器。作为对比， `__autoload` 因为是一个函数，所以只能被定义`一次`。

当我们去 `new` 一个找不到的 `class` 时，PHP 就会去自动调用 `sql_autoload_resister` 注册的函数，这个函数通过它的参数传进去：

`sql_autoload_resister($param)` 这个参数可以有多种形式：
```
sql_autoload_resister('loadFunction'); // 函数名
sql_autoload_resister(array('loadObject', 'loadFunction')); // 类和静态方法
sql_autoload_resister('loadObject::loadFunction'); // 类和方法的静态调用

// PHP 5.3 之后，也可以像这样支持匿名函数了。
spl_autoload_register(function($className){
    if (is_file('./lib/' . $className . '.php')) {
        require './lib/' . $className . '.php';
    }
});
```
```
// index.php
function loader($className)
{
    require $className . '.php';
}
spl_autoload_register('loader'); // 将 loader 函数注册到自动加载队列中。
$db = new DB(); // 找不到 DB 类，就会自动去调用刚注册的 loader 函数了
```
上面就是实现了`自动加载`的方式，我们同样也可以用类加载的方式调用，但是必须是 `static` 方法：
```
class autoLoading
{
    //必须是静态方法，不然报错
    public static function loader($className)
    {   
        require $className . '.php';
    }
}

spl_autoload_register(array('autoLoading', 'loader')); 
spl_autoload_register('autoLoading::loader');// 静态方式调用
$db = new DB(); //会自动找到
```
`注`：如你同时使用 `spl_autoload_register` 和 `__autoload` ， `__autoload` 会失效！！！

三、多个 spl_autoload_register 的使用
----------------------------------------------
`spl_autoload_register` 是可以多次重复使用的，这一点正是解决了 `__autoload` 的短板，那么如果一个页面有多个，执行顺序是按照注册的顺序，一个一个往下找，如果找到了就停止。

我们来看下这个例子， `DB.php` 就在本目录下，`Info.php` 在 `/lib/` 目录下。
```
// Info.php
class Info
{
    public function __construct()
    {
        echo 'Hello Info';
    }
}
```
```
// index.php

function loader1($className)
{
    echo 1;
    if (is_file($className . '.php')) {
        require $className . '.php';
    }
}
function loader2($className)
{
    echo 2;
    if (is_file('./app/' . $className . '.php')) {
        require './app/' . $className . '.php';
    }
}
function __autoload($className)
{
    echo 3;
    if (is_file('./lib/' . $className . '.php')) {
        require './lib/' . $className . '.php';
    }
}
//注册了3个
spl_autoload_register('loader1');
spl_autoload_register('loader2');
spl_autoload_register('__autoload'); 
$db = new DB(); // DB 就在本目录下
$info = new Info(); // Info 在/lib/Info.php
```
我们注册了3个自动加载函数。执行结果是啥呢？
```
1Hello DB
123Hello Info
```
我们分析下：

 1. `new DB` 的时候，就按照注册顺序，先去找 `loader1()` 函数了，发现找到了，就停止了，所以输出 `1 Hello Word`
 2. `new Info` 的时候，先是安装注册顺序，先找 `loader1()`，所以输出了 1，发现没找到，就去 `loader2()` 里面去找，所以输出了 2，还是没这个文件，就去 `__autoload()` 函数里找，所以，先输出了 3，再输出 `Hello Info`

`注`: `spl_autoload_register` 使用时， `__autoload `会无效，有时候，我们希望它继续有效，就可以也将它注册进来，就可以继续使用。
我们可以打印 `spl_autoload_functions()` 函数，来显示一共注册了多少个自动加载：
```
var_dump(spl_autoload_functions());
//数组的形式输出
array (size=3)
  0 => string 'load1' (length=5)
  1 => string 'load2' (length=5)
  2 => string '__autoload' (length=10)
```

四、spl_autoload_register 自动加载与 namespace 命名空间的使用
----------------------------------------------
`自动加载`现在是PHP现代框架的基石，基本都是 `spl_autoload_register` 来实现自动加载。`namespace` 也是使用比较多的。所以 `spl_autoload_register` + `namespace` 就成为了一个主流。根据 `PSR-0` 的规范，`namespace` 命名已经非常规范化，所以用 `namespace` 就能找到详细的路径，从而找到类文件。

```
namespace AutoLoading;

// AutoLoading\loading.php
class loading 
{
    public static function autoload($className)
    {
        // 根据 PSR-O 的第4点 把 \ 转换层（目录风格符） DIRECTORY_SEPARATOR , 
        // 便于兼容Linux文件找。Windows 下（/ 和 \）是通用的
        // 由于 namspace 很规格，所以直接很快就能找到
       $fileName = str_replace('\\', DIRECTORY_SEPARATOR,  DIR . '\\'. $className) . '.php';
       if (is_file($fileName)) {
            require $fileName;
       } else {
            echo $fileName . ' is not exist'; die;
       }
    }
}
```
上面就是一个自动加载的核心思想方法。下面我们就来 `spl_autoload_register` 来注册这个函数：
```
// index.php

// 定义当前的目录绝对路径
define('DIR', dirname(__FILE__));

// 加载这个文件
require DIR . '/loading.php';

// 采用“命名空间”的方式注册。php 5.3 加入的
// 也必须是 static 静态方法调用，然后就像加载 namespace 的方式调用，注意：不能使用 use
spl_autoload_register("\\AutoLoading\\loading::autoload"); 

// 调用三个 namespace 类
//定位到 Lib 目录下的 Name.php 
Lib\Name::test();

// 定位到App目录下 Android 目录下的 Name.php
App\Android\Name::test();

// 定位到App目录下 Ios 目录下的 Name.php
App\Ios\Name::test();
```
由于我们是采用PSR-O方式来定义namespace的命名的，所以很好的定位到这个文件的在哪个目录下了。
```
// APP\Android\Name

namespace App\Android;

class Name
{
    public function __construct()
    {
        echo __NAMESPACE__ . "<br>";
    }
    
    public static function test()
    {
        echo  __NAMESPACE__ . ' static function test <br>';
    }
}
```
所以就会很容易找到文件，并输出：
```
Lib static function test 
App\Android static function test 
App\Ios static function test 
```

五、同命名空间下的相互调用
----------------------------------------------
在平时我们使用命令空间时，有时候可能是在同一个命名空间下的2个类文件在相互调用。这个时候就要注意，在自动调用的问题了。

比如在 `Lib\Factory.php` 中调用 `Lib\Db\MySQL.php`。怎么调用呢？以下是错误的示范：
```
new Lib\Db\MySQL();  
// 报错，提示说 D:\www\test\module\Lib\Lib\Db\MySQL.php is not exist
```
这种方式是在 `Lib\` 命名空间的基础上来加载的。所以会加载`2`个 `Lib`。这种方式相当于`相对路径`在`加载`。
正确的做法是，如果是在同一个命名空间下平级的2个文件。可以直接调用，不用命名空间。
```
new MySQL(); // 直接这样就可以了。
new Db\MySQL(); // 如果有个Db文件夹，就这样。
```
还有一种方法就是使用 `use` 。使用 `use` 就可以带上 `Lib` 了。`use` 使用的是`绝对路径`。
如果在 `Lib\Db\MySQL.php` 中调用 `Lib\Register.php` 呢？
```
use Lib\Register;
Register::getInstance();
```
因为现在已经在 `Lib\Db` 命名空间里了，如果你不用 `use` ，而是使用 `Lib\Register::getInstance()` 或者使用 `Register::getInstance()` 的话。将是在 `Lib\Db` 这个空间下进行`相对路径`的加载，是错误的。