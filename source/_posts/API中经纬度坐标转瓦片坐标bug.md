layout: '''百度javascirpt'
title: API中经纬度坐标转瓦片坐标bug
date: 2016-05-09 00:02:39
tags: JS, 地图, Bug
---

## 背景
目前我正在写一篇不同互联网地图经纬度坐标与瓦片坐标相互转换的文章，涉及百度地图、高德地图、谷歌地图，并提供了转换的类库。某一互联网地图的经纬度坐标与瓦片坐标相互转换只与该地图商的墨卡托投影和瓦片编号的定义有关，跟地图商采用的大地坐标系标准无关。
在百度地图的转换上出现一个错误，怀疑是bug。

## Bug描述
### 百度平面坐标
百度地图采用了自己的坐标系统和瓦片编号方法，主要参考文章[百度地图API详解之地图坐标系统][百度地图API详解之地图坐标系统]。
百度地图定义了另一种坐标系，称为**百度平面坐标系**，百度的平面坐标可以与百度经纬度坐标直接转换，与当前的瓦片等级无关。
具体转换在百度地图JavaScript API中实现--[JavaScript API Class MercatorProjection][JavaScript API Class MercatorProjection]:
```
// 根据球面坐标获得平面坐标
lngLatToPoint(lngLat:Point) 
// 根据平面坐标获得球面坐标
pointToLngLat(point:Pixel)
```

### 纬度为负数（南纬）时候的转换错误
使用[百度地图API详解之地图坐标系统][百度地图API详解之地图坐标系统]中提供的测试样例：
```
lnglat = {lng: 116.404, lat: 39.915}
```
得到结果是正确的平面坐标：
```
{ pointX: 12958175, pointY: 4825923.77 }
```
然后保持经度不变，纬度取负值：
```
lnglat = {  lng: 116.404,   lat: -39.915}
```
得到的平面坐标为：
```
{ pointX: 12958175, pointY: -4823785.38 }
```
**怀疑是bug的情况：pointY 的绝对值并不相等。**但是也有可能是百度地图的平面坐标原点纬度方向并不在赤道上，所以两个值的绝对值不等。

对`{ pointX: 12958175, pointY: -4823785.38 }`再转换为经纬度坐标，结果为：
```
{ lng: 116.404, lat: -39.900206 }
```
**疑是bug的情况：转回经纬度后lat的值不等于初始值，而且误差是公里级别的。**而`{ pointX: 12958175, pointY: 4825923.77 }`转换回经纬度的结果是正确的。多次测试发现lng无论正负与pointX的转换都是正确的；lat为正数时，转换为pointY，再由pointY转换为lat的结果是相同的；lat为负数时，相互转换结果不匹配。**所以`pointToLngLat(point:Pixel)`方法应该没有问题，`lngLatToPoint(lngLat:Point) `方法有错误，并且是在lat为负数的情况时出错**。

## Bug分析
首先查看百度地图的[JavaScript API V2.0][JavaScript API V2.0]:
> http://api.map.baidu.com/getscript?v=2.0&ak=E4805d16520de693a3fe707cdc962045&t=20160503160001

相关代码为：
![百度API bug代码](/img/API中经纬度坐标转瓦片坐标bug/百度API bug代码.png "百度API bug代码")

格式化一下，[完整代码点这里][node-baidusdk.js]，主要代码为：
```
  Au: [75, 60, 45, 30, 15, 0],
  iG: [
    [-0.0015702102444, 111320.7020616939, 1704480524535203, -10338987376042340, 26112667856603880, -35149669176653700, 26595700718403920, -10725012454188240, 1800819912950474, 82.5],
    [8.277824516172526E-4, 111320.7020463578, 6.477955746671607E8, -4.082003173641316E9, 1.077490566351142E10, -1.517187553151559E10, 1.205306533862167E10, -5.124939663577472E9, 9.133119359512032E8, 67.5],
    [0.00337398766765, 111320.7020202162, 4481351.045890365, -2.339375119931662E7, 7.968221547186455E7, -1.159649932797253E8, 9.723671115602145E7, -4.366194633752821E7, 8477230.501135234, 52.5],
    [0.00220636496208, 111320.7020209128, 51751.86112841131, 3796837.749470245, 992013.7397791013, -1221952.21711287, 1340652.697009075, -620943.6990984312, 144416.9293806241, 37.5],
    [-3.441963504368392E-4, 111320.7020576856, 278.2353980772752, 2485758.690035394, 6070.750963243378, 54821.18345352118, 9540.606633304236, -2710.55326746645, 1405.483844121726, 22.5],
    [-3.218135878613132E-4, 111320.7020701615, 0.00369383431289, 823725.6402795718, 0.46104986909093, 2351.343141331292, 1.58060784298199, 8.77738589078284, 0.37238884252424, 7.45]
  ],

for (var d = 0; d < this.Au.length; d++)
  if (b.lat >= this.Au[d]) {
    c = this.iG[d];
    break
  }
if (!c)
  for (d = this.Au.length - 1; 0 <= d; d--)
    if (b.lat <= -this.Au[d]) {
      c = this.iG[d];
      break
    }
```

以上代码用于求pointY数值，方法是根据lat的数值，分段使用不同的参数：
* 当lat大于等于0时，从数组 [75, 60, 45, 30, 15, 0]顺序判断lat的范围，然后使用对应的参数。
* 当lat小于0时，从数组[75, 60, 45, 30, 15, 0]对数值取反并逆序判断lat的范围，然后使用对应的参数。

问题在于：当lat小于0时的第一个判断等价于`b.lat <= -this.Au[this.Au.length - 1]` --> `b.lat<=0`，该判断恒成立。一直使用`this.Au[this.Au.length - 1]`做为pointY计算参数。

修改代码为：
```
if (!c)
  for (d = 0; d < this.Au.length; d++)
    if (b.lat <= -this.Au[d]) {
      c = this.iG[d];
      break
    }
```
转换后结果为：
```
{ pointX: 12958175, pointY: -4825923.77 }
```
再转换为经纬度：
```
{ lng: 116.404, lat: -39.915 }
```
问题解决。

## 结论
应该是百度地图代码写错了。不是故意混淆或加密的问题。稍微做修改就可以了。

### BTY
计算参数`this.Au`二维数组中每行的最后一个数可以组成一个新数组：
```
[82.5, 67.5, 52.5, 37.5, 22.5, 7.45]
```
最中间两个数和为90，它们外边两个数和为90，最外边两个数和为89.95！不知到百度算法是怎样的，但是有点怀疑有问题。

## 参考文献
百度地图API详解之地图坐标系统
>http://www.cnblogs.com/jz1108/archive/2011/07/02/2095376.html
[百度地图API详解之地图坐标系统]:http://www.cnblogs.com/jz1108/archive/2011/07/02/2095376.html

JavaScript API Class MercatorProjection
>http://developer.baidu.com/map/reference/index.php?title=Class:%E5%9C%B0%E5%9B%BE%E7%B1%BB%E5%9E%8B%E7%B1%BB/MercatorProjection
[JavaScript API Class MercatorProjection]:http://developer.baidu.com/map/reference/index.php?title=Class:%E5%9C%B0%E5%9B%BE%E7%B1%BB%E5%9E%8B%E7%B1%BB/MercatorProjection

JavaScript API V2.0
>http://api.map.baidu.com/getscript?v=2.0&ak=E4805d16520de693a3fe707cdc962045&t=20160503160001
[JavaScript API V2.0]:http://api.map.baidu.com/getscript?v=2.0&ak=E4805d16520de693a3fe707cdc962045&t=20160503160001

node-baidusdk.js
>https://github.com/CntChen/tile-lnglat-transform/blob/master/src/node-baidusdk.js
[node-baidusdk.js]:https://github.com/CntChen/tile-lnglat-transform/blob/master/src/node-baidusdk.js

## 完


