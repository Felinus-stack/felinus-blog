---
title: 《LangChain 实战入门：从 0 到 1 构建一个支持工具调用的 AI 助手》
tags:
    - AI
    - LangChain
    - 大模型
    - 框架
categories:
    - AI
    - React
date: "  2026-04-25 11:34:58  "
updated: " 2026-04-25 11:34:58"
abbrlink: f8882814
---
# 在大模型（LLM）应用开发中，一个非常现实的问题是：

## ❓ 不同模型接口各不相同，如何优雅地统一调用？

比如：

* 千问（Qwen）
* DeepSeek
* Kimi
* OpenAI

每家 API 风格都不一样，如果你直接对接，代码会变得非常混乱。

这时候，**LangChain** 就登场了。

---

# 🧠 一、LangChain 是什么？

一句话总结：

> LangChain 是一个用于连接大模型与应用的开发框架。

它帮你做了三件核心事情：

## 1. 统一模型调用接口

无论你用哪个模型，都可以用同一套代码调用。

## 2. 管理对话上下文（Messages）

不用再手动拼字符串。

## 3. 支持工具调用（Agent 能力基础）

让 AI 不只是“聊天”，还能“做事”。

---

# ⚙️ 二、环境准备

## 初始化项目（推荐 TypeScript）

```bash
npm init -y
```

## 安装依赖

```bash
npm install langchain @langchain/core
npm install @langchain/openai dotenv
```

## 配置环境变量

创建 `.env` 文件：

```bash
QWEN_API_KEY=你的API_KEY
```

## 配置 package.json

```json
{
  "type": "module"
}
```

### 为什么要加？

因为 LangChain 使用 **ESM 模块**，不加会报错。

---

# 🚀 三、第一个 LLM 调用

```ts
import dotenv from "dotenv";
import { ChatOpenAI } from "@langchain/openai";

dotenv.config();

const llm = new ChatOpenAI({
  model: "qwen-plus",
  apiKey: process.env.QWEN_API_KEY,
  temperature: 0.7,
  configuration: {
    baseURL: "https://dashscope.aliyuncs.com/compatible-mode/v1"
  }
});

const res = await llm.invoke("世界上最高的山峰是哪座？");

console.log(res.content);
```

## 核心理解

* `ChatOpenAI`：统一模型入口
* `invoke()`：最基础调用方式
* `content`：模型输出内容

本质就是：

> LangChain 把所有模型抽象成统一接口

---

# 💬 四、消息机制（Messages）——核心设计

LangChain 并不是用字符串拼接上下文，而是用**消息对象**：

```ts
import { HumanMessage, SystemMessage } from "langchain";

const messages = [
  new SystemMessage("你是一个智能助手"),
  new HumanMessage("明朝什么时候建立？")
];

const res = await llm.invoke(messages);
```

## 消息类型

| 类型            | 作用       |
| ------------- | -------- |
| SystemMessage | 设定 AI 行为 |
| HumanMessage  | 用户输入     |
| AIMessage     | AI 输出    |
| ToolMessage   | 工具返回     |

## 关键思想

> 对话 = 一组有结构的消息，而不是字符串

---

# ⚡ 五、5 种常用调用方式

## 1. 普通调用（invoke）

适合简单问答。

```ts
await llm.invoke("你好");
```

---

## 2. 流式调用（stream）🔥

```ts
const stream = await llm.stream("简单介绍一下人工智能");

for await (const chunk of stream) {
  console.log(chunk.text);
}
```

用于：

* 聊天界面
* 打字机效果

---

## 3. 批量调用（batch）

```ts
const res = await llm.batch([
  "什么是AI？",
  "什么是区块链？"
]);
```

用于：

* 高并发任务

---

## 4. 结构化输出（JSON）🔥

```ts
import * as z from "zod";

const schema = z.object({
  title: z.string(),
  rating: z.number()
});

const model = llm.withStructuredOutput(schema);

const res = await model.invoke("介绍泰坦尼克号");

console.log(res);
```

用于：

* 后端接口
* 数据处理

---

## 5. 多轮对话（Messages）

```ts
import {
  HumanMessage,
  SystemMessage,
  AIMessage
} from "langchain";

// 构建对话历史
const messages = [
  new SystemMessage("你是一个智能助手"),
  new HumanMessage("我叫小军"),
  new AIMessage("好的，我记住了，你叫小军"),
  new HumanMessage("我刚刚说我叫什么名字？")
];

// 调用模型
const res = await llm.invoke(messages);

console.log(res.content);
```

输出结果：

```text
你叫小军 😊
刚刚你告诉我你的名字是小军。
```

用于：

* 聊天机器人
* 上下文记忆

---

# 🛠️ 六、工具调用（重点🔥🔥🔥）

这是 LangChain 最强大的能力之一。

## 什么是工具调用？

让 AI 不只是“回答问题”，而是“调用函数解决问题”。

---

## 示例：天气查询工具

### 1. 定义工具

```ts
import { tool } from "@langchain/core/tools";

const getWeather = tool(
  async (input) => {
    const data = {
      北京: "多云 15°C",
      上海: "小雨 18°C"
    };

    return data[input] || "暂无数据";
  },
  {
    name: "get_weather",
    description: "查询城市天气"
  }
);
```

### 2. 绑定工具

```ts
const llmWithTools = llm.bindTools([getWeather]);
```

### 3. 调用流程

```ts
const res = await llmWithTools.invoke("北京天气怎么样？");
```

## 实际执行流程

1. 用户提问
2. 模型判断需要调用工具
3. 返回 `tool_call`
4. 执行函数
5. 返回 `ToolMessage`
6. 模型生成最终回答

## 本质

> LLM = 大脑，Tool = 手脚

---

# 🎯 七、你应该真正理解的 3 件事

## 1. LangChain ≠ AI

它只是一个：

> 调度框架（Orchestration Layer）

## 2. 核心三要素

* LLM（模型）
* Messages（上下文）
* Tools（能力扩展）

## 3. Agent 的基础

工具调用就是 Agent 的雏形：

👉 AI 可以：

* 查天气
* 查数据库
* 调接口
* 执行任务

---

# 🚀 八、下一步可以做什么？

当你掌握这些后，可以继续进阶：

## Agent（智能体）

自动决策 + 多工具调用

## RAG（检索增强生成）

接入知识库

## Workflow（工作流）

构建复杂 AI 应用

---

# 🎯 总结

LangChain 的价值不在“封装 API”，而在：

> 用统一的方式组织 AI 能力

如果你只记住一句话：

# LangChain = 模型 + 消息 + 工具 的组合框架

