---
title: 《LangChain.js 学习指南：从基础到 Agent 实战》
tags:
    - AI
    - LangChain
    - 大模型
    - Agent
categories:
    - AI
    - LangChain
date: "2026-06-12 12:00:00"
updated: "2026-06-12 12:00:00"
abbrlink: f8882816
---

> 本文整理了 LangChain.js 的核心概念与实践，涵盖模型创建、提示词模板、链式调用、任务拆解、Plan & Execute 模式、ReAct Agent 等内容。

---

## 📚 目录

1. 创建大模型对象
2. PromptTemplate 提示词模板
3. 链式调用（LCEL 基础）
4. AI 面试助手（实际应用）
5. LCEL 完整案例（三节点链）
6. 任务拆解（RunnableSequence）
7. Plan & Execute 模式
8. ReAct Agent
9. Tool Calling Agent
10. 总结与对比

---

## 1. 创建大模型对象

### 核心概念

LangChain 通过 `ChatOpenAI` 类统一调用各种大模型，支持 OpenAI、通义千问、Claude 等。

### 代码示例

```typescript
import { ChatOpenAI } from "@langchain/openai";

const llm = new ChatOpenAI({
  model: "qwen-plus",
  apiKey: process.env.QWEN_API_KEY,
  temperature: 0.7,        // 控制随机性，值越大越随机
  streamUsage: false,      // 是否开启流式返回
  logprobs: true,          // 是否返回每个 token 的概率信息
  configuration: {
    baseURL: "https://dashscope.aliyuncs.com/compatible-mode/v1",
  },
});
```

### 关键参数

| 参数 | 说明 |
|------|------|
| `model` | 模型名称 |
| `apiKey` | API 密钥 |
| `temperature` | 温度参数，控制随机性（0-1） |
| `streamUsage` | 是否开启流式返回 |
| `logprobs` | 是否返回 token 概率信息 |

---

## 2. PromptTemplate 提示词模板

### 核心概念

PromptTemplate 将提示词拆分为**模板 + 参数**，实现模板与数据分离，便于复用和维护。

### 代码示例

```typescript
import { PromptTemplate } from "@langchain/core/prompts";

// 定义模板
const prompt = PromptTemplate.fromTemplate(`
你是一个{role}。
请用不超过{limit}字回答以下问题：
{question}
`);

// 注入数据，生成最终提示词
const promptStr = await prompt.format({
  role: "前端面试官",
  limit: "50",
  question: "什么是闭包",
});

// 发送给模型
const res = await llm.invoke(promptStr);
console.log(res.content);
```

### 优势

- **复用性**：同一个模板，切换参数即可复用
- **可维护性**：模板与数据分离，易于修改
- **类型安全**：支持 TypeScript 类型推断

---

## 3. 链式调用（LCEL 基础）

### 核心概念

LangChain 提供 `.pipe()` 方法，将多个 Runnable（可执行单元）串联成链，数据自动从上游流向下游。

### 代码示例

```typescript
const prompt = PromptTemplate.fromTemplate(`
你是一个{role}。
请用不超过{limit}字回答以下问题：
{question}
`);

// .pipe() 将 prompt 和 llm 串联成一个新链
const chain = prompt.pipe(llm);

// 一次 invoke，数据依次流过两个节点
const res = await chain.invoke({
  role: "前端面试官",
  limit: "50",
  question: "什么是闭包",
});

console.log("链式调用结果:", res.content);
```

### 数据流

```
输入参数 → prompt（格式化）→ llm（推理）→ AIMessage
```

### 优势

- **简洁**：无需手动调用 `format()` 和 `invoke()`
- **可组合**：任意 Runnable 都可以用 `.pipe()` 串联
- **可读性**：链式调用更直观

---

## 4. AI 面试助手（实际应用）

### 核心概念

通过 PromptTemplate 实现多角色、多风格的面试问答，扩展时只需修改模板，核心逻辑不变。

### 代码示例

```typescript
const prompt = PromptTemplate.fromTemplate(`
你是一位严格的{role}。
请用专业且简洁的语言回答：
{question}
限制在{limit}字以内。
`);

const chain = prompt.pipe(llm);

// 前端面试
const res1 = await chain.invoke({
  role: "前端面试官",
  question: "什么是闭包",
  limit: "80",
});

// 后端面试：同一个模板，切换角色即可复用
const res2 = await chain.invoke({
  role: "后端面试官",
  question: "什么是微服务架构",
  limit: "80",
});
```

