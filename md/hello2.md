---
title: React 牵手 Express
---

仅仅有了 API ，功能还不完全。需要有前端来调用 API ，完成整个小功能。

在我们《JS 独孤求败》课程体系里面，前端代码我们都是用 React 来写，
所以前端和后端都是用 JS 来写。但是这里要提醒大家的是，前端和后端是**完全独立**的两个项目，通过 API 来作为桥梁。其实，我们后端虽然是 Nodejs 写的，但是用 PHP/JAVA 也能写。

### 前端项目准备

前面的课程中，我们已经学习了 React 开发。那么先在就来写一个完全跟后台无关的 React 的 Hello World ，要求：

- 页面最终显示出来的，就是你的用户名，例如 happypeter
- 要用到 react 的 state ，constructor ，生命周期
- 用 ES6/Babel/Webpack

小贴士：创建 React 应用的脚手架项目
可以推荐的是 https://github.com/facebookincubator/create-react-app
但是，我们需要对 Webpack 底层做一些了解，所以暂时不推荐使用
脚手架
小贴士结束


所有代码都放到 react-frontend 这个文件夹中，代码如下：


src/index.js

```js
import React, { Component } from 'react';
import ReactDOM from 'react-dom';


class App extends Component {
  constructor(){
    super();
    this.state = {
      username: ''
    };
  }
  componentWillMount() {
    this.setState({username: 'happypeter'});
  }
  render(){
    return(
      <div>
        {this.state.username}
      </div>
    )
  }
}

ReactDOM.render(<App/>,document.getElementById('app'));
```

package.json

```json
{
  "name": "react-frontend",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "babel-core": "^6.23.1",
    "babel-loader": "^6.4.0",
    "babel-preset-env": "^1.2.1",
    "babel-preset-react": "^6.23.0",
    "react": "^15.4.2",
    "react-dom": "^15.4.2",
    "webpack": "^2.2.1"
  }
}
```

webpack.config.js

```js
var path = require('path');

module.exports = {
  entry: path.resolve(__dirname, 'src/index.js'),
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader'
      }
    ]
  }
};
```

.babelrc

```
{
  "presets": ["env", "react"]
}
```

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>

  <div id="app"></div>
  <script src="./build/bundle.js" charset="utf-8"></script>
