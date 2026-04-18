---
title: 《vue2 生命周期》
tags:
    - 前端
    - Vue2
    - 生命周期
categories:
    - 前端
    - 生命周期
    - Vue2
date: " 2025-12-26 17:55:44"
updated: "2026-04-24 00:00:00"
abbrlink: f8882808
---

# Vue2 组件化与生命周期钩子

## 一、Vue2 的组件化

### 1. 什么是Vue2 的组件化？

把一个完整的页面，拆分成多个独立、可复用、功能单一的 “小模块（组件）”，每个组件只负责处理自己的视图、逻辑和样式，最后像 “搭积木” 一样，把这些小组件组合起来，形成完整的页面。

### 2. 为什么需要组件化？

如果不做组件化，整个页面的代码会全部写在一个.vue文件里，会出现代码臃肿、逻辑混乱、无法复用等问题。组件化的核心价值体现在：

- 复用性

- 可维护性

- 分工协作

- 降低复杂度

每个组件从 “被创建” 到 “渲染到页面”，再到 “数据更新重新渲染”，最后 “被销毁”（比如页面跳转），是一个完整的生命周期过程。我们需要在这个过程的关键节点执行自定义逻辑。

Vue2 的生命周期，是组件从 “创建-->挂载--> 更新--> 销毁”的完整时间轴，每个阶段Vue 会自动触发对应的钩子函数。

#### 生命周期流程图

https://www.cnblogs.com/qingheshiguang/p/14677198.html

## 二、示例代码

```html

<!DOCTYPE html>

<html lang="en">

  <head>

<meta charset="UTF-8" />

    <title>Vue-LifeClyle</title>

  </head>

  <style>

    .jing {

      font-size: 50px;

      font-weight: bolder;

    }

  </style>

  <body>

    <div id="app" class="jing">

      <p>{{message}}</p>

      <!-- Vue 内置组件，缓存组件实例，避免重复创建和销毁 -->

      <keep-alive>

        <jh-component msg="2026年1月1日" v-if="show"></jh-component>

 </keep-alive>

    </div>

  </body>

  <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>

<script>

    // 局部组件

    const haohao = {

      template: "<div>from haohao: {{msg}}</div>",

      props: ["msg"],

      deactivated () {

        console.log("component deactivated!");

 },

      activated () {

 console.log("component activated");

      },

    };

    const app = new Vue({

      el: "#app",

      data: function () {

        return {

          message: "jingjing",

          show: true, //控制子组件是否显示

        };

      },

      beforeCreate: function () {

        console.group("beforeCreate Vue实例创建前的状态————————————————————");

        const state = {

          el: this.$el,

          data: this.$data,

          message: this.message,

 };

        console.log(state);

 console.groupEnd();

      },

      created: function () {

        console.group("created Vue实例创建完毕后状态————————————————————");

        const state = {

          el: this.$el,

          data: this.$data,

          message: this.message,

        };

        console.log(state);

        console.groupEnd();

      },

      beforeMount: function () {

        console.group("beforeMount 挂载前状态————————————————————");

        const state = {

          el: this.$el,

          data: this.$data,

          message: this.message,

        };

        console.log(this.$el);

        console.log(state);

        console.groupEnd();

      },

      mounted: function () {

        console.group("mounted 挂载后状态————————————————————");

        const state = {

          el: this.$el,

          data: this.$data,

 message: this.message,

        };

        console.log(this.$el);

        console.log(state);

        console.groupEnd();

      },

      beforeUpdate: function () {

        console.group("beforeUpdate 更新前状态————————————————————");

 const state = {

          el: this.$el,

          data: this.$data,

          message: this.message,

        };

        console.log(this.$el);

        console.log(state);

        console.log(

          "beforeUpdate = " + document.querySelector("p").innerHTML

        );

        console.groupEnd();

      },

      updated: function () {

        console.group("updated 更新完成状态————————————————————");

        const state = {

          el: this.$el,

          data: this.$data,

          message: this.message,

        };

        console.log(this.$el);

        console.log(state);

        console.log(

          "beforeUpdate = " + document.querySelector("p").innerHTML

        );

        console.groupEnd();

      },

      beforeDestroy: function () {

        console.group("beforeDestroy 销毁前状态————————————————————");

        const state = {

          el: this.$el,

 data: this.$data,

          message: this.message,

        };

        console.log(this.$el);

        console.log(state);

        console.groupEnd();

      },

      destroyed: function () {

        console.group("destroyed 销毁完成状态————————————————————");

        const state = {

          el: this.$el,

          data: this.$data,

          message: this.message,

        };

        console.log(this.$el);

        console.log(state);

        console.groupEnd();

      },

      //注册局部组件

      components: {

        "jh-component": haohao,

      },

    });

  </script>

</html>

```

