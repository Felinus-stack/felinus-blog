---
title: 《React新手必看：5个最常用的Hooks，一片彻底搞懂》
tags:
    - 前端
    - React
    - Hooks
categories:
    - 前端
    - React
date: " 2026-03-09 15:30:07  "
updated: "2026-04-24 00:00:00"
abbrlink: f8882804
---

# 一、useState 组件的“状态存储器”

useState 它的作用是让组件拥有自己的"记忆"（状态），并能根据状态变化自动更新界面。

## 1. 核心用法

```typescript

import {useState} from 'react';

function Counter() {

    // count 是状态变量，setCount 是更新状态变量的函数

    const [count,setCount] = useState(0);// 初始值是0

    return (

        <div>

 <p>当前计数</p>

            <button onClick={() => setCount(count + 1)}>点我加1</button>

        </div>

    )

    const [user,setUser] = useState({name:'张三',age: 18});

    const updateAge = () => {

        setUser({...user,age:19})

    }

}

```

## 2. 关键知识点

- 状态更新是“替换”不是“合并”：如果你存的是对象，更新时要手动合并旧属性（用展开运算符...）

- 状态更新是异步的：不要在 setCount 之后立刻读取count，它还是旧值

- 函数式更新：如果新状态依赖旧状态，推荐用函数形式，确保拿到最新的旧状态

## 3. 错误示例与正确示例

### 错误示例：直接修改状态（对象/数组）

```typescript

const [user, setUser] = useState({ name: '张三', age: 18 });

// 错误：直接修改了原对象，React 检测不到变化

const updateAge = () => {

  user.age = 19; 

  setUser(user); 

};

```

### 正确示例：

```typescript

const [user, setUser] = useState({ name: '张三', age: 18 });

// 正确：创建一个新对象，React 会触发重渲染

const updateAge = () => {

  setUser({ ...user, age: 19 }); 

};

```

### 错误示例：依赖旧状态却不用函数式更新

```typescript

const [count, setCount] = useState(0);

// 错误：如果快速点击两次，count 可能只加了 1（因为异步）

const doubleAdd = () => {

  setCount(count + 1);

  setCount(count + 1); 

};

```

### 正确示例：

```typescript

const [count, setCount] = useState(0);

// 正确：函数参数 prevCount 永远是最新的旧状态

const doubleAdd = () => {

  setCount(prevCount => prevCount + 1);

  setCount(prevCount => prevCount + 1); 

};

```

# 二、useEffect：处理“副作用”的神器

useEffect是复杂的但也最强大的Hook。它的作用是处理组件渲染之外的逻辑（也就是“副作用”），比如：数据请求，DOM操作，定时器，订阅事件等。

## 1. 核心概念

副作用分为两种:

- 需要清理的：比如定时器，事件监听（不清理会内存泄露）

- 不需要清理的：比如修改文档标题，发送一次网络请求。

## 2. 基本语法

```typescript

import { useEffect } from 'react';

useEffect(() => {

  // 这里写副作用逻辑

  // （可选）返回一个清理函数

  return () => {

    // 这里写清理逻辑

  };

}, [依赖数组]); // 控制副作用什么时候执行

```

## 3. 重点拆解：依赖数组的3种情况

useEffect 什么时候执行，完全由第二个参数依赖数组决定

### 情况1：不传依赖数组

后果：组件每次更新（包括父组件传参变化，自己状态变化）都会执行，极易造成死循环。

### 情况2：传空数组[]

```typescript

useEffect(() => {

  console.log('只在组件挂载（第一次渲染）后执行一次');

  // 适合：发送初始数据请求、绑定全局事件

}, []);

```

执行时机：仅在组件“出生”时执行一次，“死亡”时执行清理函数

### 情况3：传具体的依赖[count,name]

```typescript

const [count, setCount] = useState(0);

const [name, setName] = useState('');

useEffect(() => {

  console.log('count 或 name 变化时执行');

  document.title = `点击了 ${count} 次`;

}, [count, name]); // 只要数组里的任一值变了，就重新执行

```

## 4. 重点拆解：闭包陷阱

这是React中最经典的坑。

### 错误示例：闭包导致拿到旧的值

```typescript

function Timer() {

  const [count, setCount] = useState(0);

  useEffect(() => {

    const timer = setInterval(() => {

      // 这里的 count 永远是初始值 0！

      // 因为 useEffect 只在挂载时执行了一次，它“捕获”了当时的 count

      console.log(count); 

    }, 1000);

    return () => clearInterval(timer);

  }, []); // 依赖数组为空

  return <div>{count}</div>;

}

```

### 解决方法 1：正确添加依赖

```typescript

useEffect(() => {

  const timer = setInterval(() => {

    console.log(count);

  }, 1000);

  return () => clearInterval(timer);

}, [count]); // 把 count 加进去，count 一变，定时器会销毁重建

```

### 解决方法 2：使用函数式更新（如果只是想改状态）

```typescript

useEffect(() => {

  const timer = setInterval(() => {

    // 不依赖外部的 count，直接用函数式更新拿最新值

    setCount(prevCount => prevCount + 1);

  }, 1000);

  return () => clearInterval(timer);

}, []); // 依然可以是空数组

```

## 5. 重点拆解：清理函数（新手第三大重灾区域）

如果你的副作用创建了一些 “持久化” 的东西（比如定时器、全局事件监听、WebSocket 连接），必须在组件销毁前清理掉，否则会内存泄漏或报错。

