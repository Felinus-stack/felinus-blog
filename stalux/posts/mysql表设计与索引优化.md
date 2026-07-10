---
title: 《MySQL核心知识》
tags:
    - MySQL
    - 表设计
    - 索引优化
    - B+Tree
    - SQL优化
categories:
    - 数据库
    - 后端开发
date: "2026-02-06 11:34:58"
updated: "2026-02-06 11:34:58"
abbrlink: f8882813
---
# MySQL 表设计、字段类型、主外键关系与索引优化详解

数据库设计是后端开发中的核心能力。

一个优秀的数据库设计需要保证：

- 数据正确性
- 数据一致性
- 可维护性
- 查询性能
- 可扩展性

本文重点介绍：

1. 表结构设计
2. 字段类型选择
3. 主键与外键关系设计
4. MySQL 索引优化


---

# 一、MySQL 表结构设计


## 1. 什么是表设计？

表设计就是：

> 根据业务需求，将现实中的业务实体抽象成数据库中的表，并设计字段、关系以及约束。


例如：

一个电商系统包含：

- 用户
- 商品
- 订单
- 订单详情
- 商品分类


对应数据库：

```text
user 用户表

product 商品表

order 订单表

order_item 订单详情表

category 商品分类表
```


---

# 2. 表设计基本原则


## 原则一：一个表表示一个核心实体


例如：

用户：

```text
user
```


商品：

```text
product
```


订单：

```text
order
```


不要设计：

```text
user_product_order
```


这种大宽表。


原因：

- 数据重复
- 更新困难
- 查询复杂
- 扩展性差


---

# 3. 根据业务分析实体关系


数据库设计第一步：

## 找出业务实体


例如博客系统：

实体：

用户：

```text
User
```


文章：

```text
Article
```


评论：

```text
Comment
```


点赞：

```text
Like
```


关系：

```text
User

1

↓

N

Article


Article

1

↓

N

Comment
```


---

# 二、字段类型选择


字段类型选择会影响：

- 存储空间
- 查询效率
- 索引性能


MySQL 常见数据类型：

- 数值类型
- 字符串类型
- 时间类型


---

# 1. 数值类型


## INT


适合：

- 普通 ID
- 数量
- 状态值


例如：

```sql
id INT
```


---

## BIGINT


适合大型系统。


例如：

```sql
user_id BIGINT
```


互联网项目通常：

```text
BIGINT + 自增ID
```


作为主键。


---

## TINYINT


小整数类型。


常用于状态字段：

```sql
status TINYINT
```


例如：

```text
0 未删除

1 正常

2 禁用
```


优点：

占用空间小。


---

# 2. 字符串类型


## VARCHAR


最常用。


例如：

```sql
username VARCHAR(50)
```


特点：

- 可变长度
- 节省存储空间
- 查询效率较高


适用于：

- 用户名
- 邮箱
- 地址


---

## CHAR


固定长度字符串。


例如：

```sql
gender CHAR(1)
```


适合：

- 性别
- 状态码


---

## TEXT


大文本类型。


例如：

```sql
content TEXT
```


适合：

- 文章内容
- 描述信息


注意：

TEXT 不适合作为索引字段。


---

# 3. 时间类型


## DATETIME


常用时间类型。


例如：

```sql
create_time DATETIME
```


保存：

```text
2026-07-09 10:30:00
```


---

## TIMESTAMP


特点：

- 自动转换时区
- 存储空间小


适合：

- 日志时间
- 创建时间


---

# 4. 常用字段设计


实际项目中，大部分表都会包含：

```sql
id BIGINT PRIMARY KEY

create_time DATETIME

update_time DATETIME

deleted TINYINT
```


例如：

```sql
CREATE TABLE user(

 id BIGINT PRIMARY KEY,

 username VARCHAR(50),

 password VARCHAR(100),

 create_time DATETIME,

 update_time DATETIME,

 deleted TINYINT DEFAULT 0

);
```


---

# 三、主键设计


## 1. 什么是主键？


主键：

> 唯一标识一条数据的字段。


例如：

| id | username |
|---|---|
|1|Tom|
|2|Jack|


id 就是主键。


---

# 2. 主键特点


## 唯一性

不能重复：

```text
1

1 ❌
```


---

## 非空

不能：

```text
NULL
```


---

## 稳定

主键不应该随意修改。


---

# 3. 主键选择


## 自增 ID


例如：

```sql
id BIGINT AUTO_INCREMENT
```


优点：

- 简单
- 查询快
- B+Tree 顺序插入


缺点：

- 容易暴露数据量


---

## UUID


例如：

```text
550e8400-e29b
```


优点：

- 全球唯一


缺点：

- 长度大
- 索引效率低


---

实际项目：

通常选择：

```text
BIGINT 自增ID
```


---

# 四、外键关系设计