### 价值

- **复用性**：同一个 `chain`，只需修改参数就能切换角色
- **扩展性**：想加新角色（算法、产品经理等），只需改 `role` 参数

---

## 5. LCEL 完整案例（三节点链）

### 核心概念

加入 `StringOutputParser` 后变成三节点链：`prompt → llm → parser`，每个节点都实现了 Runnable 接口，可以自由拼接。

### 代码示例

```typescript
import { StringOutputParser, JsonOutputParser } from "@langchain/core/output_parsers";

const prompt = PromptTemplate.fromTemplate("用一句话解释：{topic}");
const parser = new StringOutputParser();

// 三节点链
const chain = prompt.pipe(llm).pipe(parser);
const result = await chain.invoke({ topic: "闭包" });
console.log("类型：", typeof result);  // string
console.log("结果：", result);
```

### JsonOutputParser

```typescript
const jsonPrompt = PromptTemplate.fromTemplate(`
请以 JSON 格式返回以下语言的三个特性：
语言：{language}
格式：{{"features": ["特性1", "特性2", "特性3"]}}
`);

const jsonParser = new JsonOutputParser();
const jsonChain = jsonPrompt.pipe(llm).pipe(jsonParser);
const jsonResult = await jsonChain.invoke({ language: "TypeScript" });
console.log("第一个特性：", jsonResult.features[0]);
```

### 输出解析器

| 解析器 | 输出类型 | 用途 |
|--------|----------|------|
| `StringOutputParser` | string | 提取纯文本 |
| `JsonOutputParser` | object | 解析 JSON |

---

## 6. 任务拆解（RunnableSequence）

### 核心概念

**任务拆解（Task Decomposition）：** 把复杂任务拆成多个单一目标，每个子链只做一件事，提高质量。

### 为什么需要任务拆解？

```
原始任务（混合）：
"请详细解释闭包，并总结3个要点"
→ 解释不够深入
→ 总结不够精炼

拆解后（单一职责）：
Step 1: 专门解释（300字详细说明）
Step 2: 专门总结（3个核心要点）
Step 3: 专门格式化（JSON输出）
```

### 代码示例

```typescript
import { RunnableSequence } from "@langchain/core/runnables";

// 子链 1：详细解释
const explainPrompt = PromptTemplate.fromTemplate(`
你是一个前端专家，请详细介绍以下概念: {topic}
要求：
1. 包含定义、原理、使用场景
2. 不超过300字
`);
const explainChain = explainPrompt.pipe(llm).pipe(parser);

// 子链 2：提炼要点
const summaryPrompt = PromptTemplate.fromTemplate(`
请将以下内容总结为3个核心要点：
内容：{explanation}
`);
const summaryChain = summaryPrompt.pipe(llm).pipe(parser);

// 子链 3：格式化输出
const formatPrompt = PromptTemplate.fromTemplate(`
请将以下内容整理成结构化格式并直接输出JSON。
解释内容：{explanation}
总结要点：{summary}
`);
const formatChain = formatPrompt.pipe(llm).pipe(parser);

// 构建完整流水线
const fullChain = RunnableSequence.from([
  // Step 1：生成解释
  async (input: { topic: string }) => {
    const explanation = await explainChain.invoke({ topic: input.topic });
    return { explanation };
  },

  // Step 2：生成总结（基于 explanation）
  async (data: { explanation: string }) => {
    const summary = await summaryChain.invoke({ explanation: data.explanation });
    return { explanation: data.explanation, summary };
  },

  // Step 3：结构化输出
  async (data: { explanation: string; summary: string }) => {
    const json = await formatChain.invoke({
      explanation: data.explanation,
      summary: data.summary,
    });
    return json;
  },
]);

// 执行
const result = await fullChain.invoke({ topic: "闭包" });
console.log(result);
```

### RunnableSequence vs .pipe()

| 特性 | `.pipe()` | `RunnableSequence` |
|------|-----------|-------------------|
| **结构** | 线性串联 | 函数数组，灵活组合 |
| **数据传递** | 自动传递（隐式） | 显式函数，可修改数据 |
| **复杂度** | 简单任务 | 复杂多步骤任务 |
| **适用场景** | 2-3 个节点 | 4+ 个节点，需要中间处理 |

---

## 7. Plan & Execute 模式

### 核心思想

**Plan & Execute：** 先规划，再执行，最后评估

