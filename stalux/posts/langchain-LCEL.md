---
title: 《从“控制流程”到“数据流”：彻底理解 LCEL 的设计思想》
tags:
    - AI
    - LangChain
    - 大模型
    - LCEL
categories:
    - AI
    - LangChain
date: "  2026-05-2 11:34:58  "
updated: " 2026-05-2 11:34:58"
abbrlink: f8882815
---

# 从“控制流程”到“数据流”：彻底理解 LCEL 的设计思想

在学习 LangChain 的过程中，很多人会写 `.pipe()`，但却并不真正理解它为什么重要。这篇文章不会只教你“怎么用”，而是带你从底层思维转变出发，彻底搞懂 LCEL（LangChain Expression Language）的核心价值。

---

## 一、从“写步骤”到“描述流动”

在传统开发中，我们习惯用**命令式编程**来解决问题，比如：

```ts
const formatted = prompt.format({ topic: '闭包' });
const response = await model.invoke(formatted);
```

这种写法的特点是：

* 你明确控制每一步执行
* 代码强调“先做什么，再做什么”
* 流程是由你手动串起来的

而 LCEL 做了一件本质不同的事情：

👉 它让你描述“数据如何流动”，而不是“代码如何执行”

来看一行关键代码：

```ts
const chain = prompt.pipe(model);
```

这不是在“调用 prompt 然后调用 model”，而是在定义：

```
输入 → prompt → model → 输出
```

你不再关心执行顺序，而是定义了一个**数据流结构**。

---

## 二、LCEL 的核心：Pipe（管道）

`.pipe()` 的设计灵感来自 Unix 管道：

```bash
cat file.txt | grep hello | sort
```

数据像水一样，流经不同处理节点。

LCEL 中也是同样的思想：

```ts
const chain = prompt.pipe(model);
```

等价于：

```
输入数据
  ↓
PromptTemplate（填充变量）
  ↓
ChatModel（生成回复）
  ↓
输出结果
```

你没有写“调用顺序”，但执行顺序已经隐含在结构中。

---

## 三、Runnable：一切皆节点

LCEL 能成立的关键，是 LangChain 的一个核心抽象：

👉 所有组件都是 Runnable

包括：

* PromptTemplate
* LLM / ChatModel
* OutputParser
* Chain 本身

它们统一拥有这些方法：

* `invoke(input)`：单次执行
* `stream(input)`：流式输出
* `batch(inputs)`：批量执行

这带来一个非常强大的能力：

👉 任何东西都可以拼接进管道

例如：

```ts
const chain = prompt
  .pipe(model)
  .pipe(outputParser);
```

这不是“三段代码”，而是一个完整的数据处理流水线。

---

## 四、自动数据流：背后发生了什么？

当你执行：

```ts
await chain.invoke({ topic: '闭包' });
```

内部其实发生了这些事情：

1. 输入进入 PromptTemplate
   `{ topic: '闭包' }`

2. 自动生成 Prompt
   `"用一句话解释：闭包"`

3. Prompt 传递给 Model
   `prompt → ChatModel`

4. Model 返回结果
   `AIMessage { content: "..." }`

整个过程你完全没有：

* 手动传递中间变量
* 控制执行顺序
* 管理状态

👉 数据“自己流动”

这就是 LCEL 的本质。

---

## 五、命令式 vs 声明式

我们用一个对比来加深理解：

### 命令式写法（Imperative）

```ts
const formatted = prompt.format({ topic: '闭包' });
const response = await model.invoke(formatted);
```

特点：

* 强调步骤
* 你控制流程
* 代码容易变复杂

---

### 声明式写法（LCEL）

```ts
const chain = prompt.pipe(model);
await chain.invoke({ topic: '闭包' });
```

特点：

* 强调结构
* 系统控制流程
* 更容易扩展和组合

---

## 六、为什么 LCEL 对 Agent 至关重要？

Agent 本质上是一个**复杂链条系统**：

```
用户输入
 → Prompt（理解任务）
 → Model（做决策）
 → Tool（执行）
 → Model（总结）
 → 输出
```

如果用命令式写法，你会得到：

* 大量嵌套调用
* 状态混乱
* 难以维护

而 LCEL 可以这样写：

```ts
const agentChain = step1
  .pipe(model)
  .pipe(tool)
  .pipe(model);
```

👉 Agent = 可组合的链

带来的好处：

* 可以随时插入新步骤（如 RAG 检索）
* 每个节点可独立调试
* 链可以复用和嵌套

---

## 七、完整示例

```ts
import { ChatDeepSeek } from '@langchain/deepseek';
import { PromptTemplate } from '@langchain/core/prompts';
import { StringOutputParser } from '@langchain/core/output_parsers';

const model = new ChatDeepSeek({
  model: 'deepseek-reasoner',
  temperature: 0.7
});

const prompt = PromptTemplate.fromTemplate(
  "用一句话解释：{topic}"
);

const parser = new StringOutputParser();

// 构建链
const chain = prompt
  .pipe(model)
  .pipe(parser);

// 执行
const result = await chain.invoke({ topic: "闭包" });

console.log(result);
```

数据流结构：

```
input → prompt → model → parser → string
```

每一段都可以替换、复用、扩展。

---

## 八、一句话总结 LCEL

如果你只记住一句话：

👉 LCEL = 用“数据流”替代“控制流”

你不再写程序“怎么执行”，
而是描述“数据如何流经系统”。

---

## 九、写在最后：思维的真正升级

很多人学习 LCEL 时卡住，不是因为 API 难，而是因为思维还停留在：

* “我该怎么调用？”
* “下一步我要做什么？”

而 LCEL 要你思考的是：

👉 数据从哪里来？要经过哪些节点？最终变成什么？

一旦你完成这个转变，你会发现：

* 代码更短
* 结构更清晰
* 系统更容易扩展

这不仅是一个 API 的学习，而是一次编程思维的升级。
