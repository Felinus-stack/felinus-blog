---
title: 《MySQL核心知识》
tags:
    - MySQL
    - 慢查询
    - 事务
    - ACID
    - SQL优化
categories:
    - 数据库
    - 后端开发
date: "2026-07-09 11:34:58"
updated: "2026-07-09 11:34:58"
abbrlink: f8882814
---
# MySQL 慢查询分析、事务 ACID 与事务隔离级别详解

在后端开发中，MySQL 性能优化和数据一致性是非常重要的两个方向。

其中：

- **慢查询分析** 主要解决数据库性能问题
- **事务 ACID** 保证数据操作的可靠性
- **事务隔离级别** 解决多个事务并发执行时的数据一致性问题


本文主要介绍：

1. MySQL 慢查询分析
2. MySQL 事务 ACID 原理
3. MySQL 事务隔离级别
4. 常见并发问题
5. 面试回答总结


---

# 一、MySQL 慢查询分析


## 1. 什么是慢查询？

慢查询：

> 执行时间超过指定阈值的 SQL 查询。


例如：

正常查询：

```sql
SELECT *
FROM user
WHERE id = 1;
```


执行时间：

```text
5ms
```


属于正常。


但是：

```sql
SELECT *
FROM user
WHERE username='Tom';
```


执行：

```text
5s
```


就属于慢查询。


---

# 2. 为什么会出现慢查询？


常见原因：

- 没有使用索引
- 索引失效
- 查询数据量过大
- 多表连接效率低
- 数据库设计问题


---

# （1）没有使用索引


例如：

用户表：

```text
user

id

username

age
```


查询：

```sql
SELECT *

FROM user

WHERE username='Tom';
```


如果 username 没有索引：

MySQL 需要：

```text
扫描全部数据

↓

逐条比较

↓

找到 Tom
```


数据量越大：

查询越慢。


---

# （2）索引失效


例如：

创建索引：

```sql
CREATE INDEX idx_name

ON user(username);
```


正常查询：

```sql
WHERE username='Tom'
```


可以使用索引。


但是：

```sql
WHERE LEFT(username,3)='Tom'
```


由于：

对字段进行了函数计算。


导致：

索引失效。


---

# （3）SQL 查询数据量过大


例如：

```sql
SELECT *

FROM user;
```


如果：

用户表有：

1000 万数据。


一次查询全部数据会造成：

- 大量磁盘 IO
- 网络压力
- 内存压力


---

# （4）多表连接效率低


例如：

```sql
SELECT *

FROM user u

JOIN orders o

ON u.id=o.user_id;
```


如果：

连接字段没有索引。


MySQL 需要大量匹配。


---

# （5）数据库设计问题


例如：

文章表：

```text
article

id

title

content
```


content：

使用：

```text
TEXT
```


存储大量内容。


如果：

频繁查询全部字段：

```sql
SELECT *

FROM article;
```


会导致：

大量数据读取。


---

# 二、MySQL 慢查询定位流程


实际开发中：

接口响应慢：

排查流程：

```text
接口响应慢

↓

查看 SQL

↓

定位慢 SQL

↓

分析执行计划

↓

优化 SQL

↓

添加索引

↓

再次测试
```


---

# 三、开启 MySQL 慢查询日志


## 1. 什么是慢查询日志？


慢查询日志：

> MySQL 用于记录执行时间超过指定阈值的 SQL。


默认：

关闭。


---

## 2. 查看是否开启


```sql
SHOW VARIABLES LIKE 'slow_query_log';
```


结果：

```text
ON

表示开启
```


或者：

```text
OFF

表示关闭
```


---

## 3. 设置慢查询时间


查看：

```sql
SHOW VARIABLES LIKE 'long_query_time';
```


例如：

```text
10
```


表示：

超过 10 秒记录。


修改：

```sql
SET GLOBAL long_query_time=1;
```


表示：

超过 1 秒记录。


---

## 4. 查看慢查询日志位置


```sql
SHOW VARIABLES LIKE 'slow_query_log_file';
```


例如：

```text
mysql-slow.log
```


---

# 四、使用 EXPLAIN 分析 SQL


EXPLAIN：

> 查看 MySQL 执行 SQL 时的执行计划。


例如：

```sql
EXPLAIN

SELECT *

FROM user

WHERE username='Tom';
```


---

# EXPLAIN 重要字段


## 1. type


表示：

访问类型。


性能排序：

```text
system

↓

const

↓

eq_ref

↓

ref

↓

range

↓

index

↓

ALL
```


重点关注：

```text
ALL
```


表示：

全表扫描。


---

## 2. possible_keys


表示：

可能使用的索引。


例如：

```text
idx_username
```


---

## 3. key


表示：

实际使用的索引。


例如：

```text
idx_username
```


如果：

```text
NULL
```


说明：

没有使用索引。


---

## 4. rows


表示：

扫描的数据行数量。


例如：

```text
1000000
```


说明：

扫描 100 万条数据。


---

## 5. Extra


额外信息。


常见：


### Using index

表示：

覆盖索引。


性能较好。


---

### Using filesort

表示：

额外排序。


例如：

```sql
ORDER BY age;
```


没有索引：

需要额外排序。


---

### Using temporary

表示：

使用临时表。


性能较差。


---

# 五、慢查询优化方法


## 1. 添加合理索引


例如：

查询：

```sql
SELECT *

FROM user

WHERE username='Tom';
```


添加：

```sql
CREATE INDEX idx_username

ON user(username);
```


---

## 2. 避免 SELECT *


