---
title: 《Node.js 事件循环》
tags:
    - Node.js
    - JavaScript
    - 事件循环
    - 异步编程
categories:
    - Node.js
    - 后端开发
date: "2026-04-15 15:30:07"
updated: "2026-04-15 15:30:07"
abbrlink: f8882815
---

# Node.js 事件循环（Event Loop）详解

Node.js 的**事件循环（Event Loop）**是 Node.js 最核心的机制之一，也是面试中非常高频的问题。

理解事件循环，需要先理解一句话：

> **Node.js 使用单线程执行 JavaScript，通过事件循环和异步 IO，让一个线程能够处理大量并发请求。**

---

# 1. 为什么 Node.js 需要事件循环？

JavaScript 本身是：

```text
单线程语言
```

意味着：

```text
同一时间只能执行一段代码
```

例如：

```javascript
console.log("开始")

const result = fs.readFileSync("test.txt")

console.log(result)

console.log("结束")
```

执行过程：

```text
开始

↓

等待文件读取

↓

读取完成

↓

结束
```

如果文件读取需要 5 秒：

整个程序阻塞 5 秒。

---

但是服务器中：

```text
用户A 请求接口

用户B 请求接口

用户C 请求接口
```

不能让一个用户的 IO 操作阻塞其他用户。

所以 Node.js 引入：

```text
事件循环 Event Loop
```

---

# 2. Node.js 的整体运行结构

Node.js 主要由几个部分组成：

```text
              JavaScript代码

                    ↓

              V8 JavaScript引擎

                    ↓

              Node.js运行环境

        ┌───────────┴───────────┐

       Event Loop            libuv

                              ↓

                        操作系统线程池

                              ↓

                         文件/网络/数据库
```

---

其中：

## V8

负责：

```text
执行 JavaScript代码
```

## libuv

负责：

```text
异步任务
事件循环
线程池
网络IO
```

---

# 3. 事件循环核心流程

Node.js 执行代码时：

```text
同步代码

↓

微任务队列

↓

事件循环阶段

↓

宏任务

↓

重复循环
```

简单理解：

```text
先执行当前代码

再处理已经完成的异步任务

继续执行新的异步任务
```

---

# 4. Event Loop 六个阶段

Node.js 的事件循环主要分为：

```text
┌────────────────────┐
│ timers             │
├────────────────────┤
│ pending callbacks  │
├────────────────────┤
│ idle prepare       │
├────────────────────┤
│ poll               │
├────────────────────┤
│ check              │
├────────────────────┤
│ close callbacks    │
└────────────────────┘
```

每一轮循环称为：

```text
Tick
```

---

# 5. 六个阶段详细解释

---

# 第一阶段：timers

负责执行：

```javascript
setTimeout()

setInterval()
```

例如：

```javascript
setTimeout(()=>{

    console.log("timer")

},1000)
```

1 秒后进入 timers 阶段执行。

注意：

> setTimeout(1000) 不是保证 1000ms 后执行，而是至少等待 1000ms。

例如：

如果主线程正在执行：

```javascript
while(true){}
```

那么 timer 永远不会执行。

---

# 第二阶段：pending callbacks

处理一些系统级回调。

例如：

* TCP错误
* 网络异常

普通开发中接触较少。

---

# 第三阶段：idle、prepare

Node.js 内部使用。

开发者基本不会接触。

---

# 第四阶段：poll（最重要）

poll 是事件循环的核心阶段。

主要处理：

* IO 回调
* 文件读取
* 网络请求

例如：

```javascript
const fs = require("fs")


fs.readFile("./a.txt",()=>{

    console.log("读取完成")

})
```

流程：

```text
调用readFile

↓

交给系统处理

↓

Node继续执行其他代码

↓

文件读取完成

↓

poll阶段执行回调
```

---

# 第五阶段：check

执行：

```javascript
setImmediate()
```

例如：

```javascript
setImmediate(()=>{

 console.log("immediate")

})
```

会在 check 阶段执行。

---

# 第六阶段：close callbacks

执行关闭事件。

例如：

```javascript
socket.on("close",()=>{

})
```

