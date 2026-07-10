---
title: SSE与WebSocket详解
tags:
    - SSE
    - WebSocket
    - 流式输出
    - 实时通信
    - AI
categories:
    - 前端开发
    - 网络通信
date: "2026-06-28 15:30:07"
updated: "2026-06-28 15:30:07"
abbrlink: f8882821
---


# SSE 和 WebSocket 详解（AI 流式输出核心技术）

在 AI 应用开发中，例如 ChatGPT、Claude、文心一言等产品，为什么回答不是等待几十秒后一次性返回，而是像打字一样逐字生成？

核心原因：

> 服务端生成一部分数据后，会立即推送给客户端。

实现这种实时通信主要有两种技术：

- SSE（Server-Sent Events）
- WebSocket


下面从：

- 原理
- 通信流程
- 代码实现
- 技术区别
- AI 应用场景

几个方面详细讲解。

---

# 一、为什么 AI 应用需要流式输出？

## 1.1 传统 HTTP 请求模式

传统 HTTP：

```text
用户请求

    ↓

服务器处理

    ↓

调用大模型

    ↓

等待模型生成完成

    ↓

一次返回全部结果
```

例如：

用户：

```text
介绍一下 Redis
```

模型生成：

```text
Redis 是一个开源的内存数据库...
```

假设生成需要 10 秒：

```text
0s ----------------10s

等待等待等待

一次显示全部
```

存在问题：

- 用户不知道是否卡死
- 首字节等待时间长
- 交互体验差


---

# 1.2 流式输出模式

改成：

```text
用户请求

↓

服务器调用大模型

↓

生成一个 token

↓

立即发送

↓

继续生成

↓

继续发送
```

效果：

```text
Redis

Redis 是

Redis 是一个

Redis 是一个开源

Redis 是一个开源数据库
```

这就是 ChatGPT 类产品的实时输出效果。

---

# 二、SSE（Server-Sent Events）


# 2.1 SSE 是什么？

SSE：

> Server-Sent Events，服务器发送事件。


简单理解：

> 客户端打开一个 HTTP 长连接，服务器可以不断向客户端发送消息。


通信方向：

```text
             HTTP连接


客户端  ----------------->  服务端


客户端  <-----------------  服务端

              持续推送数据
```


重点：

SSE 是：

> 单向通信

方向：

```text
服务器 → 客户端
```

---

# 2.2 SSE 工作原理


完整流程：

---

## 第一步：客户端建立连接


浏览器：

```javascript
const source =
new EventSource("/chat")
```


发送请求：

```http
GET /chat HTTP/1.1

Accept:text/event-stream
```

---

## 第二步：服务器保持连接


服务器：

不是立即返回：

```json
{
 "message":"hello"
}
```

而是：

保持 TCP 长连接。


响应类型：

```http
Content-Type:
text/event-stream
```

---

## 第三步：服务器不断发送事件


服务器发送：

```text
data: Redis


data: 是一个


data: 内存数据库
```


客户端接收：

```javascript
source.onmessage=function(event){

 console.log(event.data)

}
```


输出：

```text
Redis

是一个

内存数据库
```

---

# 三、SSE 在 AI 流式输出中的架构


典型架构：

```text
                 用户

                  |
                  |
              React/Vue

                  |
                  |
              EventSource

                  |
                  |
                SSE

                  |
                  |

          Node/Spring Boot

                  |

                  |

             OpenAI API

                  |

                  |

              GPT模型
```

---

# 四、SSE + OpenAI 实现


## 4.1 前端 Vue 示例


```javascript
const eventSource =
new EventSource(
 "/api/chat?question=Redis"
)


eventSource.onmessage=(event)=>{

    console.log(
        event.data
    )

}
```


收到：

```text
Redis
```


追加：

```javascript
answer += event.data
```


页面：

```text
Redis 是一个...
```

---

## 4.2 后端 Node.js 示例


安装：

```bash
npm install express openai
```


代码：

```javascript
app.get("/chat",
async(req,res)=>{


res.setHeader(
"Content-Type",
"text/event-stream"
)


res.setHeader(
"Cache-Control",
"no-cache"
)


const stream =
await openai.chat.completions.create({

model:"gpt-5",

messages:[
{
role:"user",
content:req.query.question
}
],

stream:true

})


for await(
const chunk of stream
){


const content =
chunk.choices[0]
.delta.content


if(content){

res.write(
`data:${content}\n\n`
)

}


}


res.end()


})
```

---

# 五、SSE 的核心特点


# 5.1 优点


## （1）基于 HTTP


无需额外协议。


```text
HTTP

 |

SSE
```


前端：

```javascript
EventSource
```


浏览器原生支持。

---

## （2）自动重连


如果：

```text
网络断开

↓

重新连接
```


浏览器自动恢复。

---

## （3）非常适合 AI 输出


因为 AI：

特点：

```text
服务器不断产生文本

↓

客户端不断接收
```


刚好匹配。


