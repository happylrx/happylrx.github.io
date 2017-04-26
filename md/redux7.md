---
title: 处理两类数据
---


### 任务一：添加 React Router 进来

代码：[add react router](https://github.com/happypeter/redux-hello/commit/71f6b418afca4754b034a1b63517e96b4e45cd2e)

这部分没有什么新知识点引入。

### 任务二：添加多个 store 数据，添加 root reducer

代码：[add rootReducer](https://github.com/happypeter/redux-hello/commit/0e7564af38ca54e01443b6b50dc5af32614c22d1)

```
const rootReducer = combineReducers({
   posts: postReducer,
   comments: commentReducer
});
```

通过如上的 rootReducer 的使用，相当于对整个状态树进行的分拆。由 `postReducer` 专门更新 `posts`，由 `commentReducer` 专门负责 `comments` 数据。

上面的代码中把每篇 post （文章）也都接收到了数据自己的 id 。方便后续我们对不同文章进行区分，也对属于不同文章的评论进行分组。


### 任务三：添加另一组评论，每篇 post 显示自己的评论

代码：[show comments for each post](https://github.com/happypeter/redux-hello/commit/5c2888e357295874672bb8430e2fecd25c0cb753)

### 任务四：提交评论失败，现在修复一下

代码： [ADD_COMMENTS works](https://github.com/happypeter/redux-hello/commit/2e2e5bb774a428e6eabd75c7353374ed9c253def)


用到了[对象展开运算符](http://cn.redux.js.org/docs/recipes/UsingObjectSpreadOperator.html) 。

### 任务五：添加点赞功能

- 第一步，添加样式：[like button style](https://github.com/happypeter/redux-hello/commit/018596f43bd07f3d5634a55b580ea4815ce1684a)

- 第二步，从 store 中读取数据 [read like number from store](https://github.com/happypeter/redux-hello/commit/f661e9b618059c6123a91a4be88027bd8df1817e)

- 第三步，添加 reducer [INCREMENT_LIKES works](https://github.com/happypeter/redux-hello/commit/71b9e1bb09730cc35e98ff5745c98bbd2bbdc59b)

### 任务六：完成首页列表

- 第一步：PostBody 中添加 title [add title to PostBody](https://github.com/happypeter/redux-hello/commit/b397b768dd6a5b75163052801468f4d238141ce8)
- 第二步：首页组件从 store 中读取数据 [read post data from store](https://github.com/happypeter/redux-hello/commit/97d0e52c68938f64a9a58e5481e2934dcc77026e)

- 第三步：首页展示博客列表： [add postList to HOME](https://github.com/happypeter/redux-hello/commit/9cd963a3cd826e308ccdf3b3068340e1bd55bf6a)
