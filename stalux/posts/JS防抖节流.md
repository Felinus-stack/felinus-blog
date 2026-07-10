---
title: 《JavaScript防抖与节流详解》
tags:
    - JavaScript
    - 防抖
    - 节流
    - 性能优化
    - 前端
categories:
    - 前端开发
    - JavaScript
date: "2026-04-1 11:34:58"
updated: "2026-07-26 11:34:58"
abbrlink: f888288
---
# JavaScript 防抖与节流详解

> 本文详细介绍 JavaScript 中的防抖（Debounce）和节流（Throttle）机制，包括：
>
> - 为什么需要防抖和节流
> - 防抖原理
> - 防抖函数实现
> - 防抖应用场景
> - 节流原理
> - 节流函数实现
> - 节流应用场景
> - Lodash 中的防抖和节流


---

# 一、为什么需要防抖和节流？


在前端开发过程中，经常会遇到一些**高频触发事件**：

例如：

- input 输入
- scroll 滚动
- resize 窗口变化
- mousemove 鼠标移动
- button 连续点击


这些事件可能会在短时间内被大量触发。


例如：

用户输入：

```text
hello
```


input 事件可能触发：

```text
h

he

hel

hell

hello
```


如果每次输入都发送请求：

```text
输入一次

↓

发送一次请求
```


会造成：

- 大量 HTTP 请求
- 消耗服务器资源
- 浏览器性能下降
- 页面卡顿


因此需要：

- 防抖（Debounce）
- 节流（Throttle）


来控制函数执行频率。


---

# 二、防抖（Debounce）


## 1. 什么是防抖？


防抖核心思想：

> 当事件被频繁触发时，只在最后一次触发后等待指定时间执行函数。如果等待期间再次触发，则重新计时。


可以类比：

## 电梯


过程：

```text
第一个人进入电梯

↓

等待5秒关闭

↓

如果5秒内有人进入

↓

重新等待5秒

↓

最终停止进入后关闭
```


防抖：

```text
多次触发

↓

重新计时

↓

最后一次触发后执行
```


---

# 2. 防抖应用场景


## 场景一：搜索框输入提示


例如：

搜索框：

```html
<input type="text" placeholder="请输入内容">
```


没有防抖：

```javascript
const input = document.querySelector("input");


input.addEventListener(
"input",
(e)=>{

console.log(
"当前输入内容:",
e.target.value
);

});
```


执行过程：

```text
输入一个字符

↓

触发input事件

↓

发送请求
```


问题：

搜索提示一般需要：

```text
用户输入结束

↓

发送一次请求
```


而不是：

```text
输入一个字符

↓

请求一次
```


因此需要防抖。


---

# 场景二：窗口 resize


示例：

```javascript
function changeBox(){

const width = window.innerWidth;

console.log(width);

}


window.addEventListener(
"resize",
changeBox
);
```


问题：

调整窗口大小时：

```text
resize

resize

resize

resize
```


大量触发。


如果函数中包含：

- DOM 操作
- 计算
- 页面布局更新


会导致：

- CPU 占用增加
- 页面卡顿


---

## 为什么初始化需要执行一次 changeBox？


原因：

初始页面加载时：

```text
没有触发resize事件

↓

changeBox没有执行

↓

样式没有初始化
```


所以需要：

```javascript
changeBox();
```


主动执行一次。


---

# 场景三：表单验证


无防抖：

```javascript
function check(){

console.log("验证");

}


txt.addEventListener(
"input",
check
);
```


问题：

用户输入过程中：

```text
输入一次

↓

验证一次
```


导致：

- 高频执行
- 浪费资源
- 页面卡顿


---

# 三、防抖实现原理


防抖需要两个核心知识：


## 1. 定时器


作用：

延迟执行。


例如：

```javascript
setTimeout(()=>{

},1000)
```


---

## 2. 闭包


## 什么是闭包？


闭包：

> 内部函数可以访问外部函数作用域中的变量，即使外部函数已经执行完成。


防抖中：

需要保存：

```javascript
timer
```


让多次触发时：

共享同一个 timer。


这样才能：

```text
清除上一次定时器

↓

重新计时
```


---

# 四、防抖函数实现


```javascript
function debounce(fn, delay){

    let timer;


    return function(e){

        clearTimeout(timer);


        timer=setTimeout(()=>{

            fn.call(this,e);

        },delay);

    };

}
```


---

# 1. 防抖执行过程


第一次触发：

```text
timer不存在

↓

创建定时器

↓

等待delay
```


再次触发：

```text
清除旧timer

↓

重新创建timer

↓

重新等待delay
```


最终：

停止触发：

```text
等待delay

↓

执行函数
```


---

# 五、call 方法


防抖中：

```javascript
fn.call(this,e)
```


作用：

改变函数执行时的 this 指向。


语法：

```javascript
函数名.call(
thisArg,
参数1,
参数2
)
```


例如：

```javascript
fn.call(obj)
```


表示：

