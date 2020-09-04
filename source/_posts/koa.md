---
title: vue+koa+mysql实现一个简单的todolist
date: 2020-09-03 10:41:27
tags: 
- vue
- koa
catgories: koa
---

## Start

构建一个数据通过 `koa` 的api获取，页面通过 `vue` 渲染的前后端都有的完整demo。包括一个登陆页面和一个todolist页面的增删改查，其中用到了前端 `Vue框架` 、`koa` 提供接口、验证 `token`、`sequelize` 操作 `mysql` 等，这里记录一些关键点。

<!-- more -->
## 初始化

这里用到的是vue-cli2的webpack，项目创建好之后执行
```
yarn 
yarn add koa koa-router koa-logger koa-json koa-bodyparser nodemon vue-router element-ui axios koa-jwt bcrypt kao-static
```
> 在根目录下创建server文件夹，及子文件夹
> - config（数据库配置）
> - controllers（控制层，根据页面的请求选取"数据层"中相应的数据，然后返回给页面）
> - models（数据层，将数据库和表结构连起来）
> - routes（路由）
> - schema（表结构）

> 在根目录下创建app.js作为koa的启动文件（具体代码见项目）

后端：在控制台输入 `nodemon app.js`, 输出`Koa is listening in 8889`说明启动成功，并在8889端口监听。
前端：在控制台输入 `npm run dev` 后在 `localhost:8080` 可以展示前端页面。


## mysql
可以去官网下载`mysql`，我用到的可视化工具是`MySQLWorkbench`

安装好后开始创建数据表。用户表：用于登录验证。待办事项表用于展示待办事项。

user表：

| 字段           | 类型                                                  |
| -------------- | ----------------------------------------------------- |
| id(用戶id) | int(自增)                                              |
| user_name(用户名)       | CHAR(50)            |
| password(密码)   | CHAR(128)                            |
—

list表：

| 字段           | 类型                                                  |
| -------------- | ----------------------------------------------------- |
| id(list的id) | int(自增)                                              |
| user_id(用户id)       | int(11)            |
| content(代办内容)    | CHAR(255)                            |
| status(代办状态)    | tinyint(1)                            |
—

## Sequelize
`sequelize` 理解为用简单的方式操作数据库的ORM框架。

`server/schema` 下新建两个文件，user.js和list.js，是数据库的两张表结构，可使用sequelize-auto直接导出表结构。

在`server/config`下新建db.js, 用于初始化Sequelize和数据库的连接（具体代码见项目）

## JSON-WEB-TOKEN
运用了JSON-WEB-TOKEN的登录系统应该是这样的：
> 1. 用户在登录页输入账号密码，将账号密码（加密后）发送请求给后端
> 2. 后端验证一下用户的账号和密码的信息，如果符合就发一个token返回给客户端，如果不符合就不发送token，返回验证错误信息。
> 3. 如果登录成功，客户端将token用某种方式存下来，之后要请求其他资源的时候，在请求头里带上这个token。
> 4. 后端收到请求信息，先验证下token是否有效，有效则下发请求的资源，无效则返回验证错误。

通过这个token的方式，客户端和服务端之间的访问，是无状态的。也就是服务端不知道你这个用户到底在不在线，只要你发送的请求头里的token是正确的我就给你返回你想要的资源。这样能够不占用服务端的空间资源，而且如果涉及到服务器集群，如果服务器进行维护或者迁移或者需要CDN节点的分配的话，`无状态`的设计显然维护成本更低。

```
<!-- 用法 -->

const secret = 'vue-koa-demo' // 指定密钥，这是之后用来判断token合法性的标志

jwt.sign(userToken, secret) // 签发token

jwt.decode(token) // 解析token
```

## 密码加密
md5加密容易被破解，所以准备采用`bcrypt`的加密方式，全部走后端加密。
```
// 验证密码是否正确
bcrypt.compareSync(data.password, userInfo.password)
```

## Token的发送与验证
在全局发送请求`Header`上加入`Authorization`属性， 值是`Bearer token值`
```
Vue.prototype.$http.defaults.headers.common['Authorization'] = 'Bearer ' + token
```
/api/* 都需要走token验证
```
koa.use("/api",jwt({secret: 'vue-koa-demo'}),api.routes()) // 所有走/api/打头的请求都需要经过jwt中间件的验证。secret密钥必须跟我们当初签发的secret一致
```

## Koa serve静态资源
项目`npm run dev`打包后
```
const serve = require('koa-static')

app.use(serve(path.join(__dirname, 'dist'))) // 将webpack打包好的项目目录作为Koa静态文件服务的目录
```
这样不用启前端项目，localhost:8889就可以访问打包好的静态资源页面。


## End
这是一个前后端都有的完整小demo，结构清晰简单的小东西更容易理解和入门，完整项目[Github地址](https://github.com/Yu-Lxy/Daily_practice/tree/master/koa)。