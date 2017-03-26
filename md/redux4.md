---
title: Redux Store 数据大本营
---

Redux 是数据流管理工具。使用 Redux 的最重要的一句话：

>一切数据都要保存的 Store 之中，组件自己不保留自己的 state 数据

后续我们会学的一切的技巧基本服务于一个关系：组件和 Store 的关系。

### 为何要统一存放到 Store 中？

把所有的数据都存放到 Store 中，然后所有的组件都订阅 Store 的数据。那么数据就有了 Single Source of Truth （唯一可信数据源）。


### 创建 Store ？

其实 Store 的主体就是写到 store.js 中的傻傻的数据：

类似：

```
let comments = [
    "hello1",
    "hello2"
]
```

但是，store 中的数据，必然要涉及读和写的操作，所以我们要安装
一个包来辅助后续操作：

```
npm i --save redux
```

到 store.js 中先导入

```
import { createStore } from 'redux';
```

我们可想象到的就是， store.js 中会做这样的导出

```
let store = createStore(comments)
export default store;
```

但是，createStore 的[官方文档](http://redux.js.org/docs/api/createStore.html) 中指明这个函数是至少多传人一个参数的。

这个参数就是 reducer 。

### reducer

reducer 和 store 一样是 redux 三大核心概念之一。reducer 是
一个函数，用来修改 store 中的数据。

举个例子：

```
function commentReducer(state = [], action) {
  // console.log(state, action);
  switch (action.type) {
    case 'ADD_COMMENT':
      // console.log([...state, action.comment])
      return [...state, action.comment]
    default:
      return state;
  }
}
```

但是我们这里先写一个空的 reducer ：

```
function commentReducer(state = [], action) {
  return state;
}
```


### 代码

下面的代码实现了，数据存储到 store.js 同时组件内部读取 store.js 中的数据成功。

代码： **add store**


### 修改 Store 中的数据

前面实现了从 Store 中通过 store.getState() 读数据。下面
我们来实现修改数据的操作。这个主要通过 Redux 三大概念的，另外两个：action ， reducer 来配合使用。

基本原理：组件发出 action ，action 触发 reducer ，真正修改
数据，是通过 reducer 来完成。


### 先来看 action


前面的用词是**组件发出 action** 可见 action 是名词。那么 redux 这里 action 其实就是一个如下的对象：

```
let action = {
  type: 'ADD_COMMENT',
  comment: "hello3"
}
```

action 对象有两个部分组成：

- 第一部分，要有一个 type 属性，值是一个字符串。
- 第二部分，payload ，也就是这个 action 携带的数据

上面的这个形式，让我们联想起了 axios 发 POST 请求（action 的接受方就是 reducer）。

发出一个 action 就是在组件内部：

```
store.dispatch(action)
```

就可以发出 action 。

发出之后，需要有专门对应的 reducer 进行接收。

### Reducer

下面这个就是一个 reducer

```
function commentReducer(state = [], action) {
  // console.log(state, action);
  switch (action.type) {
    case 'ADD_COMMENT':
      // console.log([...state, action.comment])
      return [...state, action.comment]
    default:
      return state;
  }
}
```

它的作用就是接受 action ，然后根据 action 修改 store 中的数据。

代码： **action reducer**


### 修改数据的时间维度的执行流程

首先，用户点提交按钮

```
store.dispatch({type, payload})
```

然后，由 store.js 中的 commentReducer 来接收。

```
function commentReducer(state = [], action) {
  // console.log(state, action);
  switch (action.type) {
    case 'ADD_COMMENT':
      // console.log([...state, action.comment])
      return [...state, action.comment]
    default:
      return state;
  }
}
```
也就是 commentReducer 要开始执行，执行的时候，state 是什么呢？

我们可以看到 commentReducer 中 state 的默认值是 `[]` ，
也就是说如果 state 我们不给它赋值，后续执行中它的值就是 `[]`

并且这里比较奇怪的是，我们也没又看到 commentReducer 的调用语句，类似

```
let state = [
...
]
commentReducer(state, action)
```
所以感觉上是是没有给 state 。但是，其实 redux 采用的是**函数式编程**的思想。

commentReducer 的调用是在

```
const store = createStore(commentReducer, comments);
```

并且，调用过程中，state 也赋值为 `comments`。


当 `return [...state, action.comment]` 执行之后，
store.getState() 的值就被改变了。

### 思考一个问题

如果 PostBody.js 写成这样

```
import React, { Component } from 'react';
import store from '../store';


class PostBody extends Component {
  constructor(){
    super();
    this.state = {
      num: store.getState().length
    }
  }
  render(){
    return(
      <div className="post-body">
        <div className="comment-num">
          { this.state.num }
        </div>
      </div>
    )
  }
}

export default PostBody;
```

那么，初始条件下，是可以从 store 中读取评论数量的，但是为什么，store 数据更新后，这里的
评论数量不会变？

答案：就是这里的 store.getState() 没有再次被执行。

好，那么如何得到更新后的评论数量呢？

代码：**problem**
