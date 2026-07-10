---
title: 《JavaScript事件冒泡与捕获机制》
tags:
    - JavaScript
    - DOM
    - 事件处理
    - 前端
    - 事件冒泡
categories:
    - 前端开发
    - JavaScript
date: "2026-06-22 11:34:58"
updated: "2026-06-22 11:34:58"
abbrlink: f888289
---
# JavaScript 事件的冒泡与捕获机制详解

> 本文介绍 JavaScript DOM 事件传播机制，包括：
>
> - 事件冒泡
> - 事件捕获
> - 二者区别
> - 如何阻止事件传播
> - 支持冒泡与捕获的事件类型
>
> 掌握事件传播机制是理解事件委托、组件交互以及前端事件处理的重要基础。

---

# 一、事件冒泡与捕获概念

在浏览器中，当一个元素触发事件时，事件并不是只作用于当前元素，而是会按照一定规则在 DOM 树中传播。

事件传播主要分为两个阶段：

1. **事件捕获（Event Capturing）**
2. **事件冒泡（Event Bubbling）**

完整事件传播过程：

```text
事件捕获阶段

window

↓

document

↓

html

↓

body

↓

父元素

↓

子元素


事件目标阶段

↓

目标元素


事件冒泡阶段

↓

父元素

↓

body

↓

html

↓

document

↓

window
```

---

# 二、事件冒泡（Event Bubbling）


## 1. 什么是事件冒泡？

事件冒泡：

> 事件从目标元素开始触发，然后逐级向上传播到父元素，直到最顶层元素。

类似：

> 水中的气泡从底部向上浮动。


例如：

```html
<body>

<div class="grandpa">

    <div class="father">

        <div class="son"></div>

    </div>

</div>

</body>
```


结构：

```text
grandpa

└── father

    └── son
```


如果点击：

```text
son
```

事件传播：

```text
son

↓

father

↓

grandpa
```

---

# 2. 冒泡事件监听方式


使用：

```javascript
element.addEventListener(
    'event',
    function(){},
    false
)
```


第三个参数：

```javascript
false
```

表示：

事件冒泡。


默认值就是：

```javascript
false
```

所以可以省略。


---

# 3. 冒泡示例代码


```html
<body>

<div class="grandpa">

    <div class="father">

        <div class="son"></div>

    </div>

</div>


<script>


const grandpa = document.querySelector('.grandpa')

const father = document.querySelector('.father')

const son = document.querySelector('.son')



grandpa.addEventListener('click',function(){

    console.log('我是爷爷')

})


father.addEventListener('click',function(){

    console.log('我是爸爸')

})


son.addEventListener('click',function(){

    console.log('我是儿子')

})


</script>
```

---

## 点击不同元素结果


### 点击爷爷元素


执行：

```text
我是爷爷
```


---

### 点击爸爸元素


执行：

```text
我是爸爸

我是爷爷
```


原因：

点击爸爸后：

事件向父级冒泡。


---

### 点击儿子元素


执行：

```text
我是儿子

我是爸爸

我是爷爷
```


传播过程：

```text
son

↓

father

↓

grandpa
```

---

# 三、事件捕获（Event Capturing）


## 1. 什么是事件捕获？

事件捕获：

> 事件从最顶层父元素开始，逐级向下传播到目标元素。


类似：

> 水从顶部流向底部。


传播方向：

```text
grandpa

↓

father

↓

son
```


---

# 2. 捕获事件监听方式


使用：

```javascript
element.addEventListener(
    'event',
    function(){},
    true
)
```


第三个参数：

```javascript
true
```

表示：

事件捕获。


---

# 3. 捕获示例代码


```html
<body>

<div class="grandpa">

    <div class="father">

        <div class="son"></div>

    </div>

</div>


<script>


const grandpa = document.querySelector('.grandpa')

const father = document.querySelector('.father')

const son = document.querySelector('.son')



grandpa.addEventListener(
'click',
function(){

    console.log('我是爷爷')

},
true)



father.addEventListener(
'click',
function(){

    console.log('我是爸爸')

},
true)



son.addEventListener(
'click',
function(){

    console.log('我是儿子')

},
true)



</script>
```

---

# 4. 捕获执行结果


点击：

## 爷爷元素


输出：

```text
我是爷爷
```


---

点击：

## 爸爸元素


输出：

```text
我是爷爷

我是爸爸
```


---

点击：

## 儿子元素


输出：

```text
我是爷爷

我是爸爸

我是儿子
```


传播过程：

```text
grandpa

↓

father

↓

son
```


---

# 四、冒泡与捕获区别


| 特性 | 事件捕获 | 事件冒泡 |
|---|---|---|
|传播方向|从外层向内层|从内层向外层|
|传播顺序|父元素 → 子元素|子元素 → 父元素|
|触发时机|先发生|后发生|
|addEventListener参数|true|false|
|典型应用|统一处理外层事件|事件委托|


