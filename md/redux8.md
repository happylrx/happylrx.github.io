---
title: 联通 API
---


前面的应用看起来功能完整了，但是实际的数据都没有被保存到数据库（数据持久化没有做）。这集来解决这个问题。


### 调整一下代码

首先 CommentBox 组件中还使用了 getState() 和 this.setState 这样的方式，这个都是不符合 redux 的基本思想的。所以调整一下：

- [this.props.comments](https://github.com/happypeter/redux-hello/commit/62439a59e1d830e7dcbe544e058f75f1baf19b33)


注：为何不符合 redux 基本思想？那么 getState 不让用了，替代方案是什么？Redux 基本思想是

> 组件不管理自己的 state

所以，如果要更新 state 值，那也不允许直接在组件内该。不用 getState 替代方式是：

> 组件发出 action ，action 触发 reducer ，reducer 修改 store 状态树。

由于各个组件，都通过 __react-redux__ 库的 __Provider__ 和 __connect__ 接口订阅了 store 中的数据，所以，只要 store 数据变化了，那么组件也是可以自动刷新的。

### 引入 redux-thunk

thunk 的意思是“模式转换”，有了 redux-thunk ，redux 的工作模式就会发生一定的“转换”，具体来讲就是[官网](https://github.com/gaearon/redux-thunk)上说的

>Redux Thunk middleware allows you to write action creators that return a function instead of an action

翻译：Redux-thunk 中间件允许我们的 action creator 不直接返回 action ，而是去返回一个函数。（这样做的目的，就是为了实现一异步操作）。

Redux-thunk 解决的问题是：**异步操作** 。没有 thunk 之前，一旦用户点按钮，aciton 立即被发出，reducer 立即执行，store 立即改变。这个是 redux 默认工作机制。但是这种机制在很多情况下是不能满足要求的，例如，用户点按钮之后，action 中要携带的数据需要从网络上读取的这种情况。有了 thunk 之后，即使要等很长时间也没有问题，因为 diptach 操作可以等待网络请求结束之后再去执行。

先来装包

```
npm install --save redux-thunk
```

安装的版本是

```
"redux-thunk": "^2.2.0"
```



到 store.js 中，

```
import { createStore, applyMiddleware, compose } from 'redux';
```

- applyMiddleware: 使用中间件
- compose：构建


下面的代码是让我们的 redux 加载 thunk 中间件：

- [load thunk](https://github.com/happypeter/redux-hello/commit/473c8403d908b30cd4635d3a60830db82df3919a)

加载 thunk 之后，我们 dispatch（分发）action 的方式就有了很大变化。对 reducer 影响不大。

下面我们来创建 actions/commentActions.js 文件，里面会放很多函数，这些函数本身不是 action ，但是它们的返回值（或者说就是最终运算的结果是 action )，我们把这一类的函数叫 Action Creator （ Action 创建器）。

注：没有安装 thunk 之前，也是可以创建 Action Creator ，但是使用形式差别很大。但是有了 thunk 之后，Action Creator 的功能就变得很强大了。

下面的代码，使用了 redux-thunk ，虽然没有用它来完成任何异步操作：

- [thunk works](https://github.com/happypeter/redux-hello/commit/191c42a2c5d553e32f7ed331dc59bd1f4c046940)


### 后台 API


https://coding.net/u/happypeter/p/redux-hello-api/git

上面的 API 已经部署到了 http://redux-hello.haoduoshipin.com


### 实现 axios 异步操作

- [thunk aync works](https://github.com/happypeter/redux-hello/commit/ccddb99883a73565f8f064284ec39e6e5b387dbd)


### 实现 axios 读取后台评论操作

要求，store 中的死数据（仅限于评论）全部删除，从后台读取，加载到页面上。每篇文章各自显示自己的评论。

难点：后端得到的数据，跟我们前端需要的格式有差异，需要用代码调整一下数据格式。

参考：

如何从服务器加载数据到 redux 中，参考：https://github.com/happypeter/hand-in-hand-react-demo/blob/master/client/complete/src/ui/DashBoard.js#L11

代码：

- [load data from server OK](https://github.com/happypeter/redux-hello/commit/3305ae8cf37dc8c46c535091e2264e1a8eedb13b) 调整了 action ，成功的从服务器端获取了所有评论，并显示到了页面上。

现在面临的任务就是数据的整理了：

要把服务器返回的

```
{"_id":"58d88a4d6281635fbed182bd","postId":"1","commentBody":"hello1","__v":0},{"_id":"58d88b8b17deb3614c29065f","postId":"2","commentBody":"hello2","__v":0}
```

整理成下面的格式：

```
let comments = {
  1: ['nice course', 'help me a lot'],
  2: ['really good', 'save me lots of time']
}
```


解决方案如下：

```
let cc = [
  { _id: '58d88a4d6281635fbed182bd',
    postId: '1',
    commentBody: 'hello1',
    __v: 0 },
  { _id: '58d88b8b17deb3614c29065f',
    postId: '2',
    commentBody: 'hello2',
    __v: 0 },
  { _id: '98d88a4d6281635fbed182bd',
    postId: '1',
    commentBody: 'hello11',
    __v: 0 },
  { _id: '58d8bf1317deb3614c290660',
    postId: '2',
    commentBody: 'hello22',
    __v: 0 }
]

let post1Com  = cc.filter(value => value.postId != '2' ).map(item => {
  return item.commentBody;
})

let post2Com = cc.filter(value => value.postId != '1' ).map(item => {
  return item.commentBody;
});
console.log(post1Com, post2Com);
// [ 'hello1', 'hello11' ] [ 'hello2', 'hello22' ]
```


继续上代码：

- [data format change](https://github.com/happypeter/redux-hello/commit/ed7316ea044bedb7b37e3d48c923085fdf31a08b) 这个最终实现了数据格式的转换
- [bring commentList back](https://github.com/happypeter/redux-hello/commit/1852f67ee5d0c1bdb94db8f571947d4d63fc20bb) 页面重新正常显示评论
