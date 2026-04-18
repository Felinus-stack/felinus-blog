---
title: 《JS 常用方法详解（字符串+对象+数组）》
tags:
    - 前端
    - JS
    - 字符串
    - 对象
    - 数组
categories:
    - 前端
    - JS
date: "2025-11-29 13:40:15"
updated: "2026-04-24 00:00:00"
abbrlink: f8882811
---

# JS 常用方法详解（字符串+对象+数组）

#### 实际应用场景：格式化日期字符串

```javascript

// 将"20231225"转换为"2023-12-25"

function formatDate(dateStr) {

  return dateStr.replace(/^(\d{4})(\d{2})(\d{2})$/, "$1-$2-$3");

  // 正则捕获组分别匹配年、月、日，$1-$3引用捕获内容

}

```

## 二、对象常用的方法

JavaScript 对象作为键值对集合，其方法是操作数据结构的关键。以下两个方法在遍历和属性检查场景中应用广泛。

### 2.1 Object.keys()：获取对象的所有键名

#### 语法

`Object.keys(obj)`

#### 参数

obj：要操作的对象

#### 返回值

包含对象所有自有可枚举属性的数组

#### 基础示例

```javascript

const user = {

  name: "John",

  age: 30,

  email: "john@example.com"

};

console.log(Object.keys(user)); // ["name", "age", "email"]

// 遍历对象属性

Object.keys(user).forEach(key => {

  console.log(`${key}: ${user[key]}`);

  // name: John | age: 30 | email: john@example.com

});

```

#### 实际应用场景：表单数据校验

```javascript

const formData = {

  username: "",

  password: "123",

  email: "invalid-email"

};

// 检查必填字段

const requiredFields = ["username", "password"];

const errors = {};

Object.keys(formData).forEach(key => {

  if (requiredFields.includes(key) && !formData[key]) {

    errors[key] = `${key}不能为空`;

  }

});

console.log(errors); // { username: "username不能为空" }

```

### 2.2 hasOwnProperty()：检查属性是否为对象自有

#### 语法

`obj.hasOwnProperty(prop)`

#### 参数

prop：属性名（字符串或 Symbol）

#### 返回值

布尔值，表示该属性是否为对象自身定义（非继承）

#### 基础示例

```javascript

const obj = { a: 1 };

obj.__proto__.b = 2; // 给原型添加属性

console.log(obj.hasOwnProperty("a")); // true（自有属性）

console.log(obj.hasOwnProperty("b")); // false（继承属性）

console.log(obj.hasOwnProperty("toString")); // false（原型链方法）

```

#### 实际应用场景：安全遍历对象属性

```javascript

const data = { id: 1, name: "Test", __proto__: { extra: "inherited" } };

// 只遍历自有属性

for (const key in data) {

  if (data.hasOwnProperty(key)) {

    console.log(key); // id, name（不会输出extra）

  }

}

```

## 三、数组的常用方法

### 3.1 map()：批量转换数组元素

#### 语法

`arr.map(callback(currentValue [, index [, array]]) [, thisArg])`

#### 参数

- callback：对每个元素执行的函数，返回转换后的值

- currentValue：当前处理的元素（必填）

- index：元素索引（可选）

- array：原数组（可选）

- thisArg：执行回调时的 this 指向（可选）

#### 返回值

新数组，长度与原数组相同，元素为回调函数的返回值

#### 基础示例

```javascript

const numbers = [1, 2, 3, 4];

// 每个元素乘以2

const doubled = numbers.map(num => num * 2);

console.log(doubled); // [2, 4, 6, 8]

// 对象数组转换

const users = [

  { id: 1, name: "Alice" },

  { id: 2, name: "Bob" }

];

const userNames = users.map(user => user.name);

console.log(userNames); // ["Alice", "Bob"]

```

#### 实际应用场景：格式化 API 响应数据

```javascript

// 假设从API获取的原始数据

const rawProducts = [

  { id: 1, price: 100, name: "Shirt" },

  { id: 2, price: 200, name: "Pants" }

];

// 转换为前端展示所需格式

const formattedProducts = rawProducts.map(product => ({

  ...product,

  price: `$${product.price.toFixed(2)}`, // 价格格式化

  discountPrice: `$${(product.price * 0.9).toFixed(2)}`, // 计算折扣价

  isOnSale: product.price > 150 // 添加促销标记

}));

```

### 3.2 filter()：筛选数组元素

#### 语法

`arr.filter(callback(element [, index [, array]]) [, thisArg])`

#### 参数

与 map() 类似，callback 返回布尔值

#### 返回值

新数组，包含所有使 callback 返回 true 的元素

#### 基础示例

```javascript

const numbers = [10, 23, 5, 18, 30];

// 筛选偶数

const evenNumbers = numbers.filter(num => num % 2 === 0);

console.log(evenNumbers); // [10, 18, 30]

// 筛选对象数组

const products = [

  { name: "Laptop", price: 999 },

  { name: "Mouse", price: 25 },

  { name: "Keyboard", price: 50 }

];

const affordableProducts = products.filter(p => p.price < 100);

console.log(affordableProducts); // Mouse, Keyboard

```

#### 实际应用场景：多条件搜索功能

```javascript

function searchProducts(products, query) {

  const { minPrice, maxPrice, keyword } = query;

  return products.filter(product => {

 // 价格在范围内，且名称包含关键词（不区分大小写）

    return product.price >= minPrice &

           product.price <= maxPrice &

           product.name.toLowerCase().includes(keyword.toLowerCase());

  });

}

```

### 3.3 reduce()：数组元素的累加与聚合

#### 语法

`arr.reduce(callback(accumulator, currentValue [, index [, array]]) [, initialValue])`

#### 参数

- callback：累加器函数

- accumulator：累计结果（必填）

- currentValue：当前元素（必填）

- initialValue：初始累加值（可选，省略则使用数组第一个元素作为初始值）

#### 返回值

最终累加结果

#### 基础示例

```javascript

const numbers = [1, 2, 3, 4];

// 求和

const sum = numbers.reduce((acc, num) => acc + num, 0);

console.log(sum); // 10

// 求最大值

const max = numbers.reduce((acc, num) => Math.max(acc, num), -Infinity);

console.log(max); // 4

// 数组扁平化

const nestedArray = [1, [2, [3, 4], 5]];

const flatArray = nestedArray.reduce((acc, item) =>

  acc.concat(Array.isArray(item) ? flatArray(item) : item), []);

console.log(flatArray); // [1, 2, 3, 4, 5]

```

#### 实际应用场景：复杂数据聚合

```javascript

// 按类别分组产品

const products = [

  { category: "electronics", name: "Laptop" },

  { category: "clothing", name: "Shirt" },

  { category: "electronics", name: "Phone" }

];

const grouped = products.reduce((acc, product) => {

  if (!acc[product.category]) {

    acc[product.category] = []; // 初始化分类数组

  }

  acc[product.category].push(product.name);

  return acc;

}, {}); // 初始值为空对象

console.log(grouped);

// { electronics: ["Laptop", "Phone"], clothing: ["Shirt"] }

```


