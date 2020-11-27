---
title: JS作用域深入理解
date: 2020-03-24 14:01:45
tags: js
categories: js
---

## 作用域
> 作用域是指程序源代码中定义变量的区域，JavaScript 采用词法作用域（lexical scoping），也就是**静态作用域**。

### 静态作用域 & 动态作用域
**静态** ：因为 JavaScript 采用的是词法作用域，函数的作用域在函数定义的时候就决定了。
**动态** ：而与词法作用域相对的是动态作用域，函数的作用域是在函数调用时才决定的。

<!-- more -->

 看个例子🌰：
``` js
var value = 1
function foo () {
  console.log(value)
}
function bar () {
  var value = 2
  foo()
}
bar()  // 输出神马呢？

// 假设：JavaScript采用的是静态作用域。
// 那么函数的作用域在函数定义时就决定了。
// 执行到foo函数，先从foo函数内部查找局部变量value。
// 如果没有就根据书写位置，查找上面一层代码，也就是value = 1。
// 所以结果是1。

// 假设：JavaScript采用的是动态作用域。
// 那么函数的作用域在函数调用时决定。
// 执行到foo函数，先从foo函数内部查找局部变量value。
// 如果没有就从调用函数的作用域，也就是bar函数内部查找value变量，也就是value = 2。
// 所以结果是2。
```
前面说了：JavaScript采用的是静态作用域，所以这个例子的结果是 1。

如果问到 JavaScript 的执行顺序，那么直观的印象就是顺序执行。然而 JavaScript 引擎并非一行一行的分析和执行，而是一段一段的分析执行。执行一段代码的时候，会进行一个“准备工作”，比如变量提升、函数提升。

那这个“一段”是怎么划分的？怎样“准备工作”呢？

> 当执行到一个函数的时候，就会进行“准备工作”，用个更专业的说法就是“执行上下文（execution context）”。

## 执行上下文栈
那可是函数有很多，怎么管理这么多的执行上下文呢？

JavaScript 引擎创建了 **执行上下文栈**（Execution context stack, ECS）来管理执行上下文。

模拟执行上下文栈的行为，我们定义执行上下文栈是一个数组：
```js
  ECStack = []
```
当 JavaScript 开始要解释执行代码的时候，最先遇到的是全局代码，所以初始化的时候首先会向执行上下文栈压入一个全局执行上下文，用 `globalContext` 表示，并且只有当整个应用程序结束的时候，ECStask才会被清空，所以ECStask最底部永远有个 globalContext:
```js
  ECStask = [
    globalContext
  ]
```
🌰例如这段代码：
```js
function fn3 () {
  console.log('fn3')
}
function fn2 () {
  fn3()
}
function fn1 () {
  fn2()
}
fn1()
```
当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完毕的时候，就会将函数的执行上下文从栈中弹出。所以：
```js
// 伪代码

// fn1()
ECStask.push(<fn1> functionContext)

// fn1调用了fn2，创建fn2的执行上下文
ECStask.push(<fn2> functionContext)

// fn2调用了fn3，再创建fn3的执行上下文
ECStask.push(<fn3> functionContext)

// fn3执行完毕
ECStack.pop();

// fn2执行完毕
ECStack.pop();

// fn1执行完毕
ECStack.pop();
```
再看两个相似的例子🌰：
```js
var scope = 'blobal scope'
function checkscope () {
  var scope = 'local scope'
  function f () {
    return scope
  }
  return f()
}
checkscope()

// 模拟执行上下文代码：
// ECStask.push(<checkscope> functionContext)
// ECStask.push(<f> functionContext)
// ECStack.pop()
// ECStack.pop()
```
```js
var scope = 'blobal scope'
function checkscope () {
  var scope = 'local scope'
  function f () {
    return scope
  }
  return f
}
checkscope()()

// 模拟执行上下文代码：
// ECStask.push(<checkscope> functionContext)
// ECStack.pop()
// ECStask.push(<f> functionContext)
// ECStack.pop()
```
> 对于每个执行上下文都有三个重要属性：
> - 变量对象（Variable object，VO）
> - 作用域链（Scope chain）
> - this

## 变量对象
变量对象是与 执行上下文 相关的数据作用域，存储了在上下文中定义的变量和函数声明。

因为不同执行上下文的变量对象不同，所以来了解 `全局上下文的变量对象` 和 `函数上下文的变量对象`。

### 全局上下文
> 全局上下文中的变量对象就是 **全局对象**

执行全局代码时，创建全局执行上下文，全局上下文被压入执行上下文栈，然后初始化：
```js
// 压入执行上下文栈
ECStack = [
  globalContext
]
// 全局上下文初始化
globalContext = {
  VO: [global],
  Scope: [globalContext.VO],
  this: globalContext.VO
}
```

### 函数上下文
在函数上下文中，我们用活动对象（activation object, AO）来表示变量对象。

活动对象是在进入函数上下文时刻被创建的，它通过函数的 `arguments` 属性初始化。`arguments` 属性值是 `Arguments` 对象。

### 执行过程
执行上下文的代码会分成两个阶段进行处理：
> 1. 进入执行上下文
> 2. 代码执行
#### 进入执行上下文
当进入执行上下文时，这时候还没有执行代码，变量对象会包括：
1. **函数的所有形参**（如果是函数上下文） 
    - 由名称和对应组成的一个变量对象的属性被创建
    - 没有实参，属性值设为 undefined
2. **函数声明**
    - 由名称和对应值（函数对象function-object）组成一个变量对象的属性被创建
    - 如果变量对象已经存在相同名称的属性，则完全替换这个属性
3. **变量声明**
     - 由名称和对应值（undefined）组成一个变量对象的属性被创建
     - 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属性

