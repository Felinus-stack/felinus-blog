---
title: AI 应用开发核心技术详解
tags:
    - AI
    - Prompt
    - LLM
    - 工作流
    - 流式输出
categories:
    - AI开发
date: "2026-07-10 14:00:00"
updated: "2026-07-10 14:00:00"
abbrlink: f444294702
---
# AI 应用开发核心技术详解

> 核心能力：
>
> - Prompt 编排
> - 模型调用
> - 流式输出
> - 上下文记忆
> - 结构化结果解析
> - 多步骤 AI 工作流构建


---
# 1. Prompt 编排（Prompt Orchestration）

## 1.1 什么是 Prompt？

Prompt（提示词）就是给大模型的输入指令。

例如：

普通 Prompt：

```text
介绍一下小麦赤霉病
```

大模型可能返回：

```text
小麦赤霉病是一种真菌病害...
```

但是如果希望 AI 输出更专业，可以设计：

```text
你是一名农业专家。

请根据以下要求回答：

1. 分析病害名称
2. 描述主要症状
3. 分析发病原因
4. 给出防治措施

用户问题：

{question}
```

这就是：

> Prompt Engineering（提示词工程）


---

# 1.2 什么是 Prompt 编排？

简单理解：

> 不是写一个 Prompt，而是管理多个 Prompt，让 AI 按流程工作。


例如：

一个农业 AI 诊断系统：

用户输入：

```text
我的小麦叶片出现黄色斑点
```

系统不会直接问：

```text
是什么病？
```

而是拆分多个步骤。


---

## Step1：提取用户描述中的关键信息


Prompt：

```text
你是农业病害分析助手。

从下面描述中提取：

- 作物
- 部位
- 症状
- 生长期

输入：

xxx

输出：

JSON格式
```


得到：

```json
{
  "crop":"小麦",
  "part":"叶片",
  "symptom":"黄色斑点"
}
```


---

## Step2：疾病分析 Prompt


输入：

```text
根据以下症状判断可能疾病：

症状：

小麦
叶片
黄色斑点


候选知识：

xxx
```


模型结合知识库：

```text
分析结果：

可能疾病：
小麦锈病
```


---

## Step3：生成解决方案 Prompt


输入：

```text
根据诊断结果生成农户容易理解的解决方案
```


输出：

```text
建议：

1. 使用对应药剂防治
2. 控制田间湿度
3. 加强田间管理
```


这整个过程就是：

> Prompt 编排。


---

# 1.3 Prompt 编排包含什么？


## （1）Prompt 模板化


不要写死：

```javascript
const prompt =
"分析小麦病害"
```


而应该：

```javascript
const prompt = `
你是一名{role}

请分析：

{question}

要求：

{format}
`
```


运行：

```javascript
prompt.format({

role:"农业专家",

question:"小麦叶片发黄",

format:"JSON"

})
```


生成：

```text
你是一名农业专家

请分析：

小麦叶片发黄

要求：

JSON
```


---

## （2）Prompt Chain（提示链）


多个 Prompt 串联：

```text
用户问题

↓

Prompt1
信息抽取

↓

Prompt2
知识检索

↓

Prompt3
答案生成

↓

返回用户
```


类似：

```text
Chain:

A → B → C → D
```


---

# 2. 模型调用（Model Invocation）


# 2.1 什么是模型调用？


模型调用：

> 就是你的程序调用大模型 API。


流程：

```text
你的后端

      HTTP请求

          ↓

    OpenAI API

          ↓

      GPT模型

          ↓

      返回结果
```


---

## 示例代码


```javascript
const response =
await openai.chat.completions.create({

 model:"gpt-5",

 messages:[

  {
   role:"user",
   content:"介绍Redis"
  }

 ]

})


console.log(response)
```


返回：

```json
{
 "content":
 "Redis是一种内存数据库..."
}
```


---

# 2.2 模型调用需要处理什么？


## ① 模型选择


不同模型：

| 模型 | 特点 |
|---|---|
| GPT-5 | 复杂推理 |
| GPT-4o | 多模态 |
| Claude | 长文本 |
| Qwen | 国产模型 |
| DeepSeek | 低成本 |


例如：


简单分类：

```text
小模型
```


复杂推理：

```text
大模型
```


---

## ② 参数控制


例如：

```json
{
 "temperature":0.7,
 "max_tokens":2000
}
```


---

## temperature


控制随机性。


### 低 temperature

例如：

```text
0.1
```


回答稳定：

```text
苹果是一种水果。
```


---

### 高 temperature


例如：

```text
1.0
```


回答更丰富：

```text
苹果像自然馈赠的小宇宙。
```


---

# 3. 流式输出（Streaming Output）


# 3.1 普通输出


传统方式：

```text
用户：

解释Redis


↓

等待10秒


↓

一次返回：

Redis是一种...
它支持...
它具有...
```


问题：

- 等待时间长
- 用户体验差


---

# 3.2 流式输出


类似 ChatGPT：

文字一个一个出现。


过程：

```text
用户

↓

模型生成

↓

Redis

↓

是一种

↓

内存数据库

↓

...
```


---

# 3.3 技术实现


通常使用：

- SSE（Server Sent Events）
- WebSocket


架构：

```text
Frontend

React/Vue

    |

SSE连接

    |

Backend

Node/Spring Boot

    |

LLM API
```


---

## 前端代码


```javascript
const eventSource =
new EventSource("/api/chat")


eventSource.onmessage=(event)=>{

 console.log(event.data)

}
```


---

## 后端发送数据


服务器不断发送：

