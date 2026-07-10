---
title: 《Git Hooks与Chrome DevTools调试指南》
tags:
    - Git Hooks
    - DevTools
    - 网络调试
    - 前端工程化
    - 代码检查
categories:
    - 前端开发
    - 前端工程化
date: "2025-05-27 08:41:18"
updated: "2025-05-27 08:41:18"
abbrlink: f888283
---
# 前端工程化实践：Git Hooks 拦截机制与 Chrome DevTools Network 接口调试详解

在现代前端开发过程中，除了掌握框架和业务开发能力，还需要具备一定的工程化能力。

实际项目中经常会遇到：

- 提交代码前需要自动检查代码规范
- 防止错误代码进入仓库
- 接口请求失败无法定位原因
- 前后端联调出现参数问题
- 页面数据异常但不知道问题在哪里


这些问题分别对应两个重要能力：

1. **Git Hooks 拦截机制**
2. **Chrome DevTools Network 网络调试能力**


本文主要介绍：

- Git Hooks 是什么
- Git Hooks 工作流程
- 常见 Hooks 类型
- 使用 Husky 管理 Git Hooks
- 前端项目中的代码提交检查流程
- Chrome DevTools Network 使用方法
- 如何进行接口抓包和问题排查
- 常见接口问题分析


---

# 一、Git Hooks 拦截机制


## 1. 什么是 Git Hooks？


Git Hooks：

> Git 提供的一种在特定 Git 操作发生时自动执行脚本的机制。


简单来说：

Git Hooks 可以理解为：

**Git 生命周期中的自动化拦截点。**


例如：

开发人员提交代码：

```bash
git commit
```

执行过程：

```text
git commit

↓

触发 Git Hook

↓

执行检查脚本

↓

检查通过

↓

提交代码
```


如果检查失败：

```text
阻止提交
```


---

# 2. 为什么需要 Git Hooks？


在团队开发中，经常存在：


## （1）代码格式不统一


例如：

开发人员 A：

```javascript
const a=1;
```


开发人员 B：

```javascript
const a = 1;
```


导致：

- 代码风格混乱
- Code Review 成本增加


---

## （2）提交错误代码


例如：

开发人员提交：

```javascript
console.log("test")
```


或者：

```javascript
debugger;
```


进入生产环境。


---

## （3）代码质量检查


提交前自动执行：

- ESLint
- Prettier
- TypeScript 类型检查
- 单元测试


避免问题代码进入仓库。


---

# 二、Git Hooks 工作原理


Git Hooks 存放位置：

```text
.git/hooks
```


目录结构：

```text
项目

├── .git

│

└── hooks

    ├── pre-commit

    ├── commit-msg

    └── pre-push
```


这些文件本质上是：

可执行脚本。


---

# 三、常见 Git Hooks 类型


Git Hooks 根据执行阶段分为：

- 提交前 Hook
- 提交阶段 Hook
- 推送阶段 Hook


---

# 1. pre-commit


## 作用

提交代码之前执行。


流程：

```text
git commit

↓

pre-commit

↓

代码检查

↓

提交
```


常用于：

- ESLint 检查
- Prettier 格式化
- 单元测试


示例：

```bash
npm run lint
```


如果检查失败：

提交终止。


---

# 2. commit-msg


## 作用

检查提交信息。


例如团队要求：

```text
feat: 添加登录功能

fix: 修复登录bug
```


如果提交：

```text
修改代码
```


可能被拒绝。


---

# 3. pre-push


## 作用

代码 push 前执行。


流程：

```text
git push

↓

pre-push

↓

执行测试

↓

上传远程仓库
```


常用于：

- 完整测试
- 构建检查


---

# 四、前端项目中的 Git Hooks 实践


## Husky


Husky：

> 一个方便管理 Git Hooks 的工具。


安装：

```bash
npm install husky --save-dev
```


初始化：

```bash
npx husky init
```


生成：

```text
.husky

├── pre-commit

└── pre-push
```


---

# 五、结合 lint-staged 实现提交检查


问题：

如果项目很大：

每次提交检查全部文件：

效率低。


解决：

使用：

## lint-staged


作用：

只检查：

Git 暂存区文件。


流程：