例子🌰来了：
```js
function foo (a) {
  var b = 2
  function c () {}
  var d = function () {}
  b = 3
}
foo(1)
```
在进入执行上下文后，这时候的AO是：
```js
AO = {
  arguments: {
    0: 1,
    length: 1
  },
  a: 1,
  b: undefined,
  c: reference to function c () {},
  d: undefind
}
```
在代码执行阶段，会顺序执行代码，根据代码修改变量对象的值，所以执行代码后的AO是：
```js
AO = {
  arguments: {
    0: 1,
    length: 1
  },
  a: 1,
  b: 3,
  c: reference to function c () {},
  d: reference to FunctionExpression "d"
}
```
所以**变量对象**总结几句是：
1. 全局上下文的变量对象初始化是全局对象
2. 函数上下文的变量对象初始化只包括 Arguments 对象
3. 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始化属性值
4. 在代码执行阶段，会再次修改变量对象的属性值

## 作用域链
当在查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链就叫做作用域链。

下面我们以一个函数的创建和激活两个时期来讲解作用域链是如何创建和变化的。

### 函数创建
上文讲到 JavaScript 是静态作用域，函数的作用域在函数定义的时候就决定了。

这是因为函数有一个内部属性`[[scope]]`，当函数创建的时候，就会保存所有父变量对象到里面，可以理解为`[[scope]]`就是所有父变量对象的层级链（并不代表完整的作用域链）。

例子🌰：
```js
function foo () {
  function bar () {
    ...
  }
}
// 函数创建时，各自的[[scope]]为：
// foo.[[scope]] = [
//   globalContext.VO
// ]
// 
// bar.[[scope]] = [
//   fooContext.AO,
//   blobalContext.VO
// ]
```
### 函数激活
当函数激活时进入函数上下文，创建VO/AO后，就会将活动对象添加到作用域链的前端。

这时候执行上下文的作用域链，我们命名为 Scope:
```js
  Scope = [AO].concat([[Scope]])
```
作用域链创建完毕~

## 缕缕顺
因为如果直接说完函数的作用域就讲作用域链的话，里面的执行上下文就会懵。所以先说的函数执行里面的执行上下文才说的作用域链。有点乱没关系，现在来缕缕顺，当 js 解释器开始工作的时候：
- 创建全局执行上下文，压入执行上下文栈，并初始化
- 函数被创建的时候，就有内部属性作用域链 [[scope]]
- 函数被调用时：
    + 创建函数执行上下文，压入执行上下文栈中
    + 函数执行上下文栈初始化：
        - 复制函数 [[scope]] 属性创建作用域链
        - 用 arguments 创建活动对象
        - 初始化活动对象，即加入形参、函数声明、变量声明
        - 将活动对象 (AO) 压入作用域链顶端
    + 函数代码执行
- 函数执行结束的时候，位于栈顶的执行上下文被弹出，继续执行新的位于栈顶的执行上下文

> 这种方式保证了只有位于栈顶的执行上下文才会被执行，也就是实现了单线程。

## 一个大例子🌰
以下面为例，结合变量对象和执行上下文栈，我们总结一下函数执行上下文中作用域链和变量对象的创建过程：
```js
var scope = 'blobal scope'
function checkscope () {
  var scope = 'local scope'
  function f () {
    return scope
  }
  return f()
}
checkscope()
```
1. **执行全局代码**，创建全局执行上下文，全局执行上下文被压入执行上下文栈
```js
  ECStask = [
    blobalContext
  ]
```
2. **全局上下文初始化**
```js
blobalContext = {
  VO: [vlobal],
  Scope: [globalContext.VO],
  this: globalContext.VO
}
```
3. 初始化的同时，**checkscope 函数被创建**，保存作用域链到内部属性[[scope]]
```js
checkscope.[[scope]] = [
  blobalContext.VO
]
```
4. **执行 checkscope 函数**，创建 checkscope 函数执行上下文，压入执行上下文栈
```js
ECStask = [
  checkscopeContext,
  globalContext
]
```
5. **checkscope 函数执行上下文初始化**：
    - 复制函数 [[scope]] 属性创建作用域链
    - 用 arguments 创建活动对象
    - 初始化活动对象 (AO)，即加入形参、函数声明、变量声明
    - 将活动对象压入 checkscope 作用域链顶端
    - 同时 f 函数被创建，保存作用域链到 f 函数的内部属性 [[scope]]
```js
checkscopeContext = {
  AO: {
    arguments: {
      length: 0
    },
    scope: undefined,
    f: reference to function f () {}
  },
  Scope: [AO, blobalContext.VO],
  this: undefined
}

fscope.[[scope]] = [
  checkscopeContext.AO,
  blobalContext.VO
]
```
6. **执行 f 函数**，创建 f 函数执行上下文，f 函数执行上下文被压入执行上下文栈
```js
ECStask = [
  fContext,
  checkscopeContext,
  globalContext
]
```
7. **f 函数执行上下文初始化**，和第5步相似：
    - 复制函数[[scope]]属性创建作用域链
    - 用 arguments 创建活动对象
    - 初始化活动对象（AO），即加入形参、函数声明、变量声明
    - 将活动对象压入 f 作用域链顶端
```js
fContext = {
  AO: {
    arguments: {
      length: 0
    }
  },
  Scope: [AO, checkscopeContext.AO, blobalContext.VO],
  this: undefined
}
```
8. **f 函数执行**，沿着作用域链查找 scope 值，返回 scope 值
9. **f 函数执行完毕**，f 函数上下文从执行上下文栈中弹出
```js
ECStack = [
  checkscopeContext,
  globalContext
]
```
10. **checkscope 函数执行完毕**，checkscope 执行上下文从执行上下文栈中弹出
```js
ECStack = [
  globalContext
]
```