fn 内部：

```javascript
this=obj
```


---

# 六、立即执行版防抖


普通防抖：

第一次触发：

等待 delay 后执行。


有时候需要：

第一次立即执行。


实现：


```javascript
function debounce(fn,delay,flag){

let timer=null;


return function(e){


clearTimeout(timer);


if(!timer && flag){

    fn.call(this,e);


}else{


timer=setTimeout(()=>{

    fn.call(this,e);

},delay);


}


}

}
```


---

# 七、防抖实际应用


## 输入框防抖


```javascript
function handleInput(e){

console.log(e.target.value);

}


const input=document.querySelector("input");


const debouncedInput =
debounce(
handleInput,
1000,
true
);


input.addEventListener(
"input",
debouncedInput
);
```


---

## resize 防抖


```javascript
const debouncedBox =
debounce(
changeBox,
500,
false
);
```


---

## 表单验证防抖


```javascript
const debouncedCheck =
debounce(
check,
300,
false
);
```


---

# 八、节流（Throttle）


## 1. 什么是节流？


节流核心思想：

> 限制函数执行频率，在指定时间间隔内，无论触发多少次，只执行一次。


类似：

## 公交车


例如：

```text
公交车15分钟一班

↓

期间来了很多人

↓

也只能等待下一班
```


节流：

```text
持续触发

↓

固定时间执行一次
```


---

# 九、节流应用场景


## 场景一：按钮提交


问题：

用户快速点击：

```text
提交

提交

提交
```


可能造成：

- 重复提交
- 数据重复
- 服务器压力增加


---

## 场景二：scroll 滚动


无节流：

```javascript
window.addEventListener(
"scroll",
handleScroll
);
```


问题：

滚动过程中：

```text
scroll

scroll

scroll

scroll
```


持续执行。


导致：

- CPU 消耗高
- 页面卡顿


---

# 十、节流实现


## 方法一：定时器方式


```javascript
function throttle(fn,wait){

let timer=null;


return function(e){


if(!timer){


timer=setTimeout(()=>{


timer=null;


fn.call(this,e);


},wait);


}


};


}
```


---

执行过程：

```text
timer=null

↓

第一次执行

↓

创建timer

↓

等待wait

↓

timer=null

↓

允许下一次执行
```


---

# 方法二：时间戳方式


```javascript
function throttle(fn,delay){


let lastTime=0;


return function(e){


const now=Date.now();



if(now-lastTime>=delay){


lastTime=now;


fn.call(this,e);


}


};


}
```


---

原理：

记录：

```text
上一次执行时间
```


每次触发：

计算：

```text
当前时间 - 上次执行时间
```


如果：

```text
>= delay
```


执行函数。


---

# 十一、节流实际应用


## 提交按钮节流


```javascript
const throttledSubmit =
throttle(
submitForm,
2000
);
```


---

## scroll 节流


定时器：

```javascript
const throttledHandleScroll =
throttle(
handleScroll,
100
);
```


时间戳：

```javascript
const throttledScroll =
throttle(
handleScroll,
1000
);
```


---

# 十二、为什么 throttle 中需要提前处理事件？


例如：

```javascript
setTimeout(()=>{

submitForm(e)

},2000)
```


问题：

事件对象 e：

只在同步阶段有效。


异步执行时：

浏览器可能已经回收。


所以：

应该提前：

```javascript
e.preventDefault();
```


然后：

异步执行后续逻辑。


---

# 十三、Lodash 中的防抖和节流


实际项目中：

通常直接使用 lodash。


官网：

https://lodash.com/


---

## 防抖


```javascript
const debouncedInput =
_.debounce(
handleInput,
1000
);
```


---

## 节流


```javascript
const throttledInput =
_.throttle(
handleScroll,
1000
);
```


---

# 十四、防抖与节流区别


|区别|防抖|节流|
|-|-|-|
|核心|停止触发后执行一次|固定时间执行一次|
|执行次数|最后一次|周期执行|
|适合场景|输入搜索|滚动监听|
|关注点|减少执行次数|限制执行频率|


---

# 十五、面试回答


## 什么是防抖？


回答：

> 防抖是指当事件连续触发时，不立即执行函数，而是等待一定时间。如果等待期间再次触发，则重新计时，最终只执行最后一次操作。常用于搜索框输入、窗口 resize 等场景。


---

## 什么是节流？


回答：

> 节流是指限制函数执行频率，在固定时间间隔内，无论事件触发多少次，只执行一次。常用于 scroll 滚动监听、按钮防重复提交等场景。


---

# 总结


## 防抖

核心：

```text
频繁触发

↓

重新计时

↓

最后一次执行
```


常用：

- 搜索框
- resize
- 表单验证


---

## 节流

核心：

```text
持续触发

↓

固定时间执行一次
```


常用：

- scroll
- mousemove
- 防重复提交


掌握防抖和节流，是前端性能优化中的重要知识点，也是 JavaScript 高频面试内容。