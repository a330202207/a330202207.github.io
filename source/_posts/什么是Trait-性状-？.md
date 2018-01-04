---
title: 什么是Trait(性状)？
date: 2017-02-14 06:22:08
tags:
- Redis
- yum
categories:
- Linux
---
> 很多PHP开发朋友都没有弄清楚[Trait(性状)][1]。这是PHP5.4.0引入的新概念，既像`类`又像`接口`。
> `性状`是`类`的`补分实现`(即常量、属性、方法)，可以混入`一个`或者`多个`现有的PHP类中。
> `性状`有两个作用：表明类可以做什么(像是接口)；提供模块化实现(像是类)。
> 
> `PHP`使用一种`典型的继承模型`，在这种模型中，我们`先编写一个通用的根类`，实现基本功能，然后扩展这个根类，创建更具体的类，从直接父类继承实现。这叫`继承层次结构`，很多编程语言都使用了这个模式
> 大多时候，这种典型的继承模型能良好的运行，可是，如果想让两个无关的PHP类具有类似的行为，应该怎么做呢？


**假如`Shop`和`Car`两个PHP类的作用十分不同，而且在`继承层次结构`中没有`共同的父类`。 那么这两个类都应该能使用地理编码技术转化成经纬度，然后在地图上显示。要怎么解决这个问题呢？**


 1. 创建一个`父类`让`Shop`和`Car`都继承它

 2. 创建一个`接口`，定义实现地理编码功能需要哪些方法，然后让`Shop`和`Car`两个类都实现这个接口

 3. 创建一个`性状`，定义并实现地理编码相关的方法，然后把在`Shop`和`Car`两个类中混入这个性状

第一种解决方法不好，因为我们强制让两个无关的类继承同一个祖先，而很明显，这个祖先不属于各自的继承层次结构。

第二种解决方法好，因为每个类都能保有自然的继承层次结构。但是，我们要在两个类中重复实现的地理编码功能，这不符合`DRY原则`。
**`注`**：`DRY`是 `Don't Repeat Yourself(不要自我重复)`的简称，表示不要在多个地方重复编写相同的代码，如果需要修改遵守这个原则编写的代码，只需要在一出修改，改动就能体现到其他地方。

第三种解决方法最好，这么做不会搅乱这两个类原本自然的继承层次结构。


----------

### 如何创建性状？ ###

```
trait Geocodable 
{
	// 这里是性状的实现
}
```
### 如何使用性状？ ###

```
class Shop
{
	use Geocodable;

	// 这里是类的实现
}
```


----------
### 例子 ###
创建一个`Geocodable `性状
```
trait Geocodable 
{

	/**
    * 地址    
    * @var string 
    */
	protected $address;

	/**
    * 编码器对象
    * @var \Geocoder\Geocoder 
    */
	protected $geocoder;

	/** 
    * 处理后结果对象
    * @var \Geocoder\Result\Geocoded 
    */
	protected $geocoderResult;

    //注入Geocoder对象
	public function setGeocoder(\Geocoder\GeocoderInterface $geocoder)
	{
		$this->geocoder = $geocoder;
	}

    //设定地址
	public function setAddress($address)
	{
		$this->address = $address;
	}

    //返回纬度
	public function getLatitude()
	{
		if (isset($this->geocoderResult) === false) {
			$this->geocodeAddress();
		}

		return $this->geocoderResult->getLatitude();
	}

    //返回经度
	public function getLongitude()
	{
		if (isset($this->geocoderResult) === false) {
			$this->geocodeAddress();
		}

		return $this->geocoderResult->getLongitude();
	}
    
    //把地址字符串传给Geocoder实例，获取经地理编码器处理得到的结果
	protected function geocodeAddress()
	{
		$this->geocoderResult = $this->geocoder->geocode($this->address);

		return true;
	}
}
```

**使用性状**
```
$geocoderAdapter = new \Geocoder\HttpAdapter\CurlHttpAdapter();
$geocoderAdapter = new \Geocoder\Provider\GoogleMapsProvider($geocoderAdapter);
$geocoder = new \Geocoder\Geocoder($geocoderProvider);

$store = new Shop();
$store->setAddress('420 9th Avenue, New York, NY 10001 USA');
$store->setGeocoder($geocoder);

$latiude   = $store->getLatitude();
$longitude = $store->getLongitude();
echo $latitude, ':', $longitude;
```


  [1]: https://secure.php.net/manual/zh/language.oop5.traits.php
  [2]: http://gecoder-php.org