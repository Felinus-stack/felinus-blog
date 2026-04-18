---
title: 《JS Set与Map详解》
tags:
    - JS
    - Set
    - Map
categories:
    - 前端
    - JS
date: "2025-12-26 17:55:44"
updated: "2026-04-24 00:00:00"
abbrlink: f8882812
---

JS Set与Map详解（含应用场景+核心区别）

### 推荐短英文文件名（.md格式）

1. js-set-map-guide.md（首选，涵盖Set与Map核心主题，简洁直观）

2. js-set-and-map.md（简洁明了，突出两大数据结构）

3. js-set-map-usage.md（侧重用法，适配教程类文档）

# JS Set与Map详解（含应用场景+核心区别）

## 一、Set 部分

### 1.1 Set 的特点

- 不重复性：集合中的数据没有重复值，自动过滤重复元素。

- 有序性：元素的遍历顺序与元素的添加顺序一致。

- 无键名，仅存值：Set 是「值的集合」，不同于对象的「键值对」结构，遍历可直接获取元素本身。

### 1.2 Set 的创建与声明

```javascript
// 1. 创建空 Set

const set1 = new Set();

console.log(set1); // Set(0) {}

// 2. 从数组创建（自动去重）

const set2 = new Set([1, 2, 2, 3, 4]);

console.log(set2); // Set(4) {1, 2, 3, 4}（重复的 2 被过滤）

// 3. 从字符串创建（按字符拆分，自动去重）

const set3 = new Set("aabbcc");

console.log(set3); // Set(3) {"a", "b", "c"}（重复字符被过滤）

// 4. 从类数组/可迭代对象创建（扩展用法）

const set4 = new Set(document.querySelectorAll("div")); // 从 DOM 集合创建

console.log(set4); // Set(n) {div, div, ...}（自动去重相同 DOM 元素）
```

### 1.3 Set 常用方法与属性

| 方法 / 属性 | 作用 | 示例 | 输出结果 |

| --- | --- | --- | --- |

| size（属性） | 获取元素个数 | set2.size | 4 |

| add(value) | 添加元素（支持链式调用） | set1.add(5).add(6).add(5) | Set (2) {5, 6}（重复 5 被忽略） |

| has(value) | 判断元素是否存在 | set2.has(2) | true |

| delete(value) | 删除元素（成功返回 true） | set2.delete(3) | true（删除后 set2 为 {1,2,4}） |

| clear() | 清空所有元素 | set2.clear() | undefined（清空后 size 为 0） |

| forEach() | 按添加顺序遍历 | set3.forEach(val => console.log(val)) | a、b、c（依次输出） |

| 扩展运算符 (...) | 转为数组 | [...set3] | ["a", "b", "c"] |

### 1.4 Set 实际应用场景

#### （1）数组去重

```javascript
// 基础：普通数组去重

const arr = [1, 2, 2, 3, 4, 4, 5];

const uniqueArr = [...new Set(arr)];

console.log(uniqueArr); // [1, 2, 3, 4, 5]

// 扩展：复杂数组去重（需注意引用类型）

const objArr = [{ id: 1 }, { id: 2 }, { id: 1 }];

// 错误：引用类型按地址去重，无法直接去重

const wrongUnique = [...new Set(objArr)];

// 正确：先转成可比较的基本类型（如 JSON 字符串），再去重

const correctUnique = [...new Set(objArr.map((item) => JSON.stringify(item)))].map((str) =>
    JSON.parse(str),
);

console.log(correctUnique); // [{ id: 1 }, { id: 2 }]
```

#### （2）求交集、差集、并集（完善集合运算）

```javascript
const setA = new Set([1, 2, 3]);

const setB = new Set([2, 3, 4]);

// 交集：两个集合共有的元素

const intersection = new Set([...setA].filter((x) => setB.has(x)));

console.log(intersection); // Set(2) {2, 3}

// 差集：setA 有但 setB 没有的元素

const differenceA = new Set([...setA].filter((x) => !setB.has(x)));

console.log(differenceA); // Set(1) {1}

// 差集：setB 有但 setA 没有的元素

const differenceB = new Set([...setB].filter((x) => !setA.has(x)));

console.log(differenceB); // Set(1) {4}

// 并集：两个集合所有元素（自动去重）

const union = new Set([...setA, ...setB]);

console.log(union); // Set(4) {1, 2, 3, 4}
```

#### （3）快速判断元素存在（性能优于数组 indexOf）

```javascript
// 场景：判断用户权限是否存在

const permissions = new Set(["read", "write", "delete"]);

const hasWritePerm = permissions.has("write");

console.log(hasWritePerm); // true（Set 的 has 方法时间复杂度 O(1)，数组 indexOf 是 O(n)）
```

## 二、Map 的使用

### 2.1 Map 的特点

- 键类型灵活：键可以是任意类型（字符串、数字、对象、数组、Symbol 等），而普通对象的键只能是字符串、Symbol 或数字（会自动转为字符串）。

- 有序性：键值对的遍历顺序与插入顺序一致（区别于普通对象的无序特性）。

- 可迭代性：直接支持 for...of 遍历，无需额外转换。

### 2.2 Map 的声明与创建