## 三、代码的结构

创建一个app 的Vue 根实例，将其挂载在id为 app 的Dom 元素上。局部注册一个组件名为 haohao 并在根实例中将其注册，使其可以在根实例的作用域上进行使用。

## 四、打开开发者工具开始进行测试

### 1. beforeCreate 和 created

![图片1](/img1.png)

- beforeCreate：data 和 el 均未初始化，值为 undefined

- created ：Vue 实例观测的对象data已经配置好了，可以得到 app.message 中的值，但Vue实例使用的根Dom 元素还没有初始化。

#### 使用场景：

初始化数据请求，调用后端接口获取页面初始化数据，无需等待 DOM；

```javascript

created() {

  this.fetchGoodsList();

},

methods: {

  async fetchGoodsList() {

    try {

      const res = await axios.get("/api/goods/list");

      this.goodsList = res.data.list;

    } catch (err) {

      console.error("请求商品列表失败：", err);

    }

  }

}

```

初始化数据，绑定不依赖DOM的自定义事件。

### 2. beforeMount 和 mounted

#### 打印结果说明：

![图片2](/img2.png)

- beforeMount 执行时：data 和 el 均已经初始化，但是从 {{message }}的展示情况看并没有渲染数据，说明DOM仅完成占位，需等到mounted 挂载阶段再渲染具体值。

- mounted执行时：此时 el 已经成功进行渲染并且挂载在实例上面。

并且可以在控制台观察到 `component activated` 被打印出来，说明子组件 jh-component 被 <keep-alive> 包裹，随着 el 的挂载而触发。子组件的显示是在父组件 mounted 之前完成的。

#### 类比说明：

父组件就像 “房子”，子组件是 “家具”，父组件会先搭好房子框架（beforeMount），然后把家具摆进去（子组件渲染显示），最后才宣布 “房子装修完成”（mounted)。

#### 测试操作：

在控制台输入 `app.show = false`，修改data中的值后，会触发 beforeUpdate 和 updated 钩子函数，同时 deactivated 钩子会触发，表示子组件已停用、进入缓存状态，子组件会消失。

```html

<keep-alive>

   <jh-component msg="2026年1月1日" v-if="show"></jh-component> 

</keep-alive>

```

#### mounted使用场景：

在控制台输入 `app.message = 'jingjing'`，验证以下场景：

![图片3](/img3.png)

- 操作 DOM（如初始化 echarts、swiper、地图等第三方插件）；

- 获取 DOM 尺寸 / 位置（如计算元素高度、滚动位置）；

- 绑定依赖 DOM 的事件监听（如自定义滚动事件）。

![图片4](/img4.png)

#### 验证结果：

beforeUpdate 和 updated 触发时，el 中的数据都已经渲染完毕，但根据控制台的打印信息 `beforeUpdate = jingjing` 而 `updated = haohao` 可知，只有当 updated 钩子被调用时，组件DOM才会被更新。

### 3. beforeDestroy 和 destroyed

#### 测试操作：

直接在控制台输入 `app.$destroy()` 即可将Vue实例销毁，父实例主动销毁后，子组件才会随之销毁。

#### 打印结果说明：

销毁前和销毁后的DOM实例没有明显变化，其核心变化体现在响应式关联的解除：

- 实例销毁后，根组件的响应式系统、子组件、DOM关联的所有元素均被解绑；

- 组件销毁的核心是解除响应式系统的关联，销毁子组件、终止所有生命周期，但根实例的DOM节点不会被主动删除；

- 实例销毁后，Vue实例关联的所有内容均已解绑，操作实例不会有任何响应。

#### beforeDestroy使用场景：

##### 场景1：清理定时器

```javascript

<template>

  <div>当前计数：{{ count }}</div>

</template>

<script>

export default {

  data() {

    return { count: 0, timer: null }

  },

  created() {

    this.timer = setInterval(() => {

      this.count++

    }, 1000)

  },

  beforeDestroy() {

    clearInterval(this.timer)

  }

}

</script>

```