```text
修改代码

↓

git add

↓

git commit

↓

husky触发

↓

lint-staged检查修改文件

↓

提交
```


配置：

```json
{
  "lint-staged": {
    "*.{js,vue,ts}": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```


效果：

提交前自动：

- 修复格式
- 检查代码


---

# 六、Chrome DevTools Network 网络调试


## 1. 什么是 Network？


Chrome DevTools Network：

> 浏览器提供的网络请求分析工具。


主要用于：

- 查看 HTTP 请求
- 查看请求参数
- 查看响应数据
- 分析接口耗时
- 排查接口错误


---

# 七、打开 Network 面板


快捷键：

Windows：

```text
F12

或者

Ctrl + Shift + I
```


Mac：

```text
Command + Option + I
```


---

# 八、Network 面板结构


```text
Network

├── Name

├── Method

├── Status

├── Type

├── Initiator

├── Size

└── Time
```


---

# 九、如何查看接口请求


点击请求后：

可以查看：


## Headers


包含：

- Request URL
- Request Method
- Request Headers


例如：

```http
Authorization:
Bearer token
```


---

## Payload


查看请求参数。


例如：

```json
{
    "username":"Tom",
    "password":"123456"
}
```


用于排查：

- 参数是否正确
- 字段是否缺失


---

## Response


查看后端返回数据。


例如：

```json
{
    "code":200,
    "data":{
        "name":"Tom"
    }
}
```


---

# 十、Network 排查接口问题流程


```text
页面异常

↓

打开 Network

↓

找到接口请求

↓

查看请求参数

↓

查看响应状态码

↓

分析返回数据

↓

定位问题
```


---

# 十一、常见接口问题分析


## 1. 请求没有发送


可能原因：

- 点击事件没有触发
- 前端代码报错
- 请求逻辑未执行


排查：

查看 Console。


---

## 2. 404


表示：

接口不存在。


检查：

- 请求地址
- 路由路径


---

## 3. 401


表示：

未认证。


常见原因：

Token 失效。


检查：

```http
Authorization:
Bearer token
```


---

## 4. 403


表示：

没有权限。


例如：

普通用户访问管理员接口。


---

## 5. 500


表示：

服务器异常。


查看：

Response。


---

## 6. 参数错误


例如：

前端：

```json
{
 "userName":"Tom"
}
```


后端：

```json
{
 "username":"Tom"
}
```


字段名称不一致：

导致参数绑定失败。


---

# 十二、接口性能分析


## Time


表示：

请求耗时。


例如：

```text
2000ms
```


说明接口较慢。


---

## Timing


查看请求阶段：

```text
Queueing

↓

DNS

↓

TCP连接

↓

Waiting

↓

Download
```


---

# 十三、调试技巧


## Preserve log


作用：

刷新页面后：

保留之前请求。


适合：

登录跳转问题。


---

## Disable cache


作用：

关闭浏览器缓存。


避免：

缓存影响测试。


---

## Copy as cURL


复制完整请求：

方便后端排查。


---

# 十四、面试回答模板


## Git Hooks 是什么？


> Git Hooks 是 Git 提供的生命周期钩子机制，可以在 commit、push 等操作前后执行脚本。在前端项目中通常结合 Husky 和 lint-staged，在提交代码前自动执行 ESLint、Prettier 和测试，保证代码质量。


---

## 如何排查接口问题？


> 我通常会通过 Chrome DevTools Network 查看接口请求。首先确认请求是否发送，然后检查 Request URL、Method、Headers、Payload 参数，再查看 Response 返回数据和状态码。如果接口慢，还会分析 Timing 判断耗时阶段，从而定位前端、网络还是后端问题。


---

# 总结


## Git Hooks


核心：

```text
代码提交

↓

自动拦截

↓

代码检查

↓

保证质量
```


重点：

- pre-commit
- commit-msg
- pre-push
- Husky
- lint-staged


---

## Chrome DevTools Network


核心：

```text
请求分析

↓

参数检查

↓

响应查看

↓

性能定位
```


重点：

- Headers
- Payload
- Response
- Status Code
- Timing


掌握 Git Hooks 和 Chrome DevTools Network 调试能力，可以提升前端工程化能力，也是企业级前端开发的重要技能。