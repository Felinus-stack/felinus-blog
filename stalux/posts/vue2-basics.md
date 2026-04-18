---
title: 《Vue2 核心概念与用法详解》
tags:
    - 前端
    - Vue2
categories:
    - 前端
    - Vue2
date: "2026-04-11 14:50:34 "
updated: "2026-04-24 00:00:00"
abbrlink: f8882805
---

# Vue2 核心概念与用法详解

## 1. 插值表达式

- 作用：在模板中显示数据

- 本质：将 `data` 或 `computed` 中的值渲染到 DOM

- 语法：`{{ 表达式 }}`

## 2. 常见指令

### 2.1 动态绑定 HTML 属性

- `v-bind` 简写 `:`

```html

:class="className"

:src="imgUrl"

:style="styleObject"

```

### 2.2 数据双向绑定

- `v-model`：实现表单元素与数据的双向绑定

### 2.3 列表循环

- `v-for`：用于遍历数组或对象

### 2.4 条件渲染

| 指令 | 特点 | 适用场景 |

| :--- | :--- | :--- |

| `v-if` | 真正创建/销毁 DOM 元素 | 条件变化不频繁的场景 |

| `v-show` | 通过 CSS `display` 属性控制显示/隐藏 | 需要频繁切换的场景 |

## 3. 计算属性与监听器

### 3.1 computed（计算属性）

- 特点：有缓存，依赖变化时重新计算

- 适用场景：基于已有数据计算得到新值

### 3.2 watch（监听器）

- 作用：监听数据变化并执行逻辑

- 常见场景：

  1. 发起异步请求

  2. 操作 DOM

  3. 实现防抖/节流

  4. 监听路由参数变化

## 4. 组件化开发基础

### 4.1 组件定义与注册

#### 全局组件

```javascript

Vue.component('my-button', {

  template: `<button>全局按钮</button>`

})

```

#### 局部组件

```javascript

new Vue({

  components: {

    'my-header': { template: `<h1>局部组件</h1>` }

  }

})

```

### 4.2 Props 与自定义事件

#### 父传子

```html

<!-- 父组件 -->

<child :msg="message"></child>

```

```javascript

// 子组件

props: {

  msg: String,

  count: {

    type: Number,

    required: true,

    default: 0

  }

}

```

#### 子传父

```javascript

// 子组件

methods: {

  send() {

    this.$emit('get-msg', '子组件数据')

  }

}

```

```html

<!-- 父组件 -->

<child @get-msg="handleMsg"></child>

```

### 4.3 插槽

#### 默认插槽

```html

<!-- 子组件 -->

<slot>默认内容</slot>

```

#### 具名插槽

```html

<!-- 子组件 -->

<div>

  <slot name="header"></slot>

  <slot></slot>

  <slot name="footer"></slot>

</div>

```

#### 作用域插槽

```html

<!-- 子组件 -->

<slot :item="item" :index="index"></slot>

```

```html

<!-- 父组件 -->

<child>

  <template v-slot="{ item, index }">

    <li>{{ item }} - {{ index }}</li>

  </template>

</child>

```

## 5. 生命周期钩子

| 钩子函数 | 特点 | 适用场景 |

| :--- | :--- | :--- |

| beforeCreate | 无法访问 data/props/methods，DOM 未生成 | - |

| created | 可访问 data/props/methods，DOM 未挂载 | 数据初始化、发送 Ajax 请求 |

| mounted | DOM 已完成，可操作真实 DOM | DOM 操作、第三方库初始化 |

| updated | 数据变化后视图更新完成 | 数据变化后的操作 |

| destroyed | 组件销毁完成 | 清除定时器、解绑事件 |

## 6. 组件通信

### 6.1 provide/inject

```javascript

// 父组件

provide() {

  return { theme: 'dark' }

}

// 子组件

inject: ['theme']

```

### 6.2 事件总线

```javascript

// bus.js

import Vue from 'vue'

export const EventBus = new Vue()

// 发送事件

EventBus.$emit('add', 1)

// 接收事件

EventBus.$on('add', val => console.log(val))

// 解绑事件

EventBus.$off('add')

```

## 7. 状态管理（Vuex）

### 7.1 核心概念

```javascript

const store = new Vuex.Store({

  state: { /* 状态数据 */ },

  getters: { /* 计算属性 */ },

  mutations: { /* 同步修改方法 */ },

  actions: { /* 异步操作 */ },

 modules: { /* 模块化 */ }

})

```

### 7.2 辅助函数

```javascript

computed: {

 ...mapState(['count']),

  ...mapGetters(['doubleCount'])

},

methods: {

  ...mapMutations(['increment']),

  ...mapActions(['asyncIncrement'])

}

```

## 8. Vue Router

### 8.1 基本配置

```javascript

const routes = [

  { path: '/home', component: Home },

 { path: '/about', component: About }

]

const router = new VueRouter({

  mode: 'hash',

  routes

})

```

### 8.2 动态路由

```javascript

{ path: '/user/:id', component: User }

```

### 8.3 参数传递

```html

<!-- 方式1：直接 URL -->

<router-link to="/user/100">用户详情</router-link>

<!-- 方式2：对象形式 -->

<router-link :to="{ name: 'user', params: { id: 100 } }">用户详情</router-link>

```

```javascript

// 方式3：编程式导航

this.$router.push({ name: 'user', params: { id: 100 } })

```

### 8.4 参数接收

```javascript

// 方式1：$route.params

this.$route.params.id

// 方式2：props

props: ['id']

```

---

版权声明：本文为CSDN博主「旺王雪饼 www」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/W2004828/article/details/157653642
