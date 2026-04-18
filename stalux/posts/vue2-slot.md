---
title: 《Vue2 插槽（Slot）全面讲解》
tags:
    - 前端
    - Vue2
    - 插槽
categories:
    - 前端
    - 插槽
    - Vue2
date: "2025-12-28 16:49:45"
updated: "2026-04-24 00:00:00"
abbrlink: f8882807
---

# Vue2 插槽（Slot）核心用法详解

## 一、核心概念

插槽的本质是子组件内部的一个占位符，它的作用是：允许父组件在使用子组件时，向子组件的指定位置插入任意内容（文本、HTML标签、Vue2组件），进而实现子组件内容的个性化定制，无需修改子组件本身的代码。

通俗来讲：子组件留下一个“坑位置”，父组件可以往里面添加不同的内容。

## 二、插槽的分类

插槽具体可以分为3类：默认插槽（匿名插槽）、具名插槽、作用域插槽。

### 1. 默认插槽（匿名插槽）

没有指定名称，一个子组件只能有一个默认插槽。

#### 语法规则

- 子组件中：使用``作为占位符，可在插槽中设置默认内容，当父组件不传递任何内容时，显示该默认内容。

- 父组件中：在使用子组件的标签内部，直接写入要插入的内容即可。

#### 代码演示

```html

<!-- 子组件：MyDefaultSlot.vue -->

<!-- 定义默认插槽，设置默认内容 -->







```

```html

<!-- 父组件：使用MyDefaultSlot子组件 -->

<!-- 1. 不传递内容：显示插槽默认内容 -->

    <my-default-slot></my-default-slot>

<!-- 2. 传递文本内容 -->

<my-default-slot>

      父组件传递的文本内容

</my-default-slot>

<!-- 3. 传递HTML标签 + 其他组件 -->

<my-default-slot>

这是父组件传递的p标签



</my-default-slot>





import MyDefaultSlot from './MyDefaultSlot.vue';

export default {

  components: {

    MyDefaultSlot

  }

}



```

### 2. 具名插槽

当子组件需要在多个位置填充内容，匿名插槽无法满足需求时，需使用具名插槽。通过给插槽指定唯一名称，实现多个不同位置的内容填充。

#### 语法规则

- 子组件中：使用``，通过不同的name值区分不同插槽。

- 父组件中：

  1. 旧写法：在`  2. 新写法：在`  注意：v-slot只能在template标签上使用。

#### 代码演示

```html

<!-- 子组件：MyNamedSlot.vue -->

<!-- 头部插槽：name="header" -->





<!-- 内容插槽：name="content" -->





<!-- 底部插槽：name="footer" -->









```

```html

<!-- 父组件：使用MyNamedSlot子组件 -->

<my-named-slot>

<!-- 新写法：v-slot:名称（缩写#名称）-->

我的个性化卡片标题



<!-- 缩写写法：#content -->

<template #

这是父组件填充的卡片内容

可以写任意多的内容



<!-- 旧写法：slot="footer" -->







</my-named-slot>





import MyNamedSlot from './MyNamedSlot.vue';

export default {

  components: {

    MyNamedSlot

  }

}



```

### 3. 作用域插槽

作用域插槽是插槽的高级用法，核心解决的问题是：父组件在填充插槽内容时，需要使用到子组件内部的数据（默认情况下，父组件无法直接访问子组件的数据，作用域插槽就是为了实现“子组件向父组件插槽传输数据”）。

#### 语法规则

- 子组件中：在`- 父组件中：使用`#### 代码演示

```html

<!-- 子组件：MyScopeSlot.vue -->

<!-- 定义作用域插槽：将子组件的user数据传递给父组件 -->

<slot 

 name="userItem" 

        v-for="(user, index) in userList" 

        :key="user.id"

        :userInfo="user"  

        :index="index"    

      >







export default {

  data() {

    return {

      userList: [

        { id: 1, name: "张三", age: 20 },

        { id: 2, name: "李四", age: 22 },

        { id: 3, name: "王五", age: 25 }

      ]

    };

  }

};



```

```html

<!-- 父组件：使用MyScopeSlot子组件，访问子组件数据 -->

<my-scope-slot>

<!-- 具名作用域插槽：#userItem="slotProps"，slotProps是接收数据的对象（名称可自定义） -->

<template #



          索引：{{ slotProps.index }} - 姓名：{{ slotProps.userInfo.name }} - 年龄：{{ slotProps.userInfo.age }}





</my-scope-slot>

<my-scope-slot>

<template #">

{{ index + 1 }}

{{ userInfo.name }}

{{ userInfo.age }}





</my-scope-slot>





import MyScopeSlot from './MyScopeSlot.vue';

export default {

  components: {

    MyScopeSlot

  }

}



```

## 总结

插槽的出现就是为了解决 “组件骨架固定，但内容需要灵活定制” 的问题，通过使用插槽，可以更加灵活地处理组件里面的内容，无需修改子组件源码即可实现个性化定制。