---

# 六、SSE 缺点


## 6.1 单向通信


只能：

```text
服务器

↓

客户端
```


不能：

```text
客户端

↓

服务器
```


如果需要：

- 鼠标位置同步
- 游戏操作
- 实时语音上传

不适合。

---

## 6.2 连接数量限制


浏览器对于 HTTP 长连接存在数量限制。

---

## 6.3 不适合高频二进制


例如：

图片：

```text
0101010101
```


视频：

```text
大量二进制数据
```


不适合。

---

# 七、WebSocket


# 7.1 WebSocket 是什么？


WebSocket：

> 一种支持客户端和服务器双向通信的协议。


区别：


## SSE

```text
服务器 ---> 客户端
```


## WebSocket

```text
服务器 <----> 客户端
```


支持：

> 双向通信

---

# 八、WebSocket 工作流程


## 第一步：HTTP 握手


开始：

还是 HTTP：

```http
GET /chat HTTP/1.1


Upgrade:websocket

Connection:Upgrade
```


告诉服务器：

我要升级协议。


---

## 第二步：协议升级


服务器返回：

```http
101 Switching Protocols
```


表示：

升级成功。


之后：

不再使用 HTTP。


变成：

```text
WebSocket协议
```

---

# 九、WebSocket 通信模型


连接建立：

```text
        WebSocket


客户端 ================= 服务端


        长连接
```


双方都可以发送。


例如：

客户端：

```text
发送：

你好
```


服务器：

```text
收到
```


服务器：

```text
发送：

你好，我是AI
```


客户端：

```text
收到
```

---

# 十、WebSocket AI 应用


例如：

## AI 实时语音助手


流程：

```text
用户说话

↓

语音流

↓

服务器

↓

AI处理

↓

实时回答

↓

语音返回
```


需要：

```text
双向实时通信
```


所以：

> WebSocket 更适合。

---

# 十一、WebSocket 示例


## 11.1 前端


```javascript
const ws =
new WebSocket(
"ws://localhost:8080/chat"
)


ws.onopen=()=>{

ws.send(
"介绍Redis"

)

}


ws.onmessage=(event)=>{

console.log(
event.data

)

}
```


---

## 11.2 Node 后端


安装：

```bash
npm install ws
```


代码：

```javascript
const WebSocket =
require("ws")


const server =
new WebSocket.Server({
port:8080
})


server.on(
"connection",
socket=>{


socket.on(
"message",
async msg=>{


socket.send(
"Redis 是"
)


socket.send(
"一个内存数据库"
)


})


})
```

---

# 十二、SSE 和 WebSocket 核心区别


| 对比项 | SSE | WebSocket |
|---|---|---|
| 通信方向 | 单向 | 双向 |
| 协议 | HTTP | WebSocket |
| 连接方式 | HTTP 长连接 | TCP 长连接 |
| 浏览器支持 | EventSource | WebSocket API |
| 复杂度 | 简单 | 复杂 |
| 实时性 | 高 | 更高 |
| 数据类型 | 文本 | 文本 + 二进制 |
| 自动重连 | 支持 | 需要自己实现 |
| 服务器压力 | 较低 | 较高 |


---

# 十三、AI 应用中如何选择？


# 场景 1：ChatGPT 文字聊天


推荐：

```text
SSE
```


原因：

```text
用户输入一次

↓

AI不断输出
```


通信方向：

```text
AI → 用户
```


---

# 场景 2：AI 语音助手


推荐：

```text
WebSocket
```


原因：

用户：

```text
持续发送语音
```


AI：

```text
持续返回声音
```


属于：

```text
双向通信
```

---

# 场景 3：AI Agent 任务执行


例如：

```text
分析

↓

搜索

↓

调用工具

↓

生成
```


推荐：

```text
SSE
```


原因：

用户主要查看执行过程：

```text
正在分析...

正在搜索...

正在生成...
```

---

# 十四、面试回答版本


面试问题：

> AI 应用中的流式输出如何实现？


回答：

> AI 流式输出主要通过 SSE 或 WebSocket 实现。SSE 基于 HTTP 长连接，由服务端持续向客户端推送事件，非常适合大模型文本生成场景。后端调用大模型 API 时开启 stream 模式，模型生成 token 后立即通过 SSE 返回，前端通过 EventSource 接收并逐步渲染。WebSocket 则提供全双工通信能力，适用于需要客户端和服务端实时交互的场景，例如 AI 语音助手、实时协作等。


---

# 十五、结合 AI 农业问答系统


实现流程：

```text
用户：

小麦叶片发黄怎么办？

        |

        ↓

后端调用 LLM

        |

        ↓

GPT生成：

"根据症状分析..."

        |

        ↓

SSE实时返回

        |

        ↓

Vue逐字显示
```


技术选型：

```text
Vue3

+

Spring Boot / Node

+

SSE

+

LangChain

+

LLM API
```


这是目前企业级 AI 问答系统最常见的实现方式。
