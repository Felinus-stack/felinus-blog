---
title: 《JS 异步编程、Promise及事件循环详解》
tags:
    - 前端
    - JS
    - 异步编程
    - Promise
    - 事件循环
categories:
    - 前端
    - JS
date: " 2025-12-26 17:55:44"
updated: "2026-04-24 00:00:00"
abbrlink: f8882809
---

# JS 异步编程、Promise及事件循环详解

## 一、JS 中的异步编程方式及场景区分

根据项目不同场景，选择合适的异步编程方式，具体区分如下：

- 简单异步任务：使用 Promise 或 async/await

- 事件驱动场景（如 UI 交互）：使用事件监听

- 组件通信场景：使用发布订阅模式

- 老项目维护：使用回调函数（兼容历史代码）

### 技术选型考量

- 代码可读性：async/await > promise > 回调函数

- 错误处理：Promise / async / await 方式更加统一，便于管理

- 浏览器兼容性：回调函数兼容性最好，无需额外兼容处理

## 二、Promise 详解（定义、状态及方法）

### 1. 什么是 Promise？

Promise 是 JS 异步编程的一种解决方案，核心作用是解决回调地狱问题，让异步代码的编写和维护更简洁、更具可读性。

### 2. Promise 的三种状态

- pending：初始化状态，异步操作未完成，既未成功也未失败

- fulfilled：异步操作成功完成的状态

- rejected：异步操作失败的状态

### 3. Promise 的核心特点

1. 状态转换不可逆：只能从 pending 状态转换为 fulfilled 或 rejected 状态，一旦状态确定，无法再改变。

2. 状态独立：一个 Promise 的状态变化不会影响其他 Promise 实例。

### 4. Promise 的方法

#### （1）实例方法（挂载在 Promise 实例上）

- then()：接收 fulfilled 状态的回调函数，可链式调用，返回一个新的 Promise 实例。

- catch()：接收 rejected 状态的回调函数，用于捕获异步操作的错误。

- finally()：无论异步操作成功（fulfilled）还是失败（rejected），都会执行的回调函数，不接收状态参数。

#### （2）静态方法（挂载在 Promise 构造函数上）

- Promise.resolve(value)：返回一个状态为 fulfilled 的 Promise 实例，value 可直接作为成功结果。

- Promise.rejected(reason)：返回一个状态为 rejected 的 Promise 实例，reason 为失败原因。

- Promise.all(iterable)：接收一个可迭代对象（如数组、Set），只有当所有 Promise 实例都变为 fulfilled 状态，才返回成功结果（数组形式）；只要有一个实例变为 rejected 状态，立即返回该失败原因。

- Promise.allSettled(iterable)：接收一个可迭代对象，等待所有 Promise 实例都完成决议（无论 fulfilled 还是 rejected），返回一个数组，包含每个 Promise 的详细结果（状态及对应值/原因）。

## 三、JS 中的事件循环机制

### 1. 核心定义

事件循环（Event Loop）是 JS 实现异步编程的核心机制，用于协调同步代码、微任务、宏任务的执行顺序，解决 JS 单线程无法同时处理多个任务的问题。

### 2. 核心执行顺序

执行规则可总结为：同步代码 > 微任务 > 宏任务 > 页面渲染 > 重复循环

#### （1）同步代码

优先执行全局同步代码，如 console.log()、变量声明、函数定义等，执行完毕后再处理异步任务。

#### （2）微任务（Microtask）

同步代码执行完成后，立即执行所有微任务，常见类型：

- Promise 的 then()、catch()、finally() 回调函数

- async/await 中 await 后续的代码（await 关键字会暂停异步函数，等待 Promise 决议后，执行后续代码，属于微任务）

- 浏览器环境：MutationObserver（监听 DOM 变化的 API）

- queueMicrotask() 方法手动添加的微任务

#### （3）宏任务（Macrotask）

所有微任务执行完成后，执行所有宏任务，常见类型：

- 定时器：setTimeout、setInterval

- 其他：setImmediate（Node 环境）、I/O 操作（如文件读取、接口请求）、UI 渲染（浏览器环境）

### 3. 事件循环完整流程

1. 执行全局同步代码，遇到异步任务（微任务、宏任务）时，分别放入对应任务队列。

2. 全局同步代码执行完毕后，清空所有微任务队列（按顺序执行所有微任务）。

3. 微任务执行完毕后，执行所有宏任务队列中的宏任务（按顺序执行）。

4. 单个宏任务执行完毕后，再次清空所有微任务队列。

5. 重复步骤 3、4，形成循环，直至所有任务执行完毕。
