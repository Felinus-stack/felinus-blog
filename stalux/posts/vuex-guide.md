---
title: 《Vue2 Vuex 核心用法详解》
tags:
    - 前端
    - Vue2
    - Vuex
categories:
    - 前端
    - Vuex
    - Vue2
date: "2026-01-26 14:12:05"
updated: "2026-04-24 00:00:00"
abbrlink: f8882806
---

# Vue2 Vuex 核心用法实操（含分模块）

## 1. 创建Vue2项目（终端操作）

通过终端命令创建Vue2项目，后续修改项目内容以实现Vuex状态管理：

```bash

vue create 项目名称

```

创建完成后，重点修改`store/index.js`文件，编写Vuex状态管理核心内容。

## 2. Vuex核心配置（store/index.js）

### 2.1 导入相关依赖

```javascript

import Vue from 'vue'

import Vuex from 'vuex'

import user from './modules/user'

import setting from './modules/setting'

```

### 2.2 安装Vuex插件

```javascript

Vue.use(Vuex)

```

### 2.3 Vuex核心配置内容

```javascript

const store = new Vuex.Store({

  // 严格模式 (有利于初学者，检测不规范的代码 => 上线时需要关闭)

  strict: true,

  // 1. 通过 state 可以提供数据 (所有组件共享的数据)

  state: {

    title: '仓库大标题',

    count: 100,

    list: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

  },

  // 2. 通过 mutations 可以提供修改数据的方法

  mutations: {

    // 所有mutation函数，第一个参数，都是 state

    // 注意点：mutation参数有且只能有一个，如果需要多个参数，包装成一个对象

    addCount (state, obj) {

 console.log(obj)

      // 修改数据

      state.count += obj.count

    },

    subCount (state, n) {

      state.count -= n

    },

    changeCount (state, newCount) {

      state.count = newCount

    },

    changeTitle (state, newTitle) {

      state.title = newTitle

    }

  },

  // 3. actions 处理异步

  // 注意：不能直接操作 state，操作 state，还是需要 commit mutation

  actions: {

    // context 上下文 (此处未分模块，可以当成store仓库)

    // context.commit('mutation名字', 额外参数)

    changeCountAction (context, num) {

      // 这里是setTimeout模拟异步，以后大部分场景是发请求

      setTimeout(() => {

        context.commit('changeCount', num)

      }, 1000)

    }

  },

  // 4. getters 类似于计算属性

  getters: {

    // 注意点：

    // 1. 形参第一个参数，就是state

    // 2. 必须有返回值，返回值就是getters的值

    filterList (state) {

      return state.list.filter(item => item > 5)

    }

  },

  // 分模块配置（引入modules下的user和setting模块）

  modules: {

    user,

    setting

  }

})

export default store

```

## 3. Vuex核心数据（state）详解

### 3.1 核心数据定义

```javascript

// 1. 通过 state 可以提供数据 (所有组件共享的数据)

state: {

  title: '仓库大标题',

  count: 100,

  list: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

}

```

### 3.2 数据获取方式

想要获取里面的数据，原生方式必须通过 `$store.state.属性名`，例如：`$store.state.count`、`$store.state.list`（正如 component 组件文件夹下的 Son1.vue 所示）。

每次通过 `$store.state` 调用较为繁琐，为简化代码，可引入 `mapState` 辅助函数，通过数组传递属性名：

```javascript

import { mapState } from 'vuex'

computed: {

  ...mapState(['count'])

}

```

这样获取数据可直接通过 `count` 调用，大幅简化代码。

## 4. Vuex核心方法详解

### 4.1 mutations

mutations 是修改 Vuex 中 state 状态的唯一合法途径，`this.$store.state.count = 1` 这种直接修改方式是不合法的，且 mutations 必须是同步函数。

#### 定义示例

```javascript

addCount (state, obj) {

  console.log(obj)

  // 修改数据

  state.count += obj.count

}

```

#### 关键注意点

1. 所有 mutation 函数的第一个参数必须是 `state`；

2. 第二个参数是传递的形参，若需多个参数，需包装成一个对象。

#### 调用方式

