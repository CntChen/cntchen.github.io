## Ajax ContentType: x-www-form-urlencoded VS json
问题：发送Ajax请求最好的编码类型是什么？

## 背景
在公司开发的一个页面的[Ajax][Ajax]请求使用了`contentType:application/json`，被后台的同事要求用`x-www-form-urlencoded`,撕逼撕不过他，赶紧回来学学知识。

## 引入
`contentType`是指发送信息至服务器时的**内容编码类型**，`contentType`用于表明发送数据流的类型，服务器根据编码类型使用特定的解析方式，获取数据流中的数据。内容编码类型的作用，有点像本地文件的后缀。

## 常见的contentType
### x-www-form-urlencoded
这是Jquery/Zepto Ajax默认的提交类型。最简例子为：
```js
let userInfo = {
 name: 'CntChen',
 Info: 'Front-End',
}

$.ajax({
  url: 'https://github.com',
  type: 'POST',
  data: userInfo,
  success: (data) => {},
});
```
此时默认的提交的`contentType`为`application/x-www-form-urlencoded`,此时提交的数据`userInfo`将会格式化成：
```js
"name=CntChen&Info=Front-End"
```
HTML的`form`表单默认的提交编码类型也是`x-www-form-urlencoded`,可能这就是Jquery/Zepto等类库(**其实是Ajax：XmlHttpRequest**)，也默认使用`contentType:x-www-form-urlencoded`的原因，毕竟表单的历史比Ajax早多了。--*我猜的，待验证*


如果请求类型`type`是`GET`,格式化的字符串将直接拼接在url后发送到服务端；如果请求类型是POST，格式化的字符串将放在http的包体Form Data中发送。

### json
如果需要使用Json发送数据，最简例子为：
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
"{"name":"CntChen","Info":"Front-End"}"
```
### multipart/form-data

## References
+ Ajax
>http://css88.com/doc/zeptojs_api/#$.ajax
[Ajax]:http://css88.com/doc/zeptojs_api/#$.ajax

+ x-www-form-urlencoded VS json - Pros and Cons. And Vulns.
>http://homakov.blogspot.in/2012/06/x-www-form-urlencoded-vs-json-pros-and.html
[x-www-form-urlencoded VS json]:http://homakov.blogspot.in/2012/06/x-www-form-urlencoded-vs-json-pros-and.html

## END
