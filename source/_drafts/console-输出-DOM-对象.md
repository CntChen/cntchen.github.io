---
title: console 输出 DOM 对象
date: 2015-08-1 01:01:01
tags: JS DOM
---


## 背景 ##

前端开发调试的时候，经常需要查看[HTML DOM元素对象][HTML DOM元素对象]，DOM元素对象包含了HTML DOM的各种属性，例如：

* **位置**
`element.offsetHeight`
`element.offsetWidth`

* **事件绑定**
`element.onclick`
`element.onkeydown`

* **子孙DOM节点和祖先DOM节点**


* **文本**


* **样式**


总之炒鸡有用。

## 问题 ##

一般使用 `console` 控制台打出DOM元素对象。

然而，事情是这样子的：[在jsfiddle查看代码][在jsfiddle查看]


HTML：
```
<div id="messagecontent">
	<div class="message">
        <p>hello</p>
    </div>
</div>
```

JS：
```	
	var messageContent = document.getElementById('messagecontent');
	var messages = messageContent.childNodes;
    console.log(messageContent);
	console.log(messages);
	console.log(messages[0]);

```

在 chrome中

![alt text](./img/chrome_dom_console.png)



在IE中
![alt text](./img/chrome_dom_console.png)

## 方案 ##

把DOM对象封装成Array的形式

```
	var messageContent = document.getElementById('messagecontent');
	var messages = messageContent.childNodes;
    messages[0].style.color = 'red';
    console.log([messageContent]);
	console.log(messages);
	console.log(messages[0]);
	console.log([messages[0]]);
```




[HTML DOM元素对象]:http://www.w3schools.com/jsref/dom_obj_all.asp
[在jsfiddle查看]:http://jsfiddle.net/CntChen/k2a7heb1/4/