```text
data: Redis

data: 是

data: 一个内存数据库
```


---

# 4. 上下文记忆（Context Memory）


# 4.1 为什么需要记忆？


大模型本身：

> 不知道以前聊天内容。


例如：

第一次：

```text
用户：

我叫张三
```


第二次：

```text
用户：

我叫什么？
```


模型：

```text
不知道
```


所以需要：

> Memory（记忆系统）


---

# 4.2 短期记忆


保存当前聊天：


```json
[
 {
  "role":"user",
  "content":"我叫张三"
 },

 {
  "role":"assistant",
  "content":"你好张三"
 }
]
```


下一次请求：

```text
用户：

今年天气怎么样？


历史：

用户叫张三
```


模型：

```text
你好张三，今年天气...
```


---

# 4.3 长期记忆


保存数据库。


例如：

MySQL：

| user_id | memory |
|---|---|
|001|喜欢Vue|
|001|正在开发农业AI系统|


流程：

```text
用户登录

↓

查询历史

↓

加入Prompt

↓

模型回答
```


---

# 4.4 常见记忆方案


## Redis


保存：

```text
user_id

chat_history
```


适合：

- 最近聊天记录
- 短期上下文


---

## 向量数据库


例如：

- Milvus
- Chroma
- Pinecone


保存：

- 用户知识
- 历史文档
- 聊天记录


通过：

Embedding


转换：

```text
文本

↓

向量

↓

相似搜索
```


---

# 5. 结构化结果解析（Structured Output Parsing）


# 5.1 为什么需要？


大模型默认输出：

```text
小麦可能感染赤霉病，需要喷药。
```


但是程序不好处理。


前端需要：

```json
{
 "disease":"",
 "probability":"",
 "solution":""
}
```


---

# 5.2 结构化输出


要求模型返回：

```json
{
"name":"赤霉病",

"confidence":0.85,

"treatment":[

 "喷施药剂",

 "降低湿度"

]
}
```


---

# 5.3 后端解析


Node：

```javascript
const result =
JSON.parse(response)
```


得到：

```javascript
result.name
```


直接使用。


---

# 5.4 常见技术


## JSON Schema


定义：

```json
{
"type":"object",

"properties":{

"disease":{

"type":"string"

}

}

}
```


模型必须符合该结构。


---

# 6. 多步骤 AI 工作流构建


这是目前 AI Agent 的核心能力。


---

# 6.1 什么是工作流？


简单理解：

> 让 AI 不只是回答问题，而是完成任务。


---

## 普通 ChatGPT


```text
用户问题

↓

回答
```


---

## AI Agent


```text
用户需求

↓

分析任务

↓

调用工具

↓

查询数据库

↓

调用模型

↓

生成结果

↓

执行动作
```


---

# 6.2 小麦病虫害 AI 系统示例


用户：

```text
我的小麦叶子发黄怎么办？
```


工作流：

```text
用户问题

        |

        ↓

症状信息提取 Agent

        |

        ↓

知识库检索 Agent

        |

        ↓

病害分类 Agent

        |

        ↓

防治方案生成 Agent

        |

        ↓

返回答案
```


---

# 6.3 LangGraph 工作流


类似：


```text
Node节点


A

|

B

|

C


Edge连接
```


示例：

```python
workflow.add_node(
 "search",
 search_database
)


workflow.add_node(
 "generate",
 llm_generate
)


workflow.add_edge(
 "search",
 "generate"
)
```


---

# 6.4 Agent 工作流


更智能：


```text
用户

↓

AI判断

↓

需要数据库？

      是

      ↓

调用数据库


需要搜索？

      是

      ↓

调用搜索工具


最后总结
```


---

# 7. 完整 AI 应用架构


农业 AI 问答系统：

```text
              用户

               |

               ↓

          React 前端

               |

               ↓

        Spring Boot / Node

               |

      ------------------

      |                |

 Prompt管理        Memory

      |                |

      ↓                ↓

    LLM模型        Redis/MySQL


               |

               ↓

        RAG知识库检索


               |

               ↓

        工作流引擎


               |

               ↓

          返回答案
```


---

# 8. 面试回答版本


## 面试问题：

> 你了解 AI 应用开发中的 Prompt 编排和工作流吗？


回答：


> 我了解。AI 应用开发中，我主要关注 Prompt 编排、模型调用和上下文管理。Prompt 编排主要是通过模板化 Prompt、多阶段 Prompt Chain 来控制模型输入输出流程；模型调用主要负责封装大模型 API，包括参数控制、异常处理和模型切换；流式输出通过 SSE 或 WebSocket 实现实时生成体验；上下文记忆通过 Redis、数据库或者向量数据库保存用户历史信息；结构化结果解析通过 JSON Schema 约束模型输出，方便业务系统消费；在复杂场景下，会使用 LangChain/LangGraph 等框架构建多步骤 AI Workflow，实现任务拆解、工具调用和自动执行。


---

# 简历项目描述


可以写：

> 熟悉基于 LangChain/LangGraph 的 AI 应用开发，能够完成 Prompt 工程、LLM 调用、RAG 检索增强、上下文记忆、结构化输出以及 Agent 工作流设计。


---

# 总结

以上能力属于当前 AI 应用开发岗位的核心技能：

|能力|作用|
|-|-|
|Prompt 编排|控制模型行为|
|模型调用|连接大模型能力|
|流式输出|提升交互体验|
|上下文记忆|实现连续对话|
|结构化解析|方便程序处理|
|Workflow|实现复杂任务自动化|

这些技术共同组成现代 AI 应用开发体系。