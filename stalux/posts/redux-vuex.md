---
title: 《Reudx与Vuex全面深度对比：从入门到实战，前端状态管理一次搞懂》
tags:
    - Redux
    - Vuex
categories:
    - 前端
date: "2026-04-11 14:50:34 "
updated: "2026-04-24 00:00:00"
abbrlink: f8882802
---

# 一、前言：为什么我们需要状态管理？
在组件化开发的前端世界里，状态（State）本质上就是驱动视图变化的数据。而组件化开发天然带来了[组件间数据共享]的痛点：

- 父子组件传参可以通过Props实现，但跨层级、兄弟组件，甚至全局复用的状态，Props传递会出现「props 透传地狱」，维护成本极高；

- 非关联组件之间的状态同步，原生方案实现复杂，极易出现数据流向混乱、bug 难以追踪的问题；

- 复杂业务中，大量的状态分散在各个组件中，数据变更的逻辑与视图逻辑耦合，代码可维护性、可测试性急剧下降。

而 Redux 和 Vuex 就是为了解决这些问题而生的前端状态管理库，二者均基于 Flux 架构的「单向数据流」核心理念，分别是 React、Vue 生态中最主流的全局状态管理方案，帮我们把分散的全局状态收敛到统一的容器中管理，让数据变更可预测、可追踪、易维护。

# 二、Redux全面详解：React生态的状态管理标杆

Redux 是一个独立于 React 的、轻量级的 JavaScript 状态管理库，核心遵循函数式编程思想，凭借严格的数据流规则和强可预测性，成为 React 生态中最主流的状态管理方案。

## 2.1 Redux 三大核心设计原则

这三大原则是 Redux 的灵魂，也是理解 Redux 所有设计的前提，初学者必须吃透：

1. 单一数据源：整个应用的全局状态，都存储在唯一的一个 Store 树状对象中。整个应用只有一个 Store，便于调试、追踪状态变化，也方便实现持久化等通用能力。

2. State是只读的：你永远不能直接修改State，唯一改变state的方式，就是触发一个Action（描述发生了什么的普通对象），这保证了所有的状态改变都有统一的出口，变更时可以追踪，不会散落在比较隐式的地方。

3. 使用纯函数Reducer执行状态修改：Reducer是一个纯函数，接收旧的state和触发的Action，并且返回一个新的State，纯函数保证了相同的输入，永远得到相同的输出，没有任何的副作用，让状态变更可预测、易测试。

## 2.2 Redux核心概念拆解

初学者最容易被 Redux 的概念劝退，其实核心概念只有 5 个，每个概念的职责单一且清晰：

| 核心概念 | 是什么 | 核心职责 |

| :--- | :--- | :--- |

| Store | 整个应用的状态容器 | 整合 State、Action、Reducer，提供状态读取、更新监听、派发动作的能力，是 Redux 的核心入口 |

| State | 存储的状态数据 | 只读的纯对象，唯一数据源，视图根据 State 渲染，State 变化触发视图更新 |

| Action | 普通 JavaScript 对象 | 描述「要发生什么变更」的唯一载体，必须包含 type 字段（动作的唯一标识），可选携带 payload 字段（变更需要的参数） |

| Reducer | 纯函数 | 唯一能执行状态更新的地方，根据 Action 的 type，匹配对应的更新逻辑，基于旧 State 计算并返回全新的 State，禁止直接修改原 State |

| Dispatch | Store 提供的方法 | 触发状态更新的唯一方式，唯一的参数是 Action 对象，Store 接收到 Action 后，会调用 Reducer 执行状态更新 |

补充两个高频使用的衍生概念：

- Selector：从 State 中提取、派生数据的函数，相当于对 State 的查询，React 中通过 useSelector Hook 使用，可缓存计算结果，避免不必要的重渲染。

- 中间件（Middleware）：Redux 提供的扩展机制，原生 Redux 只支持同步的状态更新，中间件可以在 Action 派发后、到达 Reducer 前，拦截 Action 处理异步逻辑（比如接口请求），最常用的是 redux-thunk（内置在 Redux Toolkit 中）。

## 2.3 Redux单向数据流工作流程

Redux的核心概念是单向数据流，整个数据流转的路径固定且不可逆，这也是Redux可预测的核心来源，完整流程分为四步：

1. 用户交互触发动作：用户点击按钮、接口请求完成等场景，在组件中通过dispatch方法派发一个Action对象；

2. Store转发Action：Store接收Action后，将Action与当前State一起传递给Reducer函数；

3. Reducer执行状态更新：Reducer根据Action的type匹配对应逻辑，严格遵守不可变原则，返回一个全新的 State 对象，禁止修改原 State；

4. 视图更新：Store 检测到 State 发生变化，通知所有订阅了 State 的组件，组件读取最新的 State，重新渲染视图。