---

# 五、阻止事件冒泡与捕获


## 1. 为什么需要阻止事件传播？


例如：

页面：

```text
卡片

└── 按钮
```


给卡片绑定点击：

```javascript
card.onclick=function(){

}
```


按钮：

```javascript
button.onclick=function(){

}
```


点击按钮时：

可能同时触发：

- 按钮事件
- 卡片事件


此时需要阻止冒泡。


---

# 2. stopPropagation()


阻止事件传播：

```javascript
event.stopPropagation();
```


作用：

停止：

- 冒泡
- 捕获


---

# 六、阻止冒泡示例


代码：

```javascript
const grandpa =
document.querySelector('.grandpa')


const father =
document.querySelector('.father')


const son =
document.querySelector('.son')



grandpa.addEventListener('click',function(){

    console.log('我是爷爷')

})



father.addEventListener('click',function(event){

    console.log('我是爸爸')

    event.stopPropagation()

})



son.addEventListener('click',function(){

    console.log('我是儿子')

})
```


---

点击：

```text
father
```

输出：

```text
我是爸爸
```


不会继续：

```text
爷爷
```


---

点击：

```text
son
```

传播：

```text
son

↓

father

↓

停止
```


输出：

```text
我是儿子

我是爸爸
```


---

# 七、阻止捕获示例


```javascript
grandpa.addEventListener(
'click',
function(){

console.log('我是爷爷')

},
true)



father.addEventListener(
'click',
function(event){

console.log('我是爸爸')

event.stopPropagation()

},
true)
```


点击 son：


执行：

```text
我是爷爷

我是爸爸
```


之后停止：

不会执行：

```text
我是儿子
```


---

# 八、支持冒泡与捕获的事件


## 1. 支持事件传播的事件


|事件类型|事件|作用|
|-|-|-|
|鼠标事件|click|鼠标点击|
||dblclick|双击|
||mousedown/mouseup|鼠标按下和释放|
||mouseover/mouseout|进入和离开|
||mousemove|鼠标移动|
||mouseenter/mouseleave|进入和离开|
|键盘事件|keydown/keyup|键盘操作|
|表单事件|submit|表单提交|
||input|输入变化|
||change|值变化|
||focusin/focusout|焦点变化|
|触摸事件|touchstart/touchend|触摸开始结束|
||touchmove|触摸移动|
|其他事件|resize|窗口变化|
||scroll|滚动事件|

---

# 九、不支持冒泡的事件


## 焦点事件


|事件|作用|
|-|-|
|focus|获得焦点|
|blur|失去焦点|


特点：

不会冒泡。


---

## 媒体事件


|事件|作用|
|-|-|
|play|播放|
|pause|暂停|
|ended|播放结束|


特点：

只作用于媒体元素。


---

## 资源事件


|事件|作用|
|-|-|
|load|资源加载完成|
|error|资源加载失败|


例如：

图片：

```html
<img src="test.png">
```


加载失败：

触发：

```javascript
error
```

---

# 十、事件委托与冒泡


事件委托：

> 利用事件冒泡机制，将子元素事件统一交给父元素处理。


例如：

列表：

```html
<ul id="list">

<li>A</li>

<li>B</li>

<li>C</li>

</ul>
```


不需要：

给每个 li 绑定事件。


只需要：

```javascript
list.addEventListener(
'click',
function(event){

console.log(event.target)

}
)
```


优势：

- 减少事件监听数量
- 提高性能
- 动态元素也支持


---

# 十一、面试回答模板


## 什么是事件冒泡？


回答：

> 事件冒泡是指事件从目标元素开始，逐级向父元素传播的过程。例如点击子元素时，如果父元素也绑定了相同事件，父元素的事件处理函数也会被触发。


---

## 什么是事件捕获？


回答：

> 事件捕获是事件从最外层元素开始，逐级向目标元素传播的过程。通过 addEventListener 的第三个参数设置为 true 可以开启捕获模式。


---

## 如何阻止事件冒泡？


回答：

> 可以通过 event.stopPropagation() 方法阻止事件继续传播。该方法可以阻止事件冒泡，也可以阻止捕获阶段继续向下传播。


---

# 总结


JavaScript 事件传播机制：

```text
捕获阶段

window

↓

document

↓

父元素

↓

目标元素


↓

事件处理


↓

冒泡阶段

目标元素

↓

父元素

↓

document

↓

window
```


核心知识：

- 事件冒泡：子元素 → 父元素
- 事件捕获：父元素 → 子元素
- addEventListener 第三个参数控制传播方式
- stopPropagation() 阻止事件传播
- 事件委托依赖事件冒泡


掌握事件冒泡与捕获机制，是理解 DOM 事件模型、React/Vue 事件处理以及前端组件交互的基础。