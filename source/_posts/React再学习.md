---
title: React再学习
date: 2016-06-14 00:45:15
tags: React ES6 前端
---

## 背景
  2015年暑假实习的时候实践了一小段时间的[React][React],**那时候还是用ES5写的**，对React的整个架构和基础方法理解并不深入。2016年5月，希望可以在项目中学习,**并使用ES6编写React**。


## 引入

主要参考文档:[React Docmument][React Docmument]


## 要点

* HTML tag小写，React Component 首字母大写

 >React's JSX uses the upper vs. lower case convention to distinguish between local component classes and HTML tags.


* HTML的类名`class`和`for`

 >Since class and for are reserved words in JavaScript, the JSX elements for built-inDOM nodes should use the attribute names className and htmlFor respectively, (eg. (div className="foo" /) ). Custom elements should use class and for directly (eg./ (my-tag class="foo" /) ).



* props vs states

  **props --> data from parent to child**

  **states --> for interactivity**


* attributes

 HTML tag 的开发者自定义attribute需要使用`data-`前缀，否则不渲染该attribute

 React Component则没问题



* PropTypes

  为了保证复用的模块可以被正确地复用，可以[规定接口(Props)的类型][规定接口(Props)的类型]。



* 传递属性

 [Rest and Spread Properties][Rest and Spread Properties]

 [Destructuring assignment][Destructuring assignment]



* [Component Lifecycle][Component Lifecycle]



## 使用ES6编写React



### 纯ES6的React写法

主要参考：[es6-classes][es6-classes]

* class

```

class HelloMessage extends React.Component {

  render() {

    return <div>Hello {this.props.name}</div>;

  }

}

```

* ` propTypes` `defaultProps`要写在类定义外面！！！

```

export class Counter extends React.Component {

  constructor(props) {

    super(props);

    this.state = {count: props.initialCount};

    this.tick = this.tick.bind(this);

  }

  tick() {

    this.setState({count: this.state.count + 1});

  }

  render() {

    return (

      <div onClick={this.tick}>

        Clicks: {this.state.count}

      </div>

    );

  }

}

Counter.propTypes = { initialCount: React.PropTypes.number };

Counter.defaultProps = { initialCount: 0 };

```



* 没有自动绑定

>Methods follow the same semantics as regular ES6 classes, meaning that they don't automatically bind `this` to the instance.



### 主要的ES6+ React语法

主要参考：[React on ES6+][React on ES6+]

* Component class定义



```

// The ES5 way 

var Photo = React.createClass({ 

    handleDoubleTap: function(e) { … }, 

  render: function() { … }, 

}); 



// The ES6+ way 

class Photo extends React.Component {  

   handleDoubleTap(e) { … } 

   render() { … }

}

```



* 属性初始化（Property initializers）



```


// The ES5 way

var Video = React.createClass({

  getDefaultProps: function() {

    return {

      autoPlay: false,

      maxLoops: 10,

    };

  },

  getInitialState: function() {

    return {

      loopsRemaining: this.props.maxLoops,

    };

  },

  propTypes: {

    autoPlay: React.PropTypes.bool.isRequired,

    maxLoops: React.PropTypes.number.isRequired,

    posterFrameSrc: React.PropTypes.string.isRequired,

    videoSrc: React.PropTypes.string.isRequired,

  },

});



// The ES6+ way

class Video extends React.Component {

  static defaultProps = {

    autoPlay: false,

    maxLoops: 10,

  }

  static propTypes = {

    autoPlay: React.PropTypes.bool.isRequired,

    maxLoops: React.PropTypes.number.isRequired,

    posterFrameSrc: React.PropTypes.string.isRequired,

    videoSrc: React.PropTypes.string.isRequired,

  }

  state = {

    loopsRemaining: this.props.maxLoops,

  }

}

```



* 箭头函数（Arrow function）



```

// ES6 way


// Manually bind, wherever you need to

class PostInfo extends React.Component {

  constructor(props) {

    super(props);

    // Manually bind this method to the component instance...

    this.handleOptionsButtonClick = this.handleOptionsButtonClick.bind(this);

  }

  handleOptionsButtonClick(e) {

    // ...to ensure that 'this' refers to the component instance here.

    this.setState({showOptionsModal: true});

  }

}



// 推荐，ES6+way，需要使用babel-preset-stage-0，实现`this`绑定

class PostInfo extends React.Component {

  handleOptionsButtonClick = (e) => {

    this.setState({showOptionsModal: true});

  }

}

```



* Dynamic property names & template strings



* Destructuring & spread attributes

 

### 使用babel转译（Transpiling）

使用ES6编写react代码，还是需要使用babel转译后才能在浏览器中使用。

使用babel转译ES6 React需要三个babel预设模块：

```

babel-preset-es2015 此预设包含了所有的 es2015 插件

babel-preset-react 此预设包含了所有的 React 插件

babel-preset-stage-0 此预设包含了 stage 0 中的所有插件

```

### 看坑

* 需要`babel-preset-stage-0`模块

因为**ES6+的React语法有一些是超出ES6规范的**，比如` ES7 property initializers`和`static`关键字。



### 拓展

* JavaScript规范的阶段

如果你对ES6/ES7和各种stage是如何界定的，可以查看这个知乎回答：[如何评价 ECMAScript 2016（ES7）只新增2个特性][如何评价 ECMAScript 2016（ES7）只新增2个特性]


* 检查代码所处阶段

推荐一个工具，可以查看你的代码是处于哪个阶段：[babel repl][babel repl]

使用方法：贴入代码然后选择代码预设插件，如果没有报错，则该插件可以转译贴入的代码。



转译没有出错
![转译没有出错](/img/React再学习/转译没有出错.png "转译没有出错")


转译出错
![转译出错](/img/React再学习/转译出错.png "转译出错")


## 参考资料

React

> https://github.com/facebook/react



[React]:https://github.com/facebook/react



规定接口(Props)的类型

>https://facebook.github.io/react/docs/reusable-components.html



[规定接口(Props)的类型]:https://facebook.github.io/react/docs/reusable-components.html



[Rest and Spread Properties]:https://facebook.github.io/react/docs/transferring-props.html#rest-and-spread-properties-...



[Destructuring assignment]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment



[Component Lifecycle]:https://facebook.github.io/react/docs/working-with-the-browser.html#component-lifecycle



React Docmument

> http://facebook.github.io/react/docs/getting-started.html



[React Docmument]:http://facebook.github.io/react/docs/getting-started.html



es6-classes

>https://facebook.github.io/react/docs/reusable-components.html#es6-classes



[es6-classes]:https://facebook.github.io/react/docs/reusable-components.html#es6-classes



React on ES6+

> https://babeljs.io/blog/2015/06/07/react-on-es6-plus



[React on ES6+]:https://babeljs.io/blog/2015/06/07/react-on-es6-plus



如何评价 ECMAScript 2016（ES7）只新增2个特性

>https://www.zhihu.com/question/39993685/answer/84166978



[如何评价 ECMAScript 2016（ES7）只新增2个特性]:https://www.zhihu.com/question/39993685/answer/84166978



 babel repl

>https://babeljs.io/repl/



[babel repl]:https://babeljs.io/repl/



## 完



