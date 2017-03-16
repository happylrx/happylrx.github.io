---
title: 用 POST 传复杂数据到服务器（ axios 篇）
---

在 React 应用中，比较少会用到 form 直接提交数据的形式，而更多是采用更为灵活的一种方式，
那就通过 axios/fetch 这样的 http 客户端来发 POST 请求。

跟 form 提交形式不同，axios 提交的内容类型是

```
Content-Type: application/json
```


### 后台代码

[查看 github#tiger 仓库](https://github.com/happypeter/tiger)

### 使用 curl 调试 api

```
curl -H "Content-Type: application/json" -X POST -d '{"username":"happypeter"}' http://localhost:3000/login
```

上面代码中，`-H` 用来设置头部（ header ），这里我们设置的值是 `Content-Type: application/json` 。`-X` 用来设置请求方法，这里设置的值是 `POST` 。`-d` 用来设置数据，
具体的值就是后面的 json 字符串。


### 前台实现最简单的 axios 代码

index.js

```
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import axios from 'axios';

class App extends Component {
  componentWillMount(){
    let username = 'happypeter';
    console.log({username});
    axios.post('http://tiger.haoduoshipin.com/login', {username}).then(res => {
      console.log(res);
    })
  }
  render(){
    return(
      <div>
          test API
      </div>
    )
  }
}

ReactDOM.render(<App/>, document.getElementById('app'));
```


这样就可以发送跟前面 curl 命令一样效果的请求了。
chrome 中查看，请求 Headers 中有：

```
Content-Type:application/json;charset=UTF-8
```

表示 axios 的数据，默认就是按照 `application/json` 发送的。

### 引入一个 form 壳子

react 代码如下：

index.js

```
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import axios from 'axios';

class App extends Component {
  handleSubmit(e) {
    e.preventDefault();
    console.log('.....');
    let username = this.refs.username.value;
    axios.post('http://tiger.haoduoshipin.com/login', {username}).then(res => {
      console.log(res.data.msg);
    })
  }
  render(){
    return(
      <div>
        <form onSubmit={this.handleSubmit.bind(this)}>
          <input ref="username" type="text" />
          <input type="submit" />
        </form>
      </div>
    )
  }
}

ReactDOM.render(<App/>, document.getElementById('app'));
```

补充知识：

- handle 的意思是“处理”的意思，handleSubmit 就是”处理提交“的意思
- form  的意思是”表单“
- refs 是 React [自带功能](https://facebook.github.io/react/docs/refs-and-the-dom.html)
  ref 的中文意思是”指向“或者”指针“

### 总结

使用 axios 发送数据，是最为灵活和强大的一种方式，以后我们会用到的比较多。
