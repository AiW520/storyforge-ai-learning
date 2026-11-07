---
title: "02 · 目录、双存储与异步支路"
sidebar:
  order: 2
---

> **阅读版本：** 本课程含本地未提交修改，请先查看[源码版本与远端差异](../../../source-version/)。


> **学完能做什么：** 描述页面、原生 HTTP 服务、存储适配和 worker 的协作。

## 必要概念

适配器（adapter）把不同实现包装成同一业务接口；异步队列让请求提交和耗时执行分离，但增加失败、重试和一致性处理。

## 主链路

![StoryForge 模块](/diagrams/storyforge.svg)

[单独查看图示（可缩放）](/diagrams/storyforge.svg)

图的前四层是读者 HTTP 链路；BullMQ 是创作任务的独立支路，书架增删不需要经过模型队列。

| 目录/技术 | 已发现的实际作用 | 配合关系 |
| --- | --- | --- |
| app / components | Next 页面、React 客户端交互 | public-library 通过 publicApi 调用独立 API |
| apps/api/src/server.ts | Node http.createServer、集中 route | 同源检查、限流，再分派业务 |
| auth.ts / security.ts | 密码与会话、出站和来源校验 | 业务入口主动调用 |
| SQLite 查询 / postgres-publication-store | 同类业务两种持久化分支 | server 根据驱动选择 |
| prisma/schema.prisma / migrations | Postgres 模型与结构迭代 | 不替代 SQLite 初始化 |
| task-queue / worker | BullMQ 提交和消费耗时任务 | Redis 开关、taskId、重试与清理 |

## 数据模型和生命周期

User 与 Work 通过 BookshelfEntry 组成多对多关系。Prisma 模型用 userId/workId 复合主键防止同用户重复收藏，并声明 workId 索引与 userId/createdAt 索引。ReadingProgress 关联用户、作品和可空章节，章节删除采用 SetNull；书架关系用 Cascade。以上是模型声明，部署库是否已应用迁移需要数据库验证。

SQLite 列表只选 published、published_at 非空且至少存在一个已发布章节的作品；新增书架也核对可见性，INSERT OR IGNORE 处理已存在关系。删除只限定当前用户和作品，不会删除作品正文。

## 选型推测与维护成本

SQLite 便于单机入门，Postgres 更适合共享数据与服务化；这属于结构分析，不是作者已说明的动机。双实现增加行为漂移风险：改发布条件应在两种存储测试同一案例。BullMQ 的重试不等于业务副作用 exactly-once（恰好一次），需要核对生成结果落库与任务状态的事务边界。

## 源码定位

- [route · apps/api/src/server.ts:410–423](../../../evidence/story-route/)
- [listPublicBooks · apps/api/src/server.ts:758–774](../../../evidence/story-listing/)
- [addSqliteBookshelf / removeSqliteBookshelf · apps/api/src/server.ts:793–805](../../../evidence/story-shelf-sql/)
- [BookshelfEntry / ReadingProgress · prisma/schema.prisma:199–232](../../../evidence/story-schema/)
- [queueEnabled / enqueueGeneration · apps/api/src/task-queue.ts:60–84](../../../evidence/story-queue/)

## 常见误区

Prisma schema 里有索引，就断言当前 SQLite 库也有完全相同索引。

## 自检

收藏删除为什么不能直接 DELETE works？

<details>
<summary>展开参考思路</summary>

收藏是用户与作品的关系，删除作品会影响所有读者和作者。

</details>
