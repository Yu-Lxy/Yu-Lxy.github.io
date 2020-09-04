---
title: try catch能捕获到哪些JS异常
date: 2020-08-04 09:50:45
tags: js
categories: js
---

>写代码时经常会用到 `try catch`，防止一些JS报错，导致页面挂掉。那么到底哪些JS异常能被捕获到呢？

<!-- more -->

>**简单解释就是：在报错的时候，线程执行已经进入 `try catch` 代码块，并且处在 `try catch` 里，才能被捕捉到。**（之前，之后都无法捕捉异常）

- ###### 下面三个小栗子，解释一下 `try catch` 的前中后

🌰 例子1：
```
try{
    a(
} catch (e) {
    console.log('error', e)
}
// output
Uncaught SyntaxError: Unexpected token }
```
*例子1* 语法异常（SyntaxError），因为语法异常是在语法阶段就报错了，所以线程还没进入 `try catch` 代码块，就捕获不到异常。

🌰 例子2：
```
function d () {a.b}
try {
   d()
} catch (e) {
   console.log('error', e)
}
// output
error ReferenceError: a is not defined
```
*例子2* 报错的时机，是代码执行进入了 `try catch` ，执行 d 方法的时候，线程执行处在 `try` 里面，所以能捕捉到。

🌰 例子3：
```
try {
   function d () {a.b}
} catch (e) {
   console.log('error', e)
}
d()
// output
error ReferenceError: a is not defined
```
*例子3* 方法定义在 `try catch` 代码块里，但是执行方法在 `try catch` 外，执行 d 方法的时候报错，此时 try catch 已经执行完成，所以无法捕捉异常。

三个例子之后应该能理解怎样的JS异常可以被捕获了，但是在我们用 `promise` 的时候发现相对于外部的 `try catch` ，Promise 没有异常！

### promise & try catch
🌰 例子4：
```
try{
  new Promise((resolve, reject) => {
    a.b
  }).then(v => {
    console(v)
  })
} catch (e) {
  console.log('error', e)
}
// output
Uncaught (in promise) ReferenceError: a is not defined
```
*例子4* 线程在执行a.b的时候，适时向属于同步执行，`try catch` 并未执行完成，为什么捕获不到异常呢？

事实上，**Promise 的异常都是由 reject 和 Promise.prototype.catch 来捕获**，不管是同步还是异步。

核心原因是因为 `Promise` 在执行回调中都用 `try catch` 包裹起来了，其中所有的异常都被内部捕获到了，并未往上抛异常，所以异常都不会被外层的 `try catch` 捕捉。

🌰 例子5：
```
function a(){
  return new Promise((resolve, reject) =>{
    setTimeout(() => {
      reject('报错了')
    })
  })
}

async function foo () {
  try{
    await a()
  } catch (e) {
    console.log('error', e)
  }
}
foo()
// output
error 报错了
```
为什么 *例子5* 的异常能被 `catch` 捕获到呢，因为报错的时候，线程执行已经进入了 `try catch` 代码块，并且异常由 reject 抛出，自然可以捕获到。

 ###总结：
> **1. 在报错的时候，线程执行已经进入 `try catch` 代码块，并且处在 `try catch` 里，才能被捕捉到。**

> **2. 不要用 `try catch` 包裹 `Promise`，我们只需要给 `Promise` 增加 Promise#catch 就 OK 了**