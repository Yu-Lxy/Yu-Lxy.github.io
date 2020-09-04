---
title: 第一次使用mongoDB的记录
date: 2020-09-04 15:30:15
tags: mongoDB
categories: mongoDB
---
## 简介

一个 `vue+express+mongodb` 的小demo，实现了列表的增删查，以及简单的分页功能。
[Github项目地址](https://github.com/Yu-Lxy/Daily_practice/tree/master/mongo)

<!-- more -->

## mongo命令行操作
```
// 进入mongo命令行
mongo

// 查询所有数据库
show dbs

// 切换/创建数据库,当创建一个集合(table)的时候会自动创建当前数据库
use test

// 查询
db.表名.find()

// 条件查询
db.表名.find({price: 5})

// 插入(insertOne/insertMany/save)
db.表名.insertOne({name: '苹果', price: 5})

// 更新
db.表名.update({ name: '苹果' }, { $set: { price: 6 } })

// 删除(deleteOne/deleteMany/remove)
db.表名.updateOne({ name: '苹果' })
```

## mongoDB
> mongoDB的基本操作

1. 连接mongoDB
```
// 客户端
const MongoClient = require('mongodb').MongoClient

// 连接URL
const url = 'mongodb://localhost:27017'

// 数据库名
const dbName = 'test'

(async function() {
  // 0.创建客户端
  const client = new MongoClient(url, { useNewUrlParser: true })
  try {
    // 1.连接数据库(异步) 
    await client.connect() 
    console.log('连接成功')
  } catch (error) {
    console.error(error)
  }
  client.close()
})()
```
2. 获取数据库
```
const db = client.db(dbName)
```
3. 获取表
```
const col = db.collection(colName)
```

基本操作了解后运行一下[项目](https://github.com/Yu-Lxy/Daily_practice/tree/master/mongo)里的代码~ 👇

## 运行
首先安装以下
```
npm i express path events mongodb nodemon
```

conf.js里设置自己的mongodb配置

数据库没数据的话先执行以下添加数据：
```
cd models
nodemon testData.js
```

添加好之后ctrl+c, 再执行以下
```
cd ..
nodemon index.js
```
打开localhost:3000能看到如下样式

![image.png](https://i.loli.net/2020/08/27/ALNjeIKPOY9QTHF.png)


## EventEmitter
[项目](https://github.com/Yu-Lxy/Daily_practice/tree/master/mongo)中有一个testData.js，执行后可以插入测试数据。其中有一个 `mongodb.once()` 方法，实际上在db.js里的 `MongDB类` 中是执行了 `EventEmitter的once` 方法。events模块只提供了一个对象： `events.EventEmitter`， 其核心就是事件触发与事件监听器功能的封装，可以通过require('events')来访问该模块。
```
// 例子
const EventEmitter = require('events').EventEmitter 
const event = new EventEmitter() 
event.on('some_event', num =>  { 
  console.log('some_event 事件触发:' + num) 
}) 
let num = 0
setInterval(() =>  { 
  event.emit('some_event' , num ++ ) 
}, 1000) 
```
`event.once()` 是只执行一次的监听，所以执行 `nodemon testData.js` 后，只触发一次连库的操作并执行回调。

## 注意的点
- 后端get请求的参数从`query`里拿, post请求的参数从`body`里拿。
- `const page = + req.query.page` +号为了转Number类型。
- `.skip(n).limit(m)` 意为跳过n个取m个。
- mongoDB插入数据时自动生成的 `_id` 是 `ObjectId` 对象类型，所以当参数作为查询条件时需要引入 `mongodb的ObjectID`，传参时调用。

## End
前端小白第一次使用mongoDB的记录📝，简单小例子容易入门和理解，轻喷~ 😆