---
title: Ajax ContentType: x-www-form-urlencoded VS json
date: 2016-09-09 00:01:55
tags: HTTP AJAX JSON 编码 前端 
---

## Ajax ContentType: x-www-form-urlencoded VS json
问题：发送Ajax请求最好的编码类型是什么？

## 背景
在公司开发的一个页面的[Ajax][Ajax]请求使用了`contentType:application/json`，被后台的同事要求用`x-www-form-urlencoded`,撕逼撕不过他，赶紧回来学学知识。

## 引入
`contentType`是指http/https发送信息至服务器时的**内容编码类型**，`contentType`用于表明发送数据流的类型，服务器根据编码类型使用特定的解析方式，获取数据流中的数据。内容编码类型的作用，有点像本地文件的后缀名。

## 常见的contentType
#### x-www-form-urlencoded
这是Jquery/Zepto Ajax默认的提交类型。最简例子为：
```js
let userInfo = {
 name: 'CntChen',
 info: 'Front-End',
}

$.ajax({
  url: 'https://github.com',
  type: 'POST',
  data: userInfo,
  success: (data) => {},
});
```
此时默认的提交的`contentType`为`application/x-www-form-urlencoded`,此时提交的数据将会格式化成：
```js
name=CntChen&info=Front-End
```
HTML的`form`表单默认的提交编码类型也是`x-www-form-urlencoded`,可能这就是Jquery/Zepto等类库(**其实是Ajax：XmlHttpRequest**)也默认使用`contentType:x-www-form-urlencoded`的原因，毕竟表单的历史比Ajax早多了。--*我猜的，待验证*


如果请求类型`type`是`GET`,格式化的字符串将直接拼接在url后发送到服务端；如果请求类型是POST，格式化的字符串将放在http body的Form Data中发送。

#### json
使用Json内容编码发送数据，最简例子为：
```js
let userInfo = {
 name: 'CntChen',
 Info: 'Front-End',
}

$.ajax({
  url: 'https://github.com',
  contentType: 'application/json',
  type: 'POST',
  data: JSON.stringify(userInfo),
  success: (data) => {},
});
```
最主要的不同有2点：
* 需要**显式指定**`contentType`**为**`application/json`，覆盖默认的contentType
* 需要**使用**`JSON.stringify`**序列化**需要提交的数据对象，序列化的结果为：
```js
{"name":"CntChen","info":"Front-End"}
```

### multipart/form-data
>When you are writing client-side code, all you need to know is use multipart/form-data when your form includes any < input type="file" > elements.
-- [multipart/form-data][multipart/form-data]

## JS对象编码
对于扁平的参数对象，使用`x-www-form-urlencoded`或`json`并没有大的差别，后台都可以处理成对象，并且提交的数据量差别不大。
但是对于**对象中嵌套对象**，或**对象字段包含数组**，此时两种内容编码方式就有较大差别。
#### 对象嵌套
```js
{
    userInfo :{
     name: 'CntChen',
     info: 'Front-End',
     login: true,
    },
}
```

* to x-www-form-urlencoded `(1)`
```js
userInfo[name]=CntChen&userInfo[info]=Front-End&userInfo[login]=true
```

* to json `(2)`
```js
{"userInfo":{"name":"CntChen","Info":"Front-End","login":true}}
```

#### 对象字段为数组
```js
{
    authors:[
      {
        name: 'CntChen',
        info: 'Front-End',
      },
      {
        name: 'Eva',
        info: 'Banker',
      }
    ],
}
```

* to x-www-form-urlencoded `(3)`
```js
authors[0][name]=CntChen&authors[0][info]=Front-End&authors[1][name]=Eva&authors[1][info]=Banker
```

* to json `(4)`
```js
{"authors":[{"name":"CntChen","info":"Front-End"},{"name":"Eva","info":"Banker"}]}
```

可见：`x-www-form-urlencoded`是先将对象**铺平**，然后使用`key=value`的方式，用`&`作为间隔。对于嵌套对象的每个字段，都要传输其前缀，如`(1)`中的`userInfo`重复传输了3次;`(3)`中authors传输了4次。
如果对象是多重嵌套的，或者嵌套对象的字段较多，`x-www-form-urlencoded`会产生更多冗余信息。同时，`x-www-form-urlencoded`可读性不如`json`字符串。

## 回答问题：json最好
#### 较小的传输量
从前文可以看出，使用json字符串的形式，可以减少冗余字段的传输，减少请求的数据量。
>补充：可能你会觉得`(4)`中数组内的`name`和`info`也传输了多次，是不是也存在冗余？其实这不是冗余。因为对数组中的各对象，并不要求其具有相同的字段（数组中的对象并不是结构化的），所以不能忽略“相同”的字段名。使用`x-www-form-urlencoded`编码方式，数组内对象的字段也是重复传输。

#### 请求与返回统一
目前许多前后端交互的返回数据是json字符串，这可能是考虑**较小的传输量**而作出的选择。同时，ES3.1添加了[JSON对象][es3.1:json_support]，许多浏览器可以[直接使用JSON对象][Can I use JSON]，可以将json字符串解析为对象(`JSON.parse`),将js对象编码为js字符串（`JSON.stringify`）;
所以使用json编码请求数据，其编码解码非常方便，并且可以保持与后台返回数据的格式一致。
**一致是一件很美好的事情。**

#### 可读性高
可读性高是json格式[自带buff][JSON]。

## 结论
赶紧使用`contentType=applications/json`。

## References
+ Ajax
>http://css88.com/doc/zeptojs_api/#$.ajax
[Ajax]:http://css88.com/doc/zeptojs_api/#$.ajax

+ x-www-form-urlencoded VS json - Pros and Cons. And Vulns.
>http://homakov.blogspot.in/2012/06/x-www-form-urlencoded-vs-json-pros-and.html
[x-www-form-urlencoded VS json]:http://homakov.blogspot.in/2012/06/x-www-form-urlencoded-vs-json-pros-and.html

+ What does enctype='multipart/form-data' mean?
>http://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean
[multipart/form-data]:http://stackoverflow.com/questions/4526273/what-does-enctype-multipart-form-data-mean

[es3.1:json_support]:http://wiki.ecmascript.org/doku.php?id=es3.1:json_support

+ Can I use JSON
>http://caniuse.com/#search=JSON
[Can I use JSON]:http://caniuse.com/#search=JSON

+ JSON
>http://www.json.org/
[JSON]:http://www.json.org/

## END