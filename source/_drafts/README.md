# 前端进度 #

**请不要看这个文档，已经停止更新**

## 已经完成 ##

* 页面

  * **set source page** 页面

    1. 基于**Cookie**的搜索历史记录

    2. 基于window.history的浏览器历史回退

  * **manager task page** 任务管理页面

  * **dashboard**

    1. 基于[FreeWall][FreeWall]的响应式拖拽布局

    2. 多开图形编辑

  * 帮助页面

* 功能 

* **提示 confirm**插件

* 弹出按钮 **action**

* **Chart.js** 画图插件

  1. 支持其中默认图形

  2. 支持编写其他插件

* 日期选择插件 **pikaday**

* **search page** 的时间选择器

  1. 时间描述定义**timepicker**

  2. 时间描述字符处理

*  简化静态文字的国际化显示

	使用 CSS Class： **i18n** 然后在 Jquery 中拾取

* **CSS** 简单模块化 **common.css**

* **JS** 模块化，代码分离完成，没有使用 **[AMD][AMD] or [CMD][CMD]**

* 修改 **UiHelper.drawerMgr** 使抽屉可以在外部打开和关闭

* **source** 提交的后台 **mock**


## 需要与后台确认功能 -- set source page ##

* 服务器提示信息的国际化（前端做还是后台完成）

* 测试提交出错的提示和逻辑是否正确

* ~~提交URL和前台数据更新~~


## 未完成功能 ##

* 单页应用 **splunk 是单页应用** 使用JS实现局部刷新

* 页面刷新慢 参考 react 或 使用文本刷新法？

* 查询时间交互使用插件方便用户输入

* 添加代码注释和主要修改说明

[AMD]:http://www.requirejs.org/
[CMD]:http://seajs.org/docs/
[FreeWall]:https://github.com/kombai/freewall