在 Son1.vue 文件中，原生方式通过 `$store.commit` 调用，第一个参数是 mutations 中定义的函数名，第二个参数是传递的实参：

```javascript

this.$store.commit('addCount', {

  count: n,

  msg: '哈哈'

})

```

为简化使用，可引入 `mapMutations` 辅助函数：

```javascript

import { mapMutations } from 'vuex'

methods: {

  ...mapMutations(['addCount'])

}

```

调用时可直接通过 `this.addCount()` 函数进行，无需重复写 `$store.commit`。

### 4.2 actions

actions 是处理异步操作的专用方式，大多数场景下用于向后端请求数据，此处通过定时器模拟异步请求。

#### 定义示例

```javascript

actions: {

  // context 上下文 (此处未分模块，可以当成store仓库)

  // context.commit('mutation名字', 额外参数)

  changeCountAction (context, num) {

    // 这里是setTimeout模拟异步，以后大部分场景是发请求

    setTimeout(() => {

      context.commit('changeCount', num)

    }, 1000)

  }

}

```

#### 关键注意点

1. 第一个参数 `context`：是 Store 实例的上下文对象，并非 store 本身，但包含了操作 Vuex 仓库的所有核心属性和方法，是 action 与仓库交互的唯一入口；

2. 第二个参数是传递的形参，用于异步操作所需数据。

#### 调用方式

原生方式通过 `this.$store.dispatch` 调用，第一个参数是 actions 中定义的方法名，第二个参数是实参：

```javascript

this.$store.dispatch('changeCountAction', 666)

```

同理，可引入 `mapActions` 辅助函数简化调用：

```javascript

import { mapActions } from 'vuex'

methods: {

  ...mapActions(['changeCountAction'])

}

```

调用时直接通过 `this.changeCountAction()` 进行即可。

### 4.3 getters

类比 Vue 组件内部的 computed 计算属性，核心作用是对已有的 state 进行数据过滤、格式转换、多状态组合计算等处理，生成新的派生数据，同时实现数据复用和缓存优化，是 Vuex 中处理状态派生的标准方式。

#### 定义示例

```javascript

getters: {

  // 注意点：

  // 1. 形参第一个参数，就是state

  // 2. 必须有返回值，返回值就是getters的值

  filterList (state) {

    return state.list.filter(item => item > 5)

  }

}

```

#### 关键注意点

1. 函数第一个参数必须是 `state`；

2. 必须有返回值，返回值即为 getters 的值。

#### 调用方式

原生方式通过 `$store.getters.方法名` 调用（无需加括号）：

```javascript

this.$store.getters.filterList

```

同理，可引入 `mapGetters` 辅助函数简化调用：

```javascript

import { mapGetters } from 'vuex'

computed: {

  ...mapGetters(['filterList'])

}

```

调用时直接通过 `this.filterList` 进行即可。

## 5. Vuex分模块用法（模块化管理）

将 Vuex 所有内容放在一个文件中，不利于后期管理和维护，因此需要进行分模块处理。

具体操作：在 `store` 文件夹下创建 `modules` 文件夹，存放各模块文件（此处以 `setting.js` 和 `user.js` 为例），模块定义及属性获取方式与根仓库略有差异。

### 5.1 模块定义（以user模块为例）

```javascript

// user模块（store/modules/user.js）

const state = {

  userInfo: {

    name: 'zs',

    age: 18

  },

  score: 80

}

export default {

  state

}

```

### 5.2 模块数据获取方式

#### 原生方式

获取模块中的数据，需在 `state` 后加上模块文件名，例如：

```javascript

$store.state.user.userInfo.name

```

#### 简化方式

1. 若通过 `mapState(['user'])` 获取，使用时仍需通过 `{{user.userInfo.name}}` 调用；

2. 更简洁的方式：通过 `...mapState('user', ['userInfo'])` 调用，获取属性时可直接使用 `{{ userInfo }}`：

```javascript

import { mapState } from 'vuex'

computed: {

  ...mapState('user', ['userInfo'])

}

```

#### 补充说明

获取模块中的 mutations、actions 方法，与获取 state 逻辑一致，在辅助函数中指定模块名即可。