---

# 6. 宏任务和微任务

这是面试重点。

Node.js 中任务分两类：

---

# 宏任务（Macrotask）

包括：

```text
setTimeout

setInterval

setImmediate

IO回调
```

---

# 微任务（Microtask）

包括：

```text
Promise.then

catch

finally

process.nextTick
```

---

执行优先级：

```text
同步代码

↓

process.nextTick

↓

Promise.then

↓

timer

↓

IO

↓

setImmediate
```

---

# 7. 一个经典面试题

代码：

```javascript
console.log("start")


setTimeout(()=>{

 console.log("timeout")

},0)


Promise.resolve().then(()=>{

 console.log("promise")

})


console.log("end")
```

输出：

```text
start
end
promise
timeout
```

为什么？

---

## 第一步：执行同步代码

```javascript
console.log("start")
```

输出：

```text
start
```

遇到：

```javascript
setTimeout()
```

放入 timer 队列。

遇到：

```javascript
Promise.then()
```

放入微任务队列。

继续：

```javascript
console.log("end")
```

输出：

```text
end
```

---

## 第二步：执行微任务

Promise 优先：

输出：

```text
promise
```

---

## 第三步：进入事件循环

执行 timer：

输出：

```text
timeout
```

最终：

```text
start
end
promise
timeout
```

---

# 8. process.nextTick 和 Promise 的区别

代码：

```javascript
process.nextTick(()=>{

 console.log("nextTick")

})


Promise.resolve().then(()=>{

 console.log("promise")

})
```

结果：

```text
nextTick

promise
```

原因：

Node.js 对：

```text
process.nextTick
```

优先级最高。

执行顺序：

```text
同步代码

↓

nextTick

↓

Promise

↓

Event Loop
```

---

# 9. setTimeout 和 setImmediate 区别

经典问题：

```javascript
setTimeout(()=>{

 console.log("timeout")

},0)


setImmediate(()=>{

 console.log("immediate")

})
```

输出？

答案：

**不确定。**

原因：

两个属于不同阶段：

```text
setTimeout

↓

timers阶段


setImmediate

↓

check阶段
```

第一次循环时：

取决于环境。

---

但是：

如果在 IO 回调里面：

```javascript
fs.readFile("test.txt",()=>{


 setTimeout(()=>{

 },0)


 setImmediate(()=>{

 })


})
```

通常：

输出：

```text
immediate

timeout
```

因为：

IO回调结束后：

```text
poll

↓

check

↓

timers
```

---

# 10. async/await 与事件循环

例如：

```javascript
async function test(){

 console.log(1)

 await Promise.resolve()

 console.log(2)

}


test()


console.log(3)
```

执行：

输出：

```text
1
3
2
```

原因：

await 后面的代码：

```javascript
console.log(2)
```

会进入：

```text
Promise微任务队列
```

---

# 11. Node.js 为什么单线程还能高并发？

关键：

## JavaScript线程

负责：

```text
执行代码
处理事件循环
```

---

## IO任务

交给：

```text
操作系统

或者

libuv线程池
```

例如：

文件读取：

```text
JavaScript线程

↓

libuv线程池

↓

读取文件

↓

通知Event Loop

↓

执行回调
```

---

所以：

Node.js：

```text
一个JavaScript线程

+

多个异步IO任务

=

高并发
```

---

# 12. 面试回答版本（重点）

如果面试官问：

> Node.js 事件循环是什么？

可以回答：

> Node.js 事件循环是一种异步非阻塞机制，它允许 Node.js 在单线程中处理大量并发任务。JavaScript 主线程执行同步代码，当遇到 IO、定时器等异步任务时，会交给底层 libuv 处理，任务完成后将回调放入对应队列，事件循环按照不同阶段依次执行这些回调。其中包括 timers、poll、check 等阶段，同时微任务（process.nextTick、Promise）会优先于宏任务执行。


**一句话总结：**

> Node.js 事件循环就是：通过一个主线程不断检查任务队列，把已经完成的异步任务回调取出来执行，从而实现单线程下的高并发处理。 转换为 markdown 格式