```
用户输入 → Planner 制定计划 → Executor 逐步执行
           ↑ Replanner 评估 ←──┘
```

### 三个核心角色

| 角色 | 职责 |
|------|------|
| **Planner** | 分析任务，制定步骤计划 |
| **Executor** | 逐个执行计划中的步骤 |
| **Replanner** | 根据执行结果决定是否需要调整计划 |

### 手写版实现

```typescript
// 主循环
async function planAndExecute(task: string) {
  // 1. 制定计划
  let steps = await planTask(task);

  // 2. 逐步执行
  const completedResults: string[] = [];
  let currentStepIndex = 0;

  while (currentStepIndex < steps.length) {
    const result = await executeStep(steps[currentStepIndex]);
    completedResults.push(result);
    currentStepIndex++;

    // 3. 评估是否需要重规划
    const replanResult = await shouldReplan(task, completedResults, steps.slice(currentStepIndex));

    if (replanResult.status === "complete") {
      return replanResult.result;
    }
  }

  // 4. 生成最终总结
  return await generateSummary(task, completedResults);
}
```

### LangGraph 版实现

```typescript
import { Annotation, StateGraph, START, END } from "@langchain/langgraph";

// 定义状态
const PlanExecuteState = Annotation.Root({
  task: Annotation<string>(),
  plan: Annotation<string[]>({
    reducer: (_prev, next) => next,
    default: () => [],
  }),
  completed: Annotation<string[]>({
    reducer: (prev, next) => [...prev, ...next],
    default: () => [],
  }),
  currentStepIndex: Annotation<number>({
    reducer: (_prev, next) => next,
    default: () => 0,
  }),
  result: Annotation<string>({
    reducer: (_prev, next) => next,
    default: () => "",
  }),
});

// 构建图
const graph = new StateGraph(PlanExecuteState)
  .addNode("planner", planNode)
  .addNode("executor", executeNode)
  .addNode("replanner", replanNode)
  .addEdge(START, "planner")
  .addConditionalEdges("planner", shouldContinue)
  .addConditionalEdges("executor", () => "replanner")
  .addConditionalEdges("replanner", afterReplan)
  .compile();

// 执行
const result = await graph.invoke({ task: "我想了解北京的美食文化" });
```

### 手写版 vs LangGraph

| 特性 | 手写版 | LangGraph |
|------|--------|-----------|
| **流程控制** | while 循环 + if/else | 图结构 + 条件边 |
| **状态管理** | 手动传递变量 | 自动管理 State |
| **代码结构** | 线性函数调用 | 节点 + 边 |
| **可读性** | 一般 | 高（流程图式） |
| **可维护性** | 低 | 高 |

---

## 8. ReAct Agent

### 核心概念

**ReAct 模式：** Reasoning（思考）+ Acting（行动）+ Observation（观察）循环

```
Thought: 我需要查找天气
Action: search("北京")
Observation: 北京今天晴，25°C
Thought: 我已经知道天气了
Final Answer: 北京今天晴，25°C
```

### 手写版实现

```typescript
async function runReAct(question: string, maxSteps: number = 5): Promise<string> {
  let context = "";
  let step = 0;

  while (step < maxSteps) {
    // 1. 调用 LLM
    const response = await llm.invoke(prompt + context);
    const text = response.content;

    // 2. 检查是否有最终答案
    if (text.includes("Final Answer:")) {
      return text.split("Final Answer:")[1].trim();
    }

    // 3. 解析 Action
    const actionMatch = text.match(/Action:\s*(\w+)\[(.*?)\]/);
    if (actionMatch) {
      const [, toolName, input] = actionMatch;

      // 4. 执行工具
      const observation = await executeTool(toolName, input);

      // 5. 更新上下文
      context += `\n${text}\nObservation: ${observation}`;
      step++;
    }
  }

  return "达到最大思考步骤，未能得出最终答案";
}
```

### createAgent 版实现

```typescript
import { createAgent, tool } from "langchain";
import { z } from "zod";

// 定义工具
const searchTool = tool(
  async ({ query }: { query: string }) => {
    return "搜索结果...";
  },
  {
    name: "search",
    description: "搜索网络信息",
    schema: z.object({
      query: z.string().describe("搜索关键词"),
    }),
  }
);

// 创建 Agent
const agent = createAgent({
  model: llm,
  tools: [searchTool],
  systemPrompt: "你是一个智能助手，善于使用工具来回答用户的问题。",
});

// 执行
const result = await agent.invoke({
  messages: [{ role: "user", content: "北京今天天气怎么样？" }],
});
```

