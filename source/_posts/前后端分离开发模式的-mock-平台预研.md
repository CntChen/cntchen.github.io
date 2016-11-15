---
title: 前后端分离开发模式的 mock 平台预研
date: 2016-11-15 23:53:30
tags: 前后端分离 mock AJAX 工作流 前端
---
[原文地址](https://github.com/CntChen/cntchen.github.io/issues/1)

## 引入
*mock（模拟）: 是在项目测试中，对项目外部或不容易获取的对象/接口，**用一个虚拟的对象/接口来模拟**，以便测试。*


## 背景
### 前后端分离
* 前后端仅仅通过异步接口(AJAX/JSONP)来编程
* 前后端都各自有自己的开发流程，构建工具，测试集合
* 关注点分离，前后端变得相对独立并松耦合

![前后端分离.png](/img/前后端分离开发模式的mock平台预研/前后端分离_new.png)

### 开发流程
* 后台编写和维护接口文档，在 API 变化时更新接口文档
* 后台根据接口文档进行接口开发
* 前端根据接口文档进行开发
* 开发完成后联调和提交测试

![开发流程.png](/img/前后端分离开发模式的mock平台预研/开发流程.png)

### 面临问题
* 没有统一的文档编写规范，导致文档越来越乱，无法维护和阅读
* 开发中的接口增删改等变动，需要较大的沟通成本
* **对于一个新需求，前端开发的接口调用和自测依赖后台开发完毕**
* **将接口的风险后置，耽误项目时间**

### 解决方法
* 接口文档服务器 -- 解决接口文档编辑和维护的问题
* mock 数据 -- 解决前端开发依赖真实后台接口的问题

## 接口文档服务器
### 功能
#### 接口编辑功能
* 类型1：根据接口描述语法书写接口，并保存为文本文件，然后使用生成工具生成在线接文档（HTML）
  -- 也有一些类似 Markdown 的接口文档编辑器，参见：[There Are Four API Design Editors To Choose From Now][There Are Four API Design Editors To Choose From Now]。
![接口书写转换为接口文档.png](/img/前后端分离开发模式的mock平台预研/接口书写转换为接口文档.png)

* 类型2：提供在线的接口编辑平台，进行可交互的接口编辑
![接口文档服务器.png](/img/前后端分离开发模式的mock平台预研/接口文档服务器.png)

#### 接口查看功能
* 提供友好的接口文档查看功能

### 用法
* 后台开发人员进行接口文档编写
  -- 定义接口路径、接口上传字段、接口返回字段、字段含义、字段类型、字段取值
* 前端开发人员查看接口文档

### 优点
* 统一管理和维护接口文档
  -- 提供了接口导入、接口模块化、接口版本化、可视化编辑等功能
* 接口文档规范，可读性强，**减少前后端接口沟通成本**

## 前端 mock 方法回顾
前端开发过程中，使用 mock 数据来模拟接口的返回，对开发的代码进行业务逻辑测试。解决开发过程中对后台接口的依赖。

### 硬编码数据
将 mock 数据写在代码中。
#### 示例
```js
// $.ajax({
//   url: ‘https://cntchen.github.io/userInfo’,
//   type: 'GET',
//   success: function(dt) {
    var dt = {
      "isSuccess": true,
      "errMsg": "This is error.",
      "data": {
        "userName": "Cntchen",
        "about": "FE"
      },
    };
    if (dt.isSuccess) {
      render(dt.data);
    } else {
      console.log(dt.errMsg);
    }
//   },
//   fail: function() {}
// });
```

#### 优点
* 可以快速修改测试数据

#### 痛点
* **无法模拟异步的网络请求，无法测试网络异常**
* 肮代码，联调前需要做较多改动，**增加最终上真实环境的切换成本**
  -- 添加网络请求，修改接口、添加错误控制逻辑
* 接口文档变化需要手动更新

### 请求拦截 & mock 数据
hijack（劫持）接口的网络请求，将请求的返回替换为代码中的 mock 数据。
#### 实例
[jquery-mockjax][jquery-mockjax]
> The jQuery Mockjax Plugin provides a simple and extremely flexible interface for mocking or simulating ajax requests and responses

#### 优点
* 可以模拟异步的网络请求
* 可以快速修改测试数据

#### 痛点
* 依赖特定的框架，如`Jquery`
* 增加最终上真实环境的切换成本
* 接口文档变换需要手动更新

### 本地 mock 服务器
将 mock 数据保存为本地文件。在前端调试的构建流中，用 node 开本地 mock 服务器，请求接口指向本地 mock 服务器，本地 mock 服务器 response mock 文件。
#### mock 文件
```
.mock
├── userInfo.json
├── userStars.json
├── blogs.json
└── following.json
```
#### 接口调用
`https://github.com/CntChen/userInfo` --> `localhost:port/userInfo`

#### 优点
* 对代码改动较小，联调测试只需要改动接口 url
* 可以快速修改测试数据

#### 痛点
* json 文件非常多
* 接口文档变化需要手动更新

### 代理服务器
* 使用 [charles][charles] 或 [fiddler][fiddler] 作为代理服务器
* 使用代理服务器的 map（映射）& rewrite（重写） 功能

#### map local
* 接口请求的返回映射为本地 mock 数据
 `https://github.com/CntChen/userInfo` --> `localPath/userInfo`

![map local.png](/img/前后端分离开发模式的mock平台预研/map_local_public.png)

* 编辑映射规则
![map rule.png](/img/前后端分离开发模式的mock平台预研/map_rule_public.png)

#### map remote
* 接口请求的返回映射为另一个远程接口的调用
![map remote.png](/img/前后端分离开发模式的mock平台预研/map_remote.png)

#### rewrite
* 修改接口调用的 request 或 response，添加/删除/修改 HTTP request line/response line/headers/body
![rewrite data.png](/img/前后端分离开发模式的mock平台预研/rewrite_data_public.png)

* 解决跨域问题
使用 map 后，接口调用的 response 不带 CORS headers，跨域请求在浏览器端会报错。需要重写接口返回的 header，添加 CORS 的字段。
![rewrite cors.png](/img/前后端分离开发模式的mock平台预研/rewrite_cors_public.png)

#### 优点
* 前端直接请求真实接口，无需修改代码
* 可以修改接口返回数据

#### 痛点
* **需要处理跨域问题**
* **一个变更需要代理服务器进行多处改动，配置效率低下**
* 不支持 HTTP method 的区分
  -- CORS 的 preflight 请求(OPTION)也会返回数据
* 需要有远程接口或本地 mock 文件，与使用本地 mock 文件相同的痛点

### mock 平台
#### 接口文档服务器
使用接口文档服务器来定义接口数据结构

![接口服务器.jpg](/img/前后端分离开发模式的mock平台预研/接口服务器_new.png)

#### mock服务器
**mock 服务器根据接口文档自动生成 mock 数据**，实现了接口文档即API

![mock服务器.jpg](/img/前后端分离开发模式的mock平台预研/mock服务器_new.png)

#### 优点
* 接口文档自动生成和更新 mock 数据
* 前端代码联调时改动小

#### 缺点
* **可能**存在跨域问题

## 业界实践
### 公司实践
没有找到公司级别的框架，除了阿里的 [RAP][RAP]。可能原因：
* 非关键性、开创性技术，没有太多研究价值
* 许多大公司是小团队作战，没有统一的 mock 平台
* 已经有一些稳定的接口，并不存在后台接口没有开发完成的问题
  -- 而我们想探究的问题是：前后端**同时开发时**的 mock

### github 开源库
* [faker.js][faker.js]
随机生成固定字段的 mock 数据,如`email`，`date`，`images`等，支持国际化。

* [blueprint][blueprint]
> A powerful high-level API design language for web APIs.

  一种使用类markdown语法的接口编写语言，使用[json-schema][json-schema]和[mson][mson]作为接口字段描述。有[完善的工具链](https://apiblueprint.org/tools.html)进行接口文件 Edit，Test，Mock，Parse，Converter等。

* [swagger][swagger]
> Swagger是一种 Rest API 的简单但强大的表示方式，标准的，语言无关，这种表示方式不但人可读，而且机器可读。可以作为 Rest API 的交互式文档，也可以作为 Rest API 的形式化的接口描述，生成客户端和服务端的代码。 --[Swagger：Rest API的描述语言][Swagger：Rest API的描述语言]

  定义了一套接口文档编写语法，然后可以自动生成接口文档。相关项目: [Swagger Editor][Swagger Editor] ，用于编写 API 文档。[Swagger UI][Swagger ui] restful 接口文档在线自动生成与功能测试软件。[点击查看Swagger-UI在线示例][Swagger-UI在线示例]。

* [wireMock][wireMock]
> WireMock is a simulator for HTTP-based APIs. Some might consider it a service virtualization tool or a mock server. It **supports testing of edge cases and failure modes** that the real API won't reliably produce.

### 商业化方案
* [apiary][apiary]
商业化方案，[blueprint][blueprint]开源项目的创造者。界面化，提供mock功能，生成各编程语言的调用代码(跟 postman 的 generate code snippets类似)。

### 其他实践
[API Evangelist(API 布道者)][apievangelist]

## 总结
对于前后端分离开发方式，已经有比较成熟的 mock 平台，主要解决了2个问题：
* 接口文档的编辑和维护
* 接口 mock 数据的自动生成和更新

## 后记
预研时间比较有限，有一些新的 mock 模式或优秀的 mock 平台没有覆盖到，欢迎补充。
笔者所在公司选用的平台是 [RAP][RAP]，后续会整理一篇 RAP 实践方面的文章。
问题来了：你开发中的 mock 方式是什么？

## References
* 图解基于node.js实现前后端分离
>http://yalishizhude.github.io/2016/04/19/front-back-separation/
[图解基于node.js实现前后端分离]:http://yalishizhude.github.io/2016/04/19/front-back-separation/

* TestDouble(介绍 mock 相关的概念)
> http://martinfowler.com/bliki/TestDouble.html
[mock 相关的概念]:http://martinfowler.com/bliki/TestDouble.html

* There Are Four API Design Editors To Choose From Now
> https://apievangelist.com/2014/11/21/there-are-four-api-design-editors-to-choose-from-now/
[There Are Four API Design Editors To Choose From Now]:https://apievangelist.com/2014/11/21/there-are-four-api-design-editors-to-choose-from-now/

* 联调之痛--契约测试
> http://www.ituring.com.cn/article/42460
[联调之痛]:http://www.ituring.com.cn/article/42460

* Swagger：Rest API的描述语言
> https://zhuanlan.zhihu.com/p/21353795
[Swagger：Rest API的描述语言]:https://zhuanlan.zhihu.com/p/21353795

* Swagger - 前后端分离后的契约
>http://www.cnblogs.com/whitewolf/p/4686154.html
[Swagger - 前后端分离后的契约]:http://www.cnblogs.com/whitewolf/p/4686154.html

* Swagger UI教程 API 文档神器 搭配Node使用
> http://www.jianshu.com/p/d6626e6bd72c#
[Swagger UI教程 API 文档神器 搭配Node使用]:http://www.jianshu.com/p/d6626e6bd72c#

[Silent Links]:#
[jquery-mockjax]:https://github.com/jakerella/jquery-mockjax
[charles]:http://www.charlesproxy.com/
[fiddler]:http://www.telerik.com/fiddler
[RAP]:https://github.com/thx/RAP
[faker.js]:https://github.com/marak/Faker.js/
[blueprint]:https://apiblueprint.org/
[swagger]:http://swagger.io/
[Swagger Editor]:https://github.com/swagger-api/swagger-editor
[Swagger ui]:https://github.com/swagger-api/swagger-ui
[wireMock]:http://wiremock.org/
[Swagger-UI在线示例]:http://petstore.swagger.io/#!/pet/addPet
[apiary]:https://apiary.io/
[apievangelist]:https://apievangelist.com/

## END
---