划重点：整个流程是单向的、线性的，所有的状态变更都有迹可循，调试时可以通过 Action 追溯到每一次状态变化的原因，这也是复杂项目中 Redux 的核心优势。

## 2.4 现代 Redux：Redux Toolkit（RTK）

原生 Redux 存在一个公认的痛点：样板代码过多。创建 Action、Reducer 需要写大量重复代码，手动处理不可变更新极易出错。

因此，Redux 官方推出了 Redux Toolkit（简称 RTK），作为现代 Redux 的标准编写方式，它封装了原生 Redux 的繁琐操作，内置了常用的中间件和工具，大幅简化了代码量，是目前所有 React 项目使用 Redux 的首选方案。

### RTK 核心 API：

- configureStore：封装了原生的 createStore，一键开启 Redux DevTools 调试、内置 redux-thunk 中间件，自动合并 Reducer；

- createSlice：核心 API，一键生成 Action Creator 和 Reducer，内置 Immer 库，让我们可以直接「修改」State，不用手动写不可变更新的代码；

- createAsyncThunk：封装异步请求的处理，自动生成 pending/fulfilled/rejected 三个 Action 状态，轻松处理异步流程；

- createSelector：封装派生状态的缓存逻辑，优化重渲染性能。

## 2.5 Redux 基础使用示例（RTK + React Hooks）

我们以最经典的计数器为例，展示完整的 Redux 使用流程，基于 React 函数组件 + Hooks 写法（目前 React 项目的主流写法）。

### 步骤 1：安装依赖

```bash

# 核心依赖：RTK 已经包含了 redux 核心，无需单独安装

npm install @reduxjs/toolkit react-redux

```

### 步骤 2：创建 Slice（切片，整合 Action + Reducer）

创建 src/store/counterSlice.js 文件，RTK 推荐按功能模块拆分 Slice，而不是把 Action 和 Reducer 拆分成多个文件：

```javascript

import { createSlice } from '@reduxjs/toolkit'

// 定义初始状态

const initialState = {

  count: 0,

  status: 'idle' // 用于异步状态管理

}

// 创建切片

export const counterSlice = createSlice({

  name: 'counter', // 切片名称，会自动作为 Action type 的前缀

  initialState, // 初始状态

  reducers: {

    // 同步 reducer：直接修改 state，immer 会处理不可变更新

 increment: (state) => {

      state.count += 1

    },

    decrement: (state) => {

      state.count -= 1

    },

    // 带参数的 reducer：payload 是 dispatch 时传入的参数

    incrementByAmount: (state, action) => {

      state.count += action.payload

    },

    reset: (state) => {

      state.count = 0

    }

  }

})

// 自动生成与 reducer 对应的 Action Creator

export const { increment, decrement, incrementByAmount, reset } = counterSlice.actions

// 导出 reducer，供 store 注册

export default counterSlice.reducer

```

### 步骤 3：创建 Store 并注入 React 应用

创建 src/store/index.js 文件：

```javascript

import { configureStore } from '@reduxjs/toolkit'

import counterReducer from './counterSlice'

// 创建全局 store

export const store = configureStore({

  reducer: {

    // 注册切片 reducer，state 中的 key 就是这里定义的 key

    counter: counterReducer

  }

})

```

在入口文件 src/main.jsx 中，通过 Provider 把 Store 注入整个应用：

```javascript

import React from 'react'

import ReactDOM from 'react-dom/client'

import { Provider } from 'react-redux'

import { store } from './store'

import App from './App'

ReactDOM.createRoot(document.getElementById('root')).render(

  <Provider store={store}>

    <App />

  </Provider>

)

```

### 步骤 4：在组件中使用 Redux 状态与更新方法

```javascript

import React from 'react'

import { useSelector, useDispatch } from 'react-redux'

// 导入生成的 Action Creator

import { increment, decrement, incrementByAmount, reset } from './store/counterSlice'

function Counter() {

  // 1. 从 Store 中读取需要的状态

  const { count } = useSelector((state) => state.counter)

  // 2. 获取 dispatch 方法，用于触发 Action

  const dispatch = useDispatch()

  return (

    <div>

      <h2>计数器：{count}</h2>

      <div>

        <button onClick={() => dispatch(decrement())}>减1</button>

        <button onClick={() => dispatch(increment())}>加1</button>

        <button onClick={() => dispatch(incrementByAmount(5))}>加5</button>

 <button onClick={() => dispatch(reset())}>重置</button>

      </div>

</div>

  )

}

export default Counter

```

### 补充：异步处理示例（RTK 处理接口请求）

RTK 通过 createAsyncThunk 处理异步逻辑，我们模拟一个异步加数字的场景：

