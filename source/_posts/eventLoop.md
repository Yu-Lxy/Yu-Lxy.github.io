---
title: EventLoop 的简单理解
date: 2020-07-28 10:13:09
tags: js
categories: js
---

### EventLoop是什么
> 一个循环 每次循环叫tick 每次循环的代码叫task

- V8引擎单线程无法同时干两件事
- 文件读取、网络IO缓慢且具有不确定性
- 要通过异步回调方式处理又称为异步IO
- 先同步再异步 异步放入队列等同步完成后再执行 每次循环叫一个tick(process.nextTick())

<!-- more -->

### 异步任务的区分
microtasks(微任务)：
> 唯一，整个事件循环中仅存在一个；执行为同步，同一个事件循环中的microtasks会按队列顺序串行执行完毕
- process.nextTick
- promise
- Object.observe
- MutationObserver

macrotask(宏任务)：
- setTimeout
- serInterval
- setImmediate
- I/O
- UI渲染

> 先执行微任务 再执行宏任务

如果遇到async的代码 await后面的程序就挂起 类似于微任务 等到后面的同步任务执行完了再执行

最后上一个小栗子🌰
```
(new Promise(resolve => {
  console.log('resolve')
  resolve()
}))
.then(() => console.log('promise then ...'))

async function fun(){
  console.log("fun start")
  const aa = await 1 // 到这里挂起，后面先不执行
  console.log("fun end")
}
fun()

// setTimeout 放入event table中 1秒后将回调函数放入宏任务的event queue中
setTimeout(() => {
  console.log('setTimeout ...')
}, 0)

process.nextTick(() => {
  console.log('nextTick ...')
})

console.log('console ...')


// output
resolve
fun start
console ...
nextTick ...
promise then ...
fun end
setTimeout ...
```