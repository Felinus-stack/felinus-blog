---
title: 《Redis数据类型与持久化机制》
tags:
    - Redis
    - 数据结构
    - 持久化
    - 缓存
    - NoSQL
categories:
    - Redis
    - 后端开发
date: "2026-03-29 15:30:07"
updated: "2026-03-29 15:30:07"
abbrlink: f8882818
---

# Redis 核心知识详解：数据类型与持久化机制

Redis 是目前企业开发中使用非常广泛的 **高性能 Key-Value 数据库**。

它最大的特点：

- 基于内存存储，读写速度极快
- 支持丰富的数据结构
- 支持数据持久化
- 支持分布式缓存场景

常见应用：

- 缓存
- 登录状态保存
- 分布式锁
- 消息队列
- 排行榜
- 限流
- 计数器

本文主要介绍：

1. Redis 常用数据类型
2. String 类型
3. Hash 类型
4. List 类型
5. Set 类型
6. ZSet 类型
7. Redis 持久化机制
   - RDB
   - AOF
   - 混合持久化

---

# 一、Redis 数据结构概览

Redis 不像 MySQL 一样使用表存储数据，而是：

```text
Key → Value
```

例如：

```text
user:1001

↓

{
    name:"Tom",
    age:20
}
```

其中：

- Key：数据唯一标识
- Value：实际存储的数据

Redis 支持多种 Value 类型：

| 类型 | 特点 | 应用场景 |
| --- | --- | --- |
| String | 字符串、数字 | 缓存、计数器 |
| Hash | 对象结构 | 用户信息 |
| List | 列表结构 | 消息队列 |
| Set | 无序集合 | 标签、去重 |
| ZSet | 有序集合 | 排行榜 |

---

# 二、String 类型

## 1. String 简介

String 是 Redis 最基础的数据类型。

它可以保存：

- 字符串
- 数字
- 二进制数据

例如：

```bash
SET username Tom
```

保存：

```text
key:

username


value:

Tom
```

获取：

```bash
GET username
```

结果：

```text
Tom
```

---

## 2. String 常用命令

### 设置数据

```bash
SET key value
```

例如：

```bash
SET name Tom
```

---

### 获取数据

```bash
GET key
```

例如：

```bash
GET name
```

结果：

```text
Tom
```

---

### 删除数据

```bash
DEL key
```

例如：

```bash
DEL name
```

---

### 设置过期时间

```bash
SET key value EX seconds
```

例如：

```bash
SET token abc123 EX 3600
```

表示：

```text
token 1小时后自动删除
```

---

# 3. String 应用场景

## （1）缓存数据

例如：

数据库：

```text
MySQL

user表
```

缓存：

```text
Redis

user:1001
```

流程：

```text
请求用户信息

↓

查询Redis

↓

存在

↓

直接返回


不存在

↓

查询MySQL

↓

写入Redis
```

---

## （2）计数器

例如：

文章阅读量：

```bash
INCR article:view:1001
```

第一次：

```text
1
```

第二次：

```text
2
```

应用：

- 点赞数量
- 浏览量
- 库存数量

---

# 三、Hash 类型

## 1. Hash 简介

Hash 类似 Java 中的：

```text
Map
```

结构：

```text
key

↓

field:value
```

例如：

用户对象：

```text
user:1001

{
    name:"Tom",
    age:20,
    email:"xxx@qq.com"
}
```

Redis：

```bash
HSET user:1001 name Tom

HSET user:1001 age 20
```

---

# 2. Hash 常用命令

## 添加字段

```bash
HSET key field value
```

例如：

```bash
HSET user:1 name Tom
```

---

## 获取字段

```bash
HGET key field
```

例如：

```bash
HGET user:1 name
```

返回：

```text
Tom
```

---

## 获取全部字段

```bash
HGETALL user:1
```

返回：

```text
name Tom

age 20
```

---

# 3. Hash 应用场景

## 用户信息缓存

MySQL：

```text
user表
```

Redis：

```text
user:1001
```

保存：

```text
name

age

email

avatar
```

优势：

不用存整个 JSON。

可以单独修改字段。

例如：

修改年龄：

```bash
HSET user:1001 age 21
```

---

# 四、List 类型

## 1. List 简介

List 是一个有序列表。

特点：

- 按插入顺序保存
- 可以重复
- 支持左右插入

结构：

```text
List

A

B

C

D
```

---

# 2. List 常用命令

## 左侧插入

```bash
LPUSH key value
```

---

## 右侧插入

```bash
RPUSH key value
```

---

## 获取列表

```bash
LRANGE key start end
```

例如：

```bash
LRANGE queue 0 -1
```

表示：

获取全部元素。

---

# 3. List 应用场景

## 消息队列

生产者：

```text
生产消息

↓

Redis List
```

消费者：

```text
读取消息

↓

处理任务
```

例如：

```text
订单创建

↓

加入队列

↓

异步发送邮件
```

---

# 五、Set 类型

## 1. Set 简介

Set 是无序集合。

特点：

- 不重复
- 无序

例如：

```text
Java

Python

Go
```

添加：

```bash
SADD language Java
```

---

# 2. Set 常用命令

## 添加元素

```bash
SADD key value
```

---

## 查看全部元素

```bash
SMEMBERS key
```