```javascript

// 在 counterSlice.js 中新增

import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'

// 1. 创建异步 thunk，模拟接口请求

export const incrementAsync = createAsyncThunk(

  'counter/incrementAsync',

  async (amount) => {

    // 模拟异步请求：2秒后返回结果

    await new Promise((resolve) => setTimeout(resolve, 2000))

    return amount

  }

)

// 2. 在 createSlice 中新增 extraReducers，处理异步状态

export const counterSlice = createSlice({

  name: 'counter',

  initialState,

  reducers: { /* 原有同步 reducer 不变 */ },

  // 处理异步 thunk 生成的 action

  extraReducers: (builder) => {

    builder

      .addCase(incrementAsync.pending, (state) => {

        state.status = 'loading'

      })

      .addCase(incrementAsync.fulfilled, (state, action) => {

        state.status = 'idle'

        state.count += action.payload

      })

      .addCase(incrementAsync.rejected, (state) => {

        state.status = 'error'

      })

  }

})

// 3. 组件中使用

dispatch(incrementAsync(10)) // 调用异步方法

```

# 三、Redux VS Vuex 核心维度深度对比

## 3.1 核心概念映射表

先通过一张表，快速建立两个库的概念对应关系，避免学习时混淆：

| 核心能力 | Redux（RTK） | Vuex |

| :--- | :--- | :--- |

| 状态容器 | Store | Store |

| 全局状态数据 | State | State |

| 派生 / 计算状态 | Selector（useSelector/createSelector） | Getter |

| 状态更新的唯一入口 | Reducer | Mutation |

| 触发状态更新的方法 | Dispatch（派发 Action） | Commit（触发 Mutation） |

| 异步 / 业务逻辑处理 | Thunk / createAsyncThunk / 中间件 | Action |

| 模块化拆分 | Slice / combineReducers | Module |

## 3.2 底层设计与核心理念差异（最本质区别）

这是两者所有差异的根源，完全贴合各自所属框架的设计哲学：

Redux：核心是函数式编程思想 + 不可变数据（Immutable）。它遵循严格的不可变原则，Reducer 必须返回一个全新的 State 对象，不能直接修改原 State。这完全贴合 React 的更新逻辑 ——React 是通过引用的变化来判断组件是否需要重渲染，不可变数据可以精准地追踪状态变化，也让时间旅行、状态回溯等调试能力成为可能。Redux 本身是框架无关的，它可以在 Vue、甚至原生 JS 项目中使用，只是和 React 生态的契合度最高。

Vuex：核心是响应式编程思想 + 可变数据（Mutable）。它深度绑定 Vue 的响应式系统，在 Mutation 中可以直接修改 State 的属性，Vue 会自动通过 Object.defineProperty（Vue2）/ Proxy（Vue3）追踪数据变化，自动触发视图更新。Vuex 是强耦合 Vue 的，只能在 Vue 项目中使用，完全贴合 Vue 的易用性理念，把底层的响应式细节封装起来，开发者只需要关注业务逻辑，不用手动处理数据的不可变更新。

## 3.3 状态修改方式差异

| 特性 | Redux（RTK） | Vuex |

| :--- | :--- | :--- |

| 能否直接修改 State | 严格禁止，原生 Redux 必须返回新 State，RTK 内置 Immer 实现「写法上的修改」，底层还是生成新 State | 组件中直接修改会生效，但严格禁止，官方要求必须通过 Mutation 修改 |

| 修改状态的唯一方式 | Dispatch 派发 Action，通过 Reducer 生成新 State | Commit 触发 Mutation，在 Mutation 中直接修改 State |

| 不可变要求 | 底层严格要求不可变，RTK 简化了写法，不用手动处理 | 无不可变要求，依托响应式系统，直接修改即可 |

| 对异步修改的支持 | 原生不支持，需要通过 thunk/saga 等中间件处理，RTK 内置了 thunk | 原生支持，明确拆分：Action 处理异步，Mutation 处理同步 |

## 3.4 异步处理方案差异

异步处理是状态管理中最常用的能力，两者的设计差异极大：

Redux：原生 Redux 只支持同步的 Action 派发，Reducer 作为纯函数，绝对不能包含异步逻辑。所有的异步操作必须在 Action 派发之前，或者通过中间件拦截 Action 来处理。

- 最基础的方案是 redux-thunk（RTK 内置），让我们可以派发一个函数，在函数里执行异步逻辑，完成后再 dispatch 真正的 Action；

- 复杂异步场景可以用 redux-saga、redux-observable 等中间件，支持取消、防抖、竞态处理等高级能力，灵活性极强，但学习成本也更高。

Vuex：原生就内置了异步处理方案，做了严格的职责拆分：

- Mutation 必须是同步函数，绝对不能写异步代码；

- Action 天生支持异步（async/await），可以在里面写任意异步逻辑，完成后通过 commit 触发 Mutation 修改状态。对新手极其友好，不用引入额外的库，不用理解中间件的概念，按照规则写即可，满足绝大多数业务场景的异步需求。