不推荐：

```sql
SELECT *

FROM user;
```


推荐：

```sql
SELECT id,username

FROM user;
```


原因：

减少数据传输。


---

## 3. 分页优化


问题：

```sql
SELECT *

FROM user

LIMIT 100000,10;
```


offset 很大：

会扫描大量无用数据。


优化：

```sql
SELECT *

FROM user

WHERE id>100000

LIMIT 10;
```


---

## 4. 避免索引失效


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

# 六、MySQL 事务 ACID


## 1. 什么是事务？


事务：

> 一组数据库操作，要么全部成功，要么全部失败。


例如：

银行转账：

```text
A账户减少100

↓

B账户增加100
```


两个操作：

必须同时成功。


---

# 2. ACID 四大特性


ACID：

- Atomicity
- Consistency
- Isolation
- Durability


---

# 一、Atomicity（原子性）


## 含义：

> 事务中的操作要么全部完成，要么全部不完成。


例如：

转账：

```text
扣钱

+

加钱
```


如果：

扣钱成功。

但是：

加钱失败。


应该：

回滚。


实现：

```text
Undo Log
```


---

# 二、Consistency（一致性）


## 含义：

> 事务执行前后，数据库状态保持一致。


例如：

转账前：

```text
A:

1000


B:

1000
```


转账后：

```text
A:

900


B:

1100
```


总金额：

仍然：

```text
2000
```


---

# 三、Isolation（隔离性）


## 含义：

> 多个事务同时执行时，相互之间不能互相影响。


实现：

依靠：

- 锁机制
- MVCC


---

# 四、Durability（持久性）


## 含义：

> 事务提交后，数据永久保存。


执行：

```sql
COMMIT;
```


即使：

数据库重启。


数据仍然存在。


实现：

```text
Redo Log
```


---

# 七、MySQL 事务隔离级别


事务隔离级别：

> 控制多个事务并发执行时的数据可见性。


MySQL 支持：

4 种隔离级别。


---

# 1. READ UNCOMMITTED


读未提交。


特点：

最低隔离级别。


可以读取：

其他事务未提交的数据。


产生：

## 脏读


---

# 2. READ COMMITTED


读已提交。


只能读取：

已经提交的数据。


解决：

脏读。


但是存在：

不可重复读。


---

# 3. REPEATABLE READ


可重复读。


MySQL InnoDB 默认级别。


特点：

同一个事务：

多次读取结果一致。


解决：

- 脏读
- 不可重复读


---

# 4. SERIALIZABLE


串行化。


最高隔离级别。


特点：

事务串行执行。


优点：

数据一致性最高。


缺点：

性能最低。


---

# 八、三种并发问题


# 1. 脏读（Dirty Read）


定义：

> 一个事务读取了另一个事务未提交的数据。


流程：

```text
事务A修改数据

↓

事务B读取

↓

事务A回滚

↓

事务B读取错误数据
```


---

# 2. 不可重复读


定义：

> 同一个事务中，多次读取同一数据结果不同。


原因：

其他事务修改并提交。


---

# 3. 幻读


定义：

> 同一个事务中，两次查询得到的数据数量不同。


例如：

第一次：

```sql
SELECT *

FROM user

WHERE age>20;
```


结果：

```text
10条
```


另一个事务新增：

```text
age=25用户
```


第二次：

```text
11条
```


多出来的数据：

叫幻读。


---

# 九、事务隔离级别对比


| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
| --- | --- | --- | --- |
| READ UNCOMMITTED | 可能 | 可能 | 可能 |
| READ COMMITTED | 避免 | 可能 | 可能 |
| REPEATABLE READ | 避免 | 避免 | 可能 |
| SERIALIZABLE | 避免 | 避免 | 避免 |


---

# 十、MySQL 默认隔离级别


MySQL InnoDB：

默认：

```text
REPEATABLE READ
```


原因：

在性能和一致性之间取得平衡。


---

# 十一、面试回答模板


## 什么是慢查询？如何优化？


回答：

> 慢查询是指执行时间超过指定阈值的 SQL。排查慢查询时，我会先开启慢查询日志定位 SQL，然后通过 EXPLAIN 分析执行计划，查看是否走索引、扫描行数以及是否出现全表扫描。优化方式包括建立合理索引、避免索引失效、减少查询字段、优化分页以及调整 SQL 结构。


---

## 什么是事务 ACID？


回答：

> MySQL 事务具有 ACID 四个特性，分别是原子性、一致性、隔离性和持久性。原子性保证事务操作要么全部成功，要么全部失败；一致性保证数据满足业务规则；隔离性保证多个事务并发执行时互不影响；持久性保证事务提交后的数据不会丢失。


---

## MySQL 事务隔离级别有哪些？


回答：

> MySQL 提供 READ UNCOMMITTED、READ COMMITTED、REPEATABLE READ 和 SERIALIZABLE 四种隔离级别。InnoDB 默认使用 REPEATABLE READ，通过 MVCC 和锁机制保证事务一致性，可以避免脏读和不可重复读，同时兼顾性能。


---

# 总结


MySQL 性能与事务核心知识：

```text
慢查询分析

↓

慢 SQL 定位

↓

EXPLAIN 分析

↓

索引优化


事务

↓

ACID

↓

隔离级别

↓

解决并发问题
```


掌握：

- 慢查询分析
- EXPLAIN 执行计划
- 索引优化
- ACID
- 四种事务隔离级别
- 脏读、不可重复读、幻读


基本覆盖后端开发 MySQL 面试中的核心内容。