---

## 判断是否存在

```bash
SISMEMBER key value
```

---

# 3. Set 应用场景

## （1）去重

例如：

用户点赞：

```text
article_like_1

1001
1002
1003
```

同一个用户不能重复点赞。

---

## （2）共同好友

用户A：

```text
Tom

Jack
```

用户B：

```text
Tom

Lucy
```

求共同好友：

```bash
SINTER userA userB
```

结果：

```text
Tom
```

---

# 六、ZSet 类型（Sorted Set）

## 1. ZSet 简介

ZSet：

有序集合。

区别：

Set：

```text
无序
```

ZSet：

```text
按照 score 排序
```

结构：

```text
member

score
```

例如：

排行榜：

```text
Tom     100

Jack    90

Lucy    80
```

---

# 2. ZSet 常用命令

## 添加元素

```bash
ZADD key score member
```

例如：

```bash
ZADD rank 100 Tom
```

---

## 获取排行榜

```bash
ZRANGE key start end
```

例如：

```bash
ZRANGE rank 0 9
```

获取前10名。

---

## 获取分数

```bash
ZSCORE key member
```

---

# 3. ZSet 应用场景

## 排行榜

例如：

游戏排行榜：

```text
玩家A 10000分

玩家B 9000分

玩家C 8000分
```

Redis：

```text
game_rank

A 10000

B 9000

C 8000
```

根据 score 自动排序。

---

# 七、Redis 持久化机制

Redis 数据存储在内存中：

优点：

```text
速度快
```

缺点：

```text
服务器宕机

数据可能丢失
```

因此 Redis 提供：

- RDB
- AOF
- 混合持久化

---

# 八、RDB 持久化

## 1. 什么是 RDB？

RDB：

Redis Database

它会：

> 在某个时间点生成 Redis 数据快照。

例如：

```text
当前Redis数据

↓

生成dump.rdb文件
```

---

## 2. RDB 工作流程

```text
Redis运行

↓

满足保存条件

↓

fork子进程

↓

生成快照

↓

保存dump.rdb
```

---

## 3. RDB 优点

### （1）恢复速度快

因为：

直接加载二进制文件。

---

### （2）文件小

适合：

数据备份。

---

## 4. RDB 缺点

数据可能丢失。

例如：

设置：

```text
5分钟保存一次
```

如果：

```text
4分钟后服务器宕机
```

这4分钟数据：

可能丢失。

---

# 九、AOF 持久化

## 1. 什么是 AOF？

AOF：

Append Only File

它记录：

```text
每一次写操作命令
```

例如：

执行：

```bash
SET name Tom
```

保存：

```text
SET name Tom
```

---

## 2. AOF 工作流程

```text
客户端写入命令

↓

Redis执行

↓

记录日志文件
```

---

## 3. AOF 优点

- 数据安全性更高
- 丢失数据少

---

## 4. AOF 缺点

- 文件更大
- 恢复速度慢

---

# 十、AOF 三种同步策略

## always

每次写入立即同步。

优点：

安全。

缺点：

性能低。


---

## everysec（默认）

每秒同步一次。

特点：

性能和安全平衡。


---

## no

由操作系统决定。

性能最高。

风险较高。

---

# 十一、混合持久化

## 1. 为什么需要混合持久化？

RDB：

优点：

- 恢复快

缺点：

- 可能丢数据


AOF：

优点：

- 数据安全

缺点：

- 恢复慢


因此 Redis 4.0 引入：

```text
混合持久化
```

---

# 2. 混合持久化原理

流程：

```text
AOF重写

↓

先写RDB快照

↓

追加AOF增量命令
```

生成：

```text
RDB数据

+

AOF日志
```

---

# 3. 恢复流程

Redis启动：

```text
读取RDB部分

↓

加载基础数据

↓

执行AOF增量命令

↓

恢复最新状态
```

---

# 十二、三种持久化方式对比

|方式|原理|优点|缺点|
|-|-|-|-|
|RDB|定时快照|恢复快、文件小|可能丢数据|
|AOF|记录命令|数据安全|文件大、恢复慢|
|混合持久化|RDB+AOF|兼顾速度和安全|复杂度更高|

---

# 十三、面试回答模板

## Redis 常用数据类型有哪些？

回答：

> Redis 提供了丰富的数据结构，常用的包括 String、Hash、List、Set 和 ZSet。String 常用于缓存和计数器，Hash 适合存储对象信息，List 可以实现消息队列，Set 常用于去重和集合运算，ZSet 可以实现排行榜等有序场景。

---

## Redis 如何保证数据不丢失？

回答：

> Redis 提供 RDB 和 AOF 两种持久化方式。RDB 是通过生成数据快照进行持久化，恢复速度快但可能存在数据丢失；AOF 是记录每次写操作日志，数据安全性更高但文件较大。Redis 4.0 后支持混合持久化，将 RDB 快照和 AOF 增量日志结合，在保证恢复速度的同时减少数据丢失。

---

# 总结

Redis 核心知识结构：

```text
Redis数据类型

↓

String

Hash

List

Set

ZSet


Redis持久化

↓

RDB

↓

AOF

↓

混合持久化
```

掌握这些内容，可以覆盖后端开发中 Redis 的基础使用和常见面试问题。