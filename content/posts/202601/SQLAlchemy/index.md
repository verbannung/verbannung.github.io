---
title: "SQLAlchemy核心架构与缓存体系"
date: 2026-01-17
draft: false
tags: ["SQLAlchemy"]
categories: ["python","orm"]
---



# SQLAlchemy 核心架构与缓存体系

## 一、Session 的本质定义

Session 是工作单元控制器（Unit of Work Controller），不是查询缓存。

### 1.1 Session 的核心职责

- 事务管理：管理 begin/commit/rollback 生命周期
- Identity Map：维护对象唯一性（同一主键只有一个实例）
- 变更追踪：跟踪对象的 new/dirty/deleted 状态
- flush 协调：将内存变更转换为 SQL 语句

### 1.2 Session 缓存职责区分

| 缓存类型                  | Session 是否负责 | 说明                                       |
| ------------------------- | ---------------- | ------------------------------------------ |
| Identity Map（对象缓存）  | 负责             | 按主键存储对象，保证同一主键只有一个实例   |
| Query Cache（查询缓存）   | 不负责           | 不按查询条件缓存，相同 SQL 仍会执行        |

---

## 二、连接池与 Session 的关系

### 2.1 架构

多个 Session 共用同一个 Engine 的连接池，但不共用缓存：

```text
create_engine()
    |
QueuePool (共享连接池)
+-- pool_size=5, max_overflow=10
+-- 连接 1, 2, 3, ...
    |
Session 1 --> Identity Map 1 (独立缓存)
Session 2 --> Identity Map 2 (独立缓存)
Session 3 --> Identity Map 3 (独立缓存)
```

### 2.2 关键结论

| 组件         | 是否共享 | 说明                                 |
| ------------ | -------- | ------------------------------------ |
| 连接池       | 共享     | 多个 Session 从同一 Pool 获取连接    |
| Identity Map | 独立     | 每个 Session 有自己的对象缓存        |
| 事务状态     | 独立     | 每个 Session 管理自己的事务          |

---

## 三、Identity Map 的核心价值

### 3.1 解决的问题

Identity Map 解决的不是性能问题（避免 SQL），而是一致性问题（避免同一数据有多个不同副本）。

### 3.2 核心价值

- 内存一致性：同一数据库行在内存中只有一个对象，避免修改冲突
- 变更追踪：所有修改集中在一个对象上，Session 能准确追踪
- 关系一致性：order.user 和 session.get(User, 1) 返回同一对象
- 写入合并：多次修改同一对象，flush 时合并为一条 UPDATE
- rollback 恢复：只需恢复一个对象的状态

### 3.3 同一 Session 内对象唯一性

```python
session = Session()

user = session.query(User).filter(User.id == 1).first()
user_again = session.query(User).filter(User.id == 1).first()

# SQL 执行了 2 次（没有 Query Cache）
# 但 user is user_again 为 True（Identity Map 保证对象唯一）
```

在同一个 Session 内，无论查询多少次，无论怎么传递，同一主键永远是同一个 Python 对象。

---

## 四、缓存清空机制

### 4.1 expire_on_commit 默认行为

expire_on_commit=True（默认）：commit 后对象属性被过期

```python
session.commit()
# 对象仍在 Identity Map，但属性被标记为过期
# 下次访问属性时，自动触发懒加载查询
```

### 4.2 相同 SQL 查询的行为

下一次查询相同 SQL，不执行 update，是否会从缓存中拿到对象？

- Session 不是 query cache，仍然执行 SQL
- 但返回的对象是同一实例（Identity Map 保证）

唯一例外：session.get(User, 1) 不执行 SQL，直接从 Identity Map 返回

---

## 五、三层缓存体系

### 5.1 ClassManager（类级别）

- 职责：映射配置、属性定义、加载策略
- 缓存：无（配置全局不变）

### 5.2 InstanceState（对象级别）

- 职责：对象生命周期、脏检测、Identity Map
- 缓存：同一主键只有一个对象实例
- 用途：复用查询结果

```python
insp = inspect(user)
print(insp.persistent)   # 是否持久化
print(insp.modified)     # 是否被修改
```

### 5.3 AttributeState（属性级别）

- 职责：属性值追踪、修改历史、懒加载
- 缓存：属性值 + 修改历史 `([新], [旧], [未加载])`
- 用途：支持回滚恢复

```python
attr_state = insp.attrs.age
print(attr_state.value)    # 当前值
print(attr_state.history)  # ([40], [30], [])
```

---

## 六、四层事务与回滚

### 6.1 四层追踪体系

```text
Layer 4 (事务层)：Session + Unit of Work
    +-- 回滚时：数据库 ROLLBACK
        |
Layer 3 (对象层)：InstanceState + Identity Map
    +-- 回滚时：清除 modified 状态
        |
Layer 2 (属性层)：AttributeState + 修改历史
    +-- 回滚时：从历史恢复旧值
        |
Layer 1 (类层)：ClassManager
    +-- 无变化
```

### 6.2 回滚后缓存状态

| 对象类型                      | 回滚后状态            | 缓存是否有用 | 需要操作       |
|---------------------------| --------------------- | ------------ | -------------- |
| Persistent (数据库中已存在的对象)   | 仍在 Identity Map     | 部分有用     | expire()       |
| Pending (新增但未 commit 的对象) | 从 Session 移除       | 无用         | 重新 add()     |
| Modified (SAVEPOINT)      | 被 expire             | 无用         | 自动重新加载   |
| Unmodified (SAVEPOINT)    | 保留                  | 有用         | 无             |


---

## 七、并发与线程安全

### 7.1 Session 不是线程安全的

官方文档：Session 不能在多个线程之间共享，每个线程应该有自己的 Session。

### 7.2 并发场景

| 场景                   | 是否安全   | 说明                                   |
| ---------------------- | ---------- | -------------------------------------- |
| 单线程单 Session       | 安全       | 正常使用                               |
| 多线程共用 Session     | 不安全     | 禁止这样做                             |
| 多线程各自 Session     | 部分安全   | 需要处理数据库层并发（锁）             |
| 异步任务共用 Session   | 不安全     | 每个 task 用独立 AsyncSession          |

### 7.3 并发修改解决方案

```python
# 方案 1：乐观锁（version 字段）
class User(Base):
    version = Column(Integer, default=1)
    __mapper_args__ = {"version_id_col": version}

# 方案 2：悲观锁（SELECT FOR UPDATE）
user = session.query(User).filter(User.id == 1).with_for_update().first()
```

---

## 八、架构总结

### 8.1 核心架构理解

1. 属性层追踪变化
    - AttributeState (Layer 2) 维护修改历史
    - 历史格式：`([新值], [旧值], [未加载值])`

2. 回滚版本判断依赖四层协同
    - 属性历史提供旧值
    - 对象状态标记是否修改
    - 事务层执行数据库回滚

3. 对象层主要职责是复用查询结果
    - Identity Map 保证同一主键只有一个对象实例
    - 多次查询返回同一 Python 对象

### 8.2 Session 定位

Session 是工作单元控制器，负责：

- 协调对象与数据库之间的同步
- 管理事务边界
- 维护对象唯一性（Identity Map）

---

## 参考文档

- Session 基础：<https://docs.sqlalchemy.org/20/orm/session_basics.html>
- 事务管理：<https://docs.sqlalchemy.org/20/orm/session_transaction.html>
- 连接池：<https://docs.sqlalchemy.org/20/core/pooling.html>
- ORM 内部：<https://docs.sqlalchemy.org/20/orm/internals.html>