### 手写版 vs createAgent

| 特性 | 手写版 | createAgent |
|------|--------|-------------|
| **Prompt** | 自己写 Thought/Action/Observation | 框架自动生成 |
| **解析** | 正则解析模型输出 | 原生 Tool Calling |
| **上下文** | 手动注入 Observation | 自动管理 |
| **可靠性** | 容易解析出错 | 更可靠 |
| **代码量** | 多 | 少 |

---

## 9. Tool Calling Agent

### 核心概念

**Tool Calling：** 利用 LLM 原生的 Function Calling 能力，直接输出结构化的工具调用请求，无需文本解析。

### 工作流程

```
1. 定义工具（名称、描述、参数 schema）
2. 将工具绑定到 LLM（bindTools）
3. LLM 分析用户请求，决定是否调用工具
4. 执行工具，将结果返回给 LLM
5. LLM 基于工具结果生成最终回答
```

### Zod Schema

```typescript
import { z } from "zod";

// 定义工具参数 schema
const searchSchema = z.object({
  query: z.string().describe("搜索关键词"),
});

const calculatorSchema = z.object({
  expression: z.string().describe("数学表达式，如 2+3*4"),
});
```

### 代码示例

```typescript
import { tool } from "@langchain/core/tools";
import { HumanMessage, ToolMessage } from "@langchain/core/messages";

// 定义工具
const searchTool = tool(
  async ({ query }: { query: string }) => {
    return "搜索结果...";
  },
  {
    name: "search",
    description: "搜索网络信息",
    schema: z.object({
      query: z.string().describe("搜索关键词"),
    }),
  }
);

// 绑定工具到 LLM
const llmWithTools = llm.bindTools([searchTool]);

// 执行循环
async function runToolCallingAgent(question: string): Promise<string> {
  const messages: any[] = [new HumanMessage(question)];

  for (let i = 0; i < 5; i++) {
    const response = await llmWithTools.invoke(messages);

    // 无工具调用，返回最终答案
    if (!response.tool_calls || response.tool_calls.length === 0) {
      return response.content;
    }

    messages.push(response);

    // 执行工具调用
    for (const toolCall of response.tool_calls) {
      const result = await searchTool.invoke(toolCall.args);
      messages.push(new ToolMessage({
        content: result,
        tool_call_id: toolCall.id,
      }));
    }
  }

  return "达到最大迭代次数";
}
```

### Tool Calling vs ReAct

| 特性 | ReAct | Tool Calling |
|------|-------|--------------|
| **工具调用方式** | Prompt 引导输出文本 | 原生 Function Calling |
| **解析方式** | 正则提取 | 无需解析 |
| **参数定义** | 文本描述 | Zod schema |
| **可靠性** | 容易解析出错 | 更可靠 |

---

## 10. 总结与对比

### 学习路径

```
基础 → 进阶 → 实战
 │       │       │
 ▼       ▼       ▼
模型    链式调用   Agent
模板    任务拆解   工具调用
```

### 核心概念对比

| 概念 | 说明 | 适用场景 |
|------|------|----------|
| **PromptTemplate** | 参数化提示词 | 所有场景 |
| **LCEL (.pipe())** | 简单链式调用 | 2-3 个节点 |
| **RunnableSequence** | 复杂多步骤任务 | 4+ 个节点 |
| **Plan & Execute** | 先规划后执行 | 复杂多步骤任务 |
| **ReAct** | 思考→行动→观察循环 | 需要工具调用的场景 |
| **Tool Calling** | 原生 Function Calling | 需要工具调用的场景 |
| **LangGraph** | 有状态的工作流图 | 复杂工作流编排 |

### 技术选型建议

| 场景 | 推荐方案 |
|------|----------|
| 简单问答 | PromptTemplate + LCEL |
| 多步推理 | RunnableSequence |
| 复杂任务 | Plan & Execute |
| 工具调用 | Tool Calling Agent |
| 复杂工作流 | LangGraph |

---

## 📝 总结

本文从基础到进阶，系统讲解了 LangChain.js 的核心概念：

1. **基础**：模型创建、PromptTemplate、LCEL 链式调用
2. **进阶**：RunnableSequence 任务拆解、输出解析器
3. **实战**：Plan & Execute、ReAct Agent、Tool Calling Agent

掌握这些概念后，你就可以用 LangChain.js 构建各种 AI 应用了！🚀