### 正确示例：清理定时器

```typescript

useEffect(() => {

  const timer = setInterval(() => {

    console.log('定时器在跑');

  }, 1000);

  // 重要：返回一个清理函数

  return () => {

    console.log('组件销毁或依赖变化，清理定时器');

    clearInterval(timer);

  };

}, []);

```

### 清理函数的时机：

1. 组件卸载（销毁）时

2. 副作用重新执行前（即依赖数组变化时，先清理旧的，再执行新的）

# 三、useRef:不仅是“DOM选择器”

useRef有两个核心用途：

1. 获取DOM元素（最常用）

2. 保存一个“不会触发重新渲染”（进阶用法）

## 1. 核心用法 1：获取 DOM

```typescript

import { useRef, useEffect } from 'react';

function InputFocus() {

  // 1. 创建一个 ref 对象

  const inputRef = useRef(null);

  useEffect(() => {

    // 3. 组件挂载后，inputRef.current 就指向了真实的 input 元素

    inputRef.current.focus(); // 让输入框自动聚焦

  }, []);

  // 2. 把 ref 绑到 DOM 元素上

  return <input ref={inputRef} type="text" />;

}

```

## 2. 核心用法 2：保存 “可变值”

useRef 保存的值，修改 .current 不会触发组件重新渲染。这一点和 useState 完全不同。

适用场景：保存定时器 ID、保存上一次的状态值

```typescript

function PreviousValue() {

  const [count, setCount] = useState(0);

  // 用 ref 保存上一次的值

  const prevCountRef = useRef(0);

  useEffect(() => {

    // 每次 count 变化后，把旧值存进 ref

    prevCountRef.current = count;

  }, [count]);

  return (

    <div>

      <p>现在的值：{count}</p>

      <p>上一次的值：{prevCountRef.current}</p>

      <button onClick={() => setCount(count + 1)}>加 1</button>

    </div>

 );

}

```

## 3. 新手必踩坑

- 不要在渲染过程中修改 .current（应该在 useEffect 或事件处理函数里改）。

- 记住是 .current：新手经常忘记写 current，直接用 ref 是取不到值的。

# 四、useCallback & useMemo：性能优化利器（但别滥用）

这两个Hook经常放在一起讲，因为它们都是用来做性能优化的，而且都依赖【依赖数组】。

核心原则：不要一开始就用它们！先写功能再根据性能瓶颈优化。

## 1. useCallback:缓存函数

作用：缓存一个函数，只有当依赖数组变化时，才会有新一个函数引用。

为什么需要它？如果父组件传给子组件一个函数，父组件每次重渲染，这个函数都会变成新的，导致子组件也跟着没必要地重渲染。

```typescript

import { useState, useCallback, memo } from 'react';

// 子组件：用 memo 包裹，只有 props 变化时才会重渲染

const Child = memo(({ onButtonClick }) => {

  console.log('子组件渲染了');

  return <button onClick={onButtonClick}>子组件按钮</button>;

});

function Parent() {

  const [count, setCount] = useState(0);

  // 用 useCallback 缓存函数

  const handleClick = useCallback(() => {

    console.log('按钮被点击了');

  }, []); // 依赖为空，所以这个函数引用永远不变

  return (

    <div>

      <p>父组件计数：{count}</p>

      <button onClick={() => setCount(count + 1)}>父组件加 1</button>

      {/* 传入缓存后的函数 */}

      <Child onButtonClick={handleClick} />

    </div>

  );

}

```

效果：点击父组件的 “加 1”，父组件重渲染，但子组件不会再渲染了（因为 handleClick 引用没变）。

## 2. useMemo：缓存计算结果

作用：缓存一个计算量很大的值，只有依赖数组变化时，才重新计算。

为什么需要它？避免每次渲染都进行昂贵的计算（比如遍历一个超大数组）。

```typescript

import { useState, useMemo } from 'react';

function ExpensiveCalc() {

  const [count, setCount] = useState(0);

  const [todos, setTodos] = useState([]);

  // 模拟一个很耗时的计算

  const expensiveResult = useMemo(() => {

    console.log('进行了一次耗时计算...');

    return count * 2;

  }, [count]); // 只有 count 变了，才重新计算

  return (

    <div>

      <p>耗时计算结果：{expensiveResult}</p>

      <button onClick={() => setCount(count + 1)}>改 Count</button>

      {/* 点击这个按钮，expensiveResult 不会重新计算 */}

      <button onClick={() => setTodos([...todos, '新任务'])}>加 Todo</button>

    </div>

  );

}

```

## 3. 新手必踩坑：滥用优化

- 不要什么都包：如果只是简单的函数或计算，用了 useCallback/useMemo 反而会增加代码复杂度，甚至可能因为比较依赖数组而更慢。

- 依赖数组不能错：和 useEffect 一样，依赖数组里必须包含所有用到的变量，否则会有闭包陷阱。

# 五、总结与学习建议

恭喜你！坚持看到这里，你已经掌握了 React 开发中 90% 的场景所需的 Hooks。

## 核心回顾

- useState：管状态，让组件 “记住” 东西。

- useEffect：管副作用（数据请求、定时器），重点搞定依赖数组、闭包陷阱、清理函数。

- useRef：拿 DOM 或存不需要触发渲染的值。

- useCallback/useMemo：性能优化，最后再用，别滥用。
