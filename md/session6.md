---
title: 用户注册登录（ API 功能 ）
---


这集实现用户注册和登录登出相关的 API 功能。

### 用户注册

创建 controllers/user.js 内容如下：


```js
let User = require('../models/user.js');
// 注册
exports.signup = function (req, res) {
  let _user = req.body;
  User.findOne({username:_user.username},function (err,user) {
    if (err) return res.status(500).json({msg: '注册失败，请重试',err});
    if (user) {
      return res.status(403).json({msg: '用户名重复，请重新注册'})
    }else {
      // 这里不适用 var 声明是因为函数的形参有了这个变量，为 null
      user = new User(_user);
      user.save(function (err,user) {
        if (err) return res.status(500).json({msg: '注册失败，请重试',err});
        res.json({
          userId: user._id,
          username: user.username,
          msg: '注册成功'
        })
      })
    }
  })
}
```

models/user.js 内容：

```js
var mongoose = require('mongoose');
var bcrypt = require('bcrypt');
let SALT_WORK_FACTOR = 10;
var ObjectId = mongoose.Schema.Types.ObjectId;

var UserSchema = new mongoose.Schema(
  {
    username: { type:String, maxlength: 18 },
    password: String,
    // admin>10
    role: { type: Number, default: 0 },
    shops: [{ type: ObjectId, ref: 'Shopping' }]
  },
  {timestamps:true}
)
// Schema(模式)的中间件方法，这里用不了了。。。因为在购物车里会有save方法的调用，会再次触发这个方法，导致密码出问题
// e,这里还是不换了，将shopping中改为了更新方法
UserSchema.pre('save',function (next) {
  var user = this;

  bcrypt.genSalt(SALT_WORK_FACTOR, function (err,salt) {
    if (err) return next(err);
    bcrypt.hash(user.password, salt, function (err, hash) {
      if (err) return next(err);
      user.password = hash;
      next()
    })
  })
})

// user 的实例方法
UserSchema.methods = {
  comparePassword: function (_password, cb) {
    // console.log(_password);
    console.log(this.password);
    bcrypt.compare(_password, this.password, function (err, isMatch) {
      if (err) return cb(err);
      cb(null, isMatch);
    })
  }
}

module.exports = mongoose.model('User',UserSchema);
```

上面的代码要求安装 bcrypt 这个包：

```
npm i --save bcrypt
```

routes.js 中添加一行

```
app.post('/user/signup', User.signup)
```


API 文档在这里： https://github.com/happypeter/tiger-api

后台 API 实现代码：

[API: POST /user/signup](https://github.com/happypeter/tiger-api/commit/9eb1371b690268d5a041470e1b2ecbb411f30969)

下面用 curl 来测试一下用户注册对应的 API ：

```
$ curl -X POST -H 'Content-Type: application/json' -d '{"username": "happypeter", "password": "111111"}' localhost:3005/user/signup
{"userId":"58d494413cc84e4dd6c0094f","username":"happypeter","msg":"注册成功"}%
```


### 用户登录登出


[API: /signin /logout](https://github.com/happypeter/tiger-api/commit/2b55d6cbb2f99bb6fa85b7dcdaddff88d3047002)
