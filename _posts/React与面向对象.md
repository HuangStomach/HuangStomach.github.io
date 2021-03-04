---
title: React与面向对象
date: 2018-06-10 13:21:23
tags: 
- React
- JavaScript
categories:
- JavaScript
---

## 开始

实际上官方提供给我们的教程已经非常清晰明确了[reactjs.org](https://reactjs.org/tutorial/tutorial.html#before-we-start)

当我们安装好之后，就会拥有一个`create-react-app`对应模板的react应用

结构如下:

``` shell
├── node_modules
├── package.json
├── public
├── README.md
├── src
└── yarn.lock

```

其中要注意的一点是，我们之后编写的代码不会直接被浏览器所识别，我们应尽量把所有需要`react-loader`为我们做转换的代码放置在`src`目录下

`src`目录结构:

``` shell
.
├── App.css
├── App.js
├── App.test.js
├── index.css
├── index.js
├── logo.svg
└── registerServiceWorker.js
```

这个目录中会有很多的js文件和css文件

`public`目录结构:

```
├── favicon.ico
├── index.html
└── manifest.json
```

我们可以看到`public`目录结构更加像我们所熟悉的静态`web`应用的样子，我们访问的也是响应的`index.html`:

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width，initial-scale=1，shrink-to-fit=no">
    <meta name="theme-color" content="#000000">
    <title>React App</title>
  </head>
  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>
    <div id="root"></div>
  </body>
</html>
```

我这里把注释删除掉了一些，实际上我们编写的代码会被替换到`<div id="root"></div>`中，我们可以在`src`目录下的`index.js`看到:

``` javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(<App />, document.getElementById('root'));
registerServiceWorker();
```

`ReactDOM`帮我们将`App.js`文件内的内容，渲染到了刚才所说的`index.html`中

我们继续打开生成的`App.js`中我们可以看到:

``` javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Welcome to React</h1>
        </header>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}

export default App;
```

这样的文件结构或者这样的用法被我们习惯的称为**组件**

## 组件

虽然对于基础web或后端开发者来说可能有些不熟悉，但实际上概念是想通的。不得不说借助于[ES6](https://babeljs.io/learn-es2015/)规范，使得js更加亲切了一些

以我写的最多的php为例:

``` javascript
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
```

实际上我们可以简单的理解为`require`引入资源，只不过react中不仅仅可以引入`.js`或者`.jsx`文件，也可以引入其他的静态资源。

下半部分可能会比较熟悉:

``` javascript
class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Welcome to React</h1>
        </header>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}

export default App;
```

典型的class的设计，诸多高级语言都会拥有的特性，使得面向对象思想也更加的清晰。

在这个文件中我们甚至可以看到继承关键字`extends`，不太相同的是我们实现的只有`render()`方法，我们稍后再说，实际上和诸多语言的class一样，组件本身实际上也是提供构造函数的，这里因为没有具体使用所以官方为我们省略掉了，我们可以自己添加上:


``` javascript
class App extends Component {
  constructor(props) {
    super(props);
  }
  
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h1 className="App-title">Welcome to React</h1>
        </header>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
      </div>
    );
  }
}

export default App;
```

这样我们就拥有了熟悉的构造函数，(其中的`props`参数之后再说)，实际上我个人觉得，每个组件我们都可以当做是一个类来理解。

## 属性

那么说到对象，不可或缺的就是内部的属性。react中提供了组件内部属性操作的相关api，譬如:

``` javascript
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 1,
    };
  }
  
  render() {
    return (
      <div className="App">
      </div>
    );
  }
}

export default App;
```

这里我先将`render()`内的多余代码删除了，可以看到我们在构造函数中为`this`赋予了新的属性`state`，官方文档中的解释就是:

> State is similar to props，but it is private and fully controlled by the component

`props`后面再谈，可以看到`state`实际上是被识别为`private`的，我们可以理解为对象内的私有变量/属性，同时使用起来也比较简单:

```javascript
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 1,
    };
  }
  
  render() {
    return (
      <div className="App" onClick={() => this.setState({value: 'X'})}>
        {this.state.value}
      </div>
    );
  }
}

export default App;
```

我们可以看到，我们可以世界在html中使用`this.state.value`进行私有变量的使用，很像php中的动态脚本，不过我们想要修改私有属性的话不能直接去进行赋值，而是需要调用`this.setState()`。

这种直接在html中写js代码的格式被称为**jsx**，我一般喜好将所有的使用jsx语法的文件后缀修改为`.jsx`

jsx语法中有许多要注意的地方，如上述代码中无法直接使用html的`class`属性，而是要使用`className`，同时注册元素方法的时候也较为不一样，这里为了简单的将方法内的`this`指向当前组件我们编写了一个箭头函数。

当有内部私有变量的时候，我们就会想象到另一个概念公共变量/属性，实际上就是`props`了

类的公共属性我们可以在实例化好类之后直接进行赋值，回到我们的`index.js`文件:

``` javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(<App />, document.getElementById('root'));
registerServiceWorker();
```

实际上当我们`import App from './App';`的时候，就会产生一个类似于实例化一样的操作，我们在这里会获取到一个`App`对象，但是我们不会像其他语言一样进行简单的赋值，而是使用html属性一样的方式:

``` javascript
ReactDOM.render(<App name='wa' />, document.getElementById('root'));
```

我们赋予App的name属性则会注入到组件内部的props中:

```javascript
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 1,
    };
  }
  
  render() {
    return (
      <div className="App" onClick={() => this.setState({value: 'X'})}>
        {this.state.value}
        <p>
          {this.props.name}
        </p>
      </div>
    );
  }
}

export default App;
```

我们也可以像普通的使用类一样的扩展我们的组件:

```javascript
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 1,
    };
  }
  
  anotherOne() {
    console.log('another one');
  }
  
  changeValue(val = 'x') {
    this.setState({value: val});
    this.anotherOne();
  }
  
  render() {
    return (
      <div className="App" onClick={() => this.changeValue()}>
        {this.props.name}
      </div>
    );
  }
}

export default App;
```

实际上无论是vue还是react，其中的设计思想都是平时比较常用的，理解了他们相通的地方，学习使用起来就会更为容易。