---
title: 写博客使用MathJax
date: 2016-05-09 18:47:59
tags: MathJax 博客 教程
---
## 背景
在写博客时有时候需要插入公式。平时主要使用wiz写作，然后在hexo、简书中发布。

## wiz
wiz（为知笔记）支持markdown语法，只要添加文章后缀为`.md`，此时是不支持编写公式的。
要同时支持markdown和MathJax公式，添加文章后缀为`.mdp`。

如果你在wiz中，你可以看到下面的公式：
$$test={hello}\times{world}$$

## hexo网站
hexo网站默认不支持MathJax，可以通过hexo插件实现--[hexo-math][hexo-math]。
通过插件来polyfill的原理是hexo发布时可以插入`script`标签，所以就在页面渲染的时候引入了公式渲染器。
使用方法：
```
npm install hexo-math --save
```
然后直接就可以使用。[hexo-math][hexo-math]的其他配置请自行查看。

在hexo中如果安装了hexo-math，应该可以看到以下公式：
$$test={hello}\times{world}$$

## 简书
简书支持markdown，不支持MathJax，可以使用图片的方式来hack。
其原理是markdown的图片可以通过指定url从网络上引用，然后在url中填入公式作为请求参数，公式生成服务器根据请求参数渲染出图片，然后返回。
可以参考[简书中编辑数学公式][简书中编辑数学公式]

可用的在线渲染器：
>http://latex.codecogs.com/svg.latex?

在`?`后面直接填入公式内容，不需要`$`或`$$`。

在简书中，你应该可以看到以下公式：
![](http://latex.codecogs.com/svg.latex?test={hello}\times{world})

该公式是一张图片。

## 其他平台的支持情况
* github不支持

## 参考资料
hexo-math
>https://github.com/akfish/hexo-math
[hexo-math]:https://github.com/akfish/hexo-math

简书中编辑数学公式
>http://blog.szrf215.com/p/e8a14ec1c614
[简书中编辑数学公式]:http://blog.szrf215.com/p/e8a14ec1c614

## 完

