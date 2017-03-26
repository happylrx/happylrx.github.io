---
title: 新建商品
---

创建分类没有问题了，现在就可以来创建商品了。


### 后台 API

代码：

[API: product new/delete/post
](https://github.com/happypeter/tiger-api/commit/2b49955be3ef01c3c71a1a87136972edab131c6e)

```
$ curl -X POST -H 'Content-Type: application/json'  -d '{"name": "shoe"}' localhost:3005/cat

$ curl -X GET localhost:3005/cats

$ curl -X POST -H 'Content-Type: application/json' localhost:3005/product/new -d '{ "name": "商品名称", "summary": "这是简介", "price": 1003, "poster": "http://7xopqp.com1.z0.glb.clouddn.com/avater.jpg", "cat": "58d341c7dd5b380f976b1645" }'
```


重点涉及到表关系，

```
cat.products
```

总是取不到数据

model/cat.js 中必须有

```
products: [{
  type: ObjectId,
  ref: 'Product'
}]
```

这样

```
$ curl -X POST -H 'Content-Type: application/json' localhost:3005/product/new -d '{ "name": "商品A", "summary": "这是简介", "price": 1003, "poster": "http://7xopqp.com1.z0.glb.clouddn.com/avater.jpg", "cat": "58d34967aa3fcd439fdd9190" }'
{"msg":"新增商品成功","product":{"__v":0,"name":"商品名称","summary":"这是简介","price":1003,"poster":"http://7xopqp.com1.z0.glb.clouddn.com/avater.jpg","cat":"58d34318d7323710331dc2bc","_id":"58d344c5d4668b112628a765"}}%
```

获取一个商品也可以成功了：

```
$ curl localhost:3005/product/detail/58d345d5d4668b112628a766
{"msg":"获取商品详情成功","product":{"_id":"58d345d5d4668b112628a766","name":"商品B","summary":"这是简介","price":1003,"poster":"http://7xopqp.com1.z0.glb.clouddn.com/avater.jpg","cat":{"_id":"58d34318d7323710331dc2bc","name":"fish"},"__v":0}}%
```

删除一个商品也没有问题

```
curl -X DELETE localhost:3005/product/delete/58d345d5d4668b112628a766
```


FIXME: 后台 API 去获取一个已经被删除的商品也返回“获取成功”

```
$ curl -X DELETE localhost:3005/product/delete/58d345d5d4668b112628a766

$ curl localhost:3005/product/detail/58d345d5d4668b112628a766
{"msg":"获取商品详情成功","product":null}%
```

### 前端实现

首先把一些配置信息都移动到 config.js 文件

[move link to config.js
](https://github.com/happypeter/tiger/commit/c778fb2aca11e6b17ba0e77d0da27e1a2d6027c6)


然后专门新建一个 new-product 页面：

[new-product page](https://github.com/happypeter/tiger/commit/1faa838134eafd9a8efdfc87a01ee6d43374210e)


接下来创建新商品：

需要注意的地方，就是测试的时候 price 一项一定不能填写任意字符串，而要填写数字，不然服务器端会返回 500

[post new product](https://github.com/happypeter/tiger/commit/d181b89a4eb39cb675d8231dfded752cc9b846c6)
