title: 基于Cookie的搜索历史提示
tags: Front-end
---
## 背景 ##

在浏览器的地址栏中输入文字，可以看到有个下拉的选择栏，提供一些用户访问的网址。在公司实习参与的项目，前端页面也需要使用该功能，所以开发了一个基于**Cookie**的搜索历史提示。这里简单介绍一下实现方法。

## 实现方法 ##



## 要点 ##

### 1. Cookie操作 ###

### 2. 键盘与鼠标交互 ###

### 3.  ###

## 看坑 ##

### 1. 过期的 `escape()` `unescape()`

**JavaScript** 自带了**字符串编码和解码**的函数,这些函数属于[**JavaScript 全局对象**][JavaScript 全局对象]。

**W3school**上的[Cookie操作方法][w3s_Cookie操作方法]代码使用了过期的 `escape()` 和 `unescape`。(ECMAScript v3 开始过期)

应该使用

	encodeURI() / decodeURI()

或者

	encodeURIComponent() / decodeURIComponent() 

我使用的是：`encodeURIComponent()` `decodeURIComponent() `

关于两者的不同，可以查看 stackoverflow上的讨论： [Best practice: escape, or encodeURI / encodeURIComponent][About escape]


### 2. 在历史记录中，如果含有[HTML转义字符][HTML转义字符]，则需要做反转义处理

*例如：*

HTML 可见看到的一条历史记录

	// 用户可见字段     
	search a > 100
	
	// 真实的HTML字段
    <a id="history">search a &gt; 100</a>

使用  `document.getElementById('history').innerHTML` 得到的是 `search a &gt; 100`，而真正的搜索历史是 `search a > 100`。这时候需要反转义。

其实方法很简单，`search a &gt; 100`字段在 `document.getElementById('history')` **DOM**对象中就存在：

    document.getElementById('history').innerHMTL // search a &gt; 100
	document.getElementById('history').innerText  // search a > 100
	document.getElementById('history').textContent // search a > 100

可以参考[HTML反转义方法][HTML反转义]

## 惊喜 ##

## 结语 ##



[JavaScript 全局对象]:http://www.w3school.com.cn/jsref/jsref_obj_global.asp

[w3s_Cookie操作方法]:http://www.w3school.com.cn/js/js_cookies.asp

[About escape]:http://stackoverflow.com/questions/75980/best-practice-escape-or-encodeuri-encodeuricomponent

[HTML转义字符]:http://tool.oschina.net/commons?type=2

[HTML反转义]:http://www.ccydesign.com/htmlencode/