外键：

> 用于表示两个表之间的关联关系。


---

# 1. 一对多关系


最常见。


例如：

一个用户：

```text
User
```


拥有多个订单：

```text
Order
```


关系：

```text
User

1

↓

N

Order
```


设计：

用户表：

```text
user

id

username
```


订单表：

```text
order

id

user_id

price
```


其中：

```text
order.user_id

关联

user.id
```


---

# 2. 多对多关系


例如：

学生和课程。


关系：

```text
Student

N

↓

N

Course
```


需要中间表：

```text
student_course
```


结构：

|student_id|course_id|
|-|-|
|1|100|
|1|101|
|2|100|


---

# 3. 外键作用


## 保证数据一致性


避免：

```text
订单 user_id=999

但是用户不存在
```


---

## 维护关联关系


删除用户时：

需要考虑：

- 级联删除
- 逻辑删除


企业中更常用：

```text
deleted 字段逻辑删除
```


---

# 五、MySQL 索引优化


索引：

> 类似书籍目录，用于快速定位数据。


---

# 1. 没有索引的问题


查询：

```sql
SELECT *

FROM user

WHERE username='Tom';
```


数据库：

```text
扫描所有数据
```


---

# 2. 有索引


执行：

```text
索引

↓

快速定位

↓

数据
```


---

# 3. MySQL 索引结构


InnoDB 默认：

```text
B+Tree
```


结构：

```text
        根节点

           ↓

     非叶子节点

           ↓

       叶子节点
```


优势：

- 查询快
- 范围查询快
- 支持排序


---

# 4. 创建索引


例如：

用户名经常查询：

```sql
CREATE INDEX idx_username

ON user(username);
```


查询：

```sql
SELECT *

FROM user

WHERE username='Tom';
```


---

# 5. 索引优化原则


## 原则一：高频查询字段建立索引


例如：

```sql
WHERE username
```


建立索引。


不要：

所有字段都建立索引。


原因：

索引：

- 占用空间
- 增加写入成本


---

# 原则二：联合索引遵循最左匹配原则


创建：

```sql
CREATE INDEX idx_user

ON user(name,age);
```


索引顺序：

```text
name

↓

age
```


有效：

```sql
WHERE name='Tom'
```


有效：

```sql
WHERE name='Tom'

AND age=20
```


无效：

```sql
WHERE age=20
```


原因：

跳过了 name。


---

# 原则三：避免索引失效


## 1. 对字段进行函数操作


错误：

```sql
WHERE YEAR(create_time)=2026
```


优化：

```sql
WHERE create_time

BETWEEN

'2026-01-01'

AND

'2026-12-31'
```


---

## 2. 左模糊查询


索引失效：

```sql
LIKE '%Tom'
```


有效：

```sql
LIKE 'Tom%'
```


---

## 3. 隐式类型转换


字段：

```sql
phone VARCHAR(20)
```


错误：

```sql
WHERE phone=123456
```


正确：

```sql
WHERE phone='123456'
```


---

# 6. 覆盖索引


例如：

```sql
SELECT username

FROM user

WHERE username='Tom'
```


如果 username 已存在索引中：

数据库无需查询原表。


称为：

```text
覆盖索引
```


优势：

减少 IO。


---

# 7. 使用 EXPLAIN 分析 SQL


例如：

```sql
EXPLAIN

SELECT *

FROM user

WHERE username='Tom';
```


重点关注：

|字段|说明|
|-|-|
|type|访问类型|
|possible_keys|可能使用索引|
|key|实际使用索引|
|rows|扫描行数|
|Extra|额外信息|


---

# 六、数据库设计完整流程


```text
需求分析

↓

确定业务实体

↓

设计数据表

↓

选择字段类型

↓

设计主外键关系

↓

添加索引

↓

SQL性能分析

↓

持续优化
```


---

# 七、面试回答模板


面试官：

> 你如何设计 MySQL 表？


回答：

> 在数据库设计时，我会首先根据业务需求抽象实体，划分不同的数据表，保证一个表对应一个核心业务对象。字段设计方面会根据数据特点选择合适的数据类型，例如 ID 使用 BIGINT，状态字段使用 TINYINT，文本使用 VARCHAR 或 TEXT。同时通过主键保证数据唯一性，通过外键维护表之间的一对多、多对多关系。在性能优化方面，会针对高频查询字段建立索引，合理设计联合索引，并使用 EXPLAIN 分析 SQL 执行计划，避免索引失效，提高查询效率。


---

# 总结


MySQL 数据库设计核心流程：

```text
业务分析

↓

实体拆分

↓

字段设计

↓

主键设计

↓

外键关系

↓

索引优化

↓

SQL性能分析
```


掌握这些内容，可以覆盖后端开发中 MySQL 数据库设计的大部分核心场景。