```javascript
// 1. 创建空 Map

const map1 = new Map();

console.log(map1); // Map(0) {}

// 2. 从二维数组创建（数组元素必须是 [key, value] 格式）

const map2 = new Map([
    ["name", "John"],

    ["age", 16],

    [1, "number"], // 键可以是数字（不会转为字符串）

    [Symbol("id"), 123], // 键可以是 Symbol

    [{ x: 1 }, "objectKey"], // 键可以是对象
]);

console.log(map2); // Map(5) {["name" => "John"], ["age" => 16], [1 => "number"], [Symbol(id) => 123], [{x:1} => "objectKey"]}

// 3. 从对象创建（核心：Object.entries() 的作用）

const obj = { a: 1, b: 2, c: 3 };

const map3 = new Map(Object.entries(obj));

console.log(map3); // Map(3) {["a" => 1], ["b" => 2], ["c" => 3]}

// 4. 从 Set 创建（Set 转 Map，需手动构造 [key, value]）

const set = new Set(["x", "y", "z"]);

const map4 = new Map([...set].map((val) => [val, val.toUpperCase()]));

console.log(map4); // Map(3) {["x" => "X"], ["y" => "Y"], ["z" => "Z"]}
```

### 2.3 补充：Object.entries() 详解

在 `const map3 = new Map(Object.entries(obj));` 中，Object.entries(obj) 是「对象转 Map」的核心桥梁，作用是：将普通对象转为「键值对二维数组」，格式为 `[["键名1", 键值1], ["键名2", 键值2], ...]`，正好匹配 Map 构造函数的参数要求。

### 2.4 Map 核心常用方法与属性

| 方法 / 属性 | 作用 | 示例 | 输出结果 |

| --- | --- | --- | --- |

| size（属性） | 获取键值对个数 | map2.size | 5 |

| set(key, value) | 添加 / 修改键值对（支持链式调用） | map1.set("gender", "男").set("age", 18) | Map (2) {["gender" => "男"], ["age" => 18]}（修改 age 为 18） |

| get(key) | 获取指定键的值（无则返回 undefined） | map2.get("name") | "John" |

| has(key) | 判断键是否存在 | map2.has(Symbol("id")) | true |

| delete(key) | 删除指定键值对（成功返回 true） | map2.delete(1) | true（删除键为 1 的键值对） |

| clear() | 清空所有键值对 | map2.clear() | undefined（清空后 size 为 0） |

| forEach() | 按插入顺序遍历 | map3.forEach((val, key) => console.log(key, val)) | a 1、b 2、c 3（依次输出） |

### 2.5 Map 的使用场景

#### （1）多类型键场景

```javascript
// 场景：用对象作为键，存储对应的数据

const user1 = { id: 1 };

const user2 = { id: 2 };

const userMap = new Map();

userMap.set(user1, "用户1的信息");

userMap.set(user2, "用户2的信息");

console.log(userMap.get(user1)); // "用户1的信息"（通过对象引用获取，普通对象无法做到）
```

#### （2）Map 与数组 / 对象的相互转换

```javascript
// 1. Map 转数组（扩展运算符）

const mapArr = [...map3];

console.log(mapArr); // [["a", 1], ["b", 2], ["c", 3]]

// 2. Map 转对象（Object.fromEntries()）

const mapToObj = Object.fromEntries(map3);

console.log(mapToObj); // { a: 1, b: 2, c: 3 }

// 3. Map 转 JSON（需先转数组）

const mapToJson = JSON.stringify([...map3]);

console.log(mapToJson); // "[[\"a\",1],[\"b\",2],[\"c\",3]]"
```

#### （3）数据缓存

```javascript
// 场景：缓存接口请求结果，键为请求参数（对象类型）

const cacheMap = new Map();

// 模拟接口请求

function fetchData(params) {
    const cacheKey = JSON.stringify(params); // 将对象参数转为字符串作为键

    if (cacheMap.has(cacheKey)) {
        console.log("从缓存获取数据");

        return cacheMap.get(cacheKey);
    }

    // 模拟接口请求耗时操作

    const data = `参数 ${cacheKey} 的请求结果`;

    cacheMap.set(cacheKey, data); // 存入缓存

    console.log("从接口获取数据");

    return data;
}

fetchData({ page: 1, size: 10 }); // 从接口获取数据

fetchData({ page: 1, size: 10 }); // 从缓存获取数据（重复请求，直接用缓存）
```

## 三、Map 与普通对象的核心区别

| 对比维度 | Map | 普通对象 |

| --- | --- | --- |

| 键类型 | 任意类型（字符串、数字、对象、Symbol 等） | 仅字符串、Symbol、数字（自动转字符串） |

| 有序性 | 插入顺序 = 遍历顺序 | ES6 后部分有序（数字键升序、字符串 / Symbol 键插入顺序） |

| 大小获取 | 直接通过 size 属性 | 需通过 Object.keys(obj).length 计算 |

| 遍历方式 | 支持 for...of、forEach 直接遍历 | 需通过 Object.keys()/Object.values() 等转换后遍历 |

| 原型链干扰 | 无（不继承原型属性） | 有（继承 Object.prototype 的属性，如 toString） |

| 性能 | 频繁增删键值对时性能更优 | 频繁增删时性能较差，适合静态数据 |

| 适用场景 | 多类型键、频繁增删、有序键值对 | 静态数据存储、简单键值映射（字符串 / Symbol 键） |