</body>
</html>
```

### 安装 http-server

前端代码其实最好也跑在一个 http 服务器之上，最简单的方法是：

```
npm install -g http-server
```

这样，我们就有了一个简单的系统工具叫 http-server

```
cd react-frontend/
http-server .
```

就能把我们的前端项目启动到一个 http 服务器上了。

FIXME：有时候会出现 bundle.js 更新了，但是刷新页面看不到效果的情况，应该是有缓存相关的问题。

### 为何 state 要设置两次

上面代码都是 React 的基础，我们不在重复，唯一可能感觉奇怪的是
为何 constructor 中设置了一个 username 的初始值，然后又在
生命周期函数 componentWillMount 中对 username 进行了覆盖。

为何要这么麻烦呢？这个后面结合 axios 向后台请求数据的代码，就会
比较容易看出作用了。

下面来看如何发请求到服务器端。



### 实现 API

开发一个功能，好的做法是，先修改后端代码，也就是先实现 API ，然后下一步就是用 curl 这样的命令，来测试一下 API ，发现 API 没毛病，然后再动手去写前端代码。

打开 express-backend 中的 index.js 代码，添加下面这个 API

```js
app.get('/username', function(req, res){
  res.send({username: 'happypeter'});
})
```

### 用 curl 来测试 API

curl 是一个安装在系统上的命令，可以用来发 http 请求，最适合用来测试 API 。

动手写前端代码之前，如果用 curl 测试一下 API ，会让写前端代码的时候，我们心里更踏实。

```
curl -X GET 'http://localhost:3000/username'
```

如果后端没有问题，应该可以看到下面的输出

```json
{username: 'happypeter'}
```

这样，后端 API 测试通过。

另注：如果执行出现如下错误：

```
$ curl -X GET localhost:3000/username
curl: (7) Failed to connect to localhost port 3000: Connection refused
```

翻译：连接 localhost:3000 失败，连接被拒绝。

错误的原因：后端没有启动。


### 安装 axios 发送 http 请求

现在我们来到前端代码中。引入 axios 。axios 按照[官网](https://github.com/mzabriskie/axios)的说法，它是一个 `http client` （ http 的客户端），换句话说，它是专门用来发 http 请求的。

axios 是常用的发 http 请求的工具（现在一般不提发 ajax 请求这个说法了）。

首先来进行装包：

```
npm install --save axios
```

把 axios 安装到 react-frontend 这个项目中。

装包之后，就可以到 src/index.js 中去使用了，代码如下

```js
import axios from 'axios';
```

我们当前的请求不希望是通过按钮来触发，而是希望，页面加载的时候，自动发出
http 请求，向服务要数据，所以，代码非常适合写到生命周期函数中：

```js
componentWillMount() {
  axios.get('http://localhost:3000/username').then(function(response){
      return console.log(response);
    }
  )
}
```


### 跨域请求 Access-Control-Allow-Origin

代码进行到上面，浏览器中用前台请求后台，会在 chrome console 中看到，如下
错误：

```
XMLHttpRequest cannot load http://localhost:3000/. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'null' is therefore not allowed access.
```

`XMLHttpRequest` 是发 HTTP 请求的底层机制，是浏览器自带功能。上面的报错翻译如下：

>无法加载后台 http://localhost:3000 . 被请求的资源中没有设置
> Access-Control-Allow-Origin 头部。源头设置为 Null ，所以不允许
> 访问。


Access-Control-Allow-Origin 字面意思：访问控制允许来源。服务器上的默认是不允许其他网址（或者网址相同，但是端口号不同）的网站请求资源的（也就是默认不允许**跨域请求**），如果需要开通权限，就需要设置这个选项。


那么，如何开通服务器上的这个资源访问权限呢？就是要**在服务器**上做下面的设置

```
Access-Control-Allow-Origin: *
```

有了这个设置，所有的第三方网站都可以访问服务器上的资源了。


### 跨域请求的解决方案

解决方案采用： https://github.com/expressjs/cors

cors 是 Cross Origin Resource Share ，安装了这个包就可以完成

```
Access-Control-Allow-Origin: *
```

这个设置了。


保证后台启动状态，现在我们看看当前后台资源的 header 设置，可以用下面的 curl
命令

```
$ curl -I http://localhost:3000/
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: text/html; charset=utf-8
Content-Length: 11
ETag: W/"b-sQqNsWTgdUEFt6mb5y4/5Q"
Date: Thu, 08 Dec 2016 01:51:44 GMT
Connection: keep-alive
```

curl 的 `-I` 选项用来专门拿到服务器返回的 header 。命令返回的信息，就是服务器端被请求资源的的 header 。上面很明显是没有 Access-Control-Allow-Origin 这一项的。下面我们安装 cors 这个包，就可以解决这个问题。


具体步骤如下：

到 https://www.npmjs.com/package/cors 可以看到装包命令如下：

```
npm install --save cors
```

再次提醒：这个包要安装到后台代码中。

然后按照文档，添加下面两行代码，再重启服务器代码：

```js
const cors = require('cors')
app.use(cors());
```

接下来再用 `curl -I` 看看输出如下：

```
HTTP/1.1 200 OK
X-Powered-By: Express
Access-Control-Allow-Origin: *
Content-Type: text/html; charset=utf-8
Content-Length: 11
ETag: W/"b-sQqNsWTgdUEFt6mb5y4/5Q"
Date: Thu, 08 Dec 2016 02:17:23 GMT
Connection: keep-alive
```

这样，我们就看到了 `Access-Control-Allow-Origin: *` 。

浏览器中，刷新一下，可以看到后台返回的 response 数据了。错误没有了。


### 调整接口数据格式

后台代码中，添加下面两个 API

```
app.get('/username', function(req, res){
  res.json({"username": "happypeter"});
})
```

对应，到前台代码中，调整 componentWillMount ，如下：

```
componentWillMount() {
  axios.get('http://localhost:3000/username').then(function(response){
      return console.log(response.data.username);
  })
}
```

这样，前台的 console 中，就可以看到返回数据

```
happypeter
```

下面进一步调整 componentWillMount 如下：

```
componentWillMount() {
  axios.get('http://localhost:3000/username').then((response) => {
      this.setState({username: response.data.username});
  })
}
```

### 总结

至此，前台页面上成功显示出了，后台的数据。这样，一个前后端分离架构，通过 API 通信的应用，我们就完成了。