##### 场景2：解绑全局自定义事件

```javascript

<script>

export default {

  mounted() {

    window.addEventListener('resize', this.handleResize);

    this.$bus.$on('refresh-data', this.refresh);

  },

  methods: {

    handleResize() { /* 窗口缩放逻辑 */ },

    refresh() { /* 数据刷新逻辑 */ }

  },

  beforeDestroy() {

    window.removeEventListener('resize', this.handleResize);

    this.$bus.$off('refresh-data', this.refresh);

  }

}

</script>

```

##### 场景3：销毁第三方库实例

```javascript

<template>

  <div id="echart-box"></div>

</template>

<script>

import * as echarts from 'echarts';

export default {

  data() {

    return { chart: null };

  },

  mounted() {

    // 初始化Echarts实例

    this.chart = echarts.init(document.getElementById('echart-box'));

    this.chart.setOption({ title: { text: '测试图表' } });

  },

  beforeDestroy() {

    // 销毁Echarts实例，释放所有资源

    if (this.chart) {

 this.chart.dispose();

      this.chart = null; // 置空避免后续误操作

    }

  }

}

</script>

```

## 五、补充：activated 和 deactivated 生命周期钩子

`activated` 和 `deactivated` 生命周期钩子，仅在被 <keep-alive> 包裹的组件中才会被触发。

### 1. activated 钩子

核心作用：缓存组件实例，组件从缓存中激活时触发。

- 第一次显示（页面加载时 show:true）：子组件先执行 created/mounted 等普通钩子，然后立刻执行 activated；

- 后续再次显示（比如先把 show 改为 false，再改回 true）：组件不会重新执行 created/mounted（因为实例被缓存了），只会执行 activated。

### 2. deactivated 钩子

当组件因 v-show="false" 等原因隐藏时触发（注意：此时组件实例没有被销毁，只是进入缓存状态），而非执行 beforeDestroy/destroyed。

## 六、总结：Vue2 生命周期钩子

| 生命周期钩子 | 阶段核心说明 | 原生 JS 类比 | 原生 JS 核心对应操作 |

|--------------|--------------|--------------|----------------------|

| beforeCreate | 组件实例刚创建，data/methods/props 未初始化，无法访问 | JS 对象初始化初期（仅创建空对象，无属性 / 方法） | 创建空对象，未定义任何数据 / 方法 |

| created      | 组件实例创建完成，data/methods/props 已初始化，无 DOM（$el 不存在） | JS 对象初始化完成（已定义属性 / 方法，未接触 DOM） | 为对象赋值数据、绑定方法，不操作 DOM |

| beforeMount  | 模板编译完成，$el 已生成但未挂载到页面真实 DOM | DOM 准备阶段（创建元素但未插入页面） | 创建 DOM 元素、拼接模板内容，未插入页面 |

| mounted      | 模板挂载到页面真实 DOM，可正常操作 DOM，是 DOM 操作最佳时机 | DOM 加载完成后（DOM 树构建完毕，元素可访问） | 监听 DOMContentLoaded，将元素插入页面并操作 |

| beforeUpdate | 响应式数据变化，DOM 仍为旧值（未更新） | 数据修改后、DOM 更新前 | 修改数据变量，未手动更新 DOM |

| updated      | DOM 已随数据变化完成更新，可获取最新 DOM 内容 | 数据修改后、手动更新 DOM 完成 | 根据新数据更新 DOM 内容 |

| destroyed    | 组件已销毁，响应式数据 / 事件 / 定时器被清理，DOM 不可用 | 资源清理完成（移除 DOM / 事件 / 定时器） | 清除定时器、移除事件监听、删除 DOM 元素 |

### 核心钩子使用场景总结

1. created 的主要使用场景：

- 初始化数据

- 发起无需依赖 DOM的网络请求

2. mounted 的主要使用场景：

- 操作 DOM（如初始化 echarts、swiper、地图等第三方插件）

- 获取 DOM 尺寸 / 位置（如计算元素高度、滚动位置）

- 绑定依赖 DOM 的事件监听（如自定义滚动事件）

3. beforeDestroy 的主要使用场景：

- 清理定时器

- 解绑全局自定义事件

- 销毁第三方库实例

