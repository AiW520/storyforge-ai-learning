---
title: "04 · 精读发布过滤与任务重试"
sidebar:
  order: 4
---

> **阅读版本：** 本课程含本地未提交修改，请先查看[源码版本与远端差异](../../../source-version/)。


> **学完能做什么：** 说明动态查询哪些部分受白名单控制，哪些部分需要参数绑定。

## 必要概念

参数绑定（parameter binding）让用户值作为数据传入 SQL；排序列通常不能用值占位符替代，需要服务端枚举选择固定表达式。

## listPublicBooks

输入为 URL 查询参数，输出为分页 JSON。search 截断至 120、genre 至 80，sort 只接受 hot/new/updated/favorite，page 最大 100，pageSize 最大 50。WHERE 语句由固定片段拼接，搜索和分类的值放在 params；排序从固定字符串中选择。

修改公开过滤会同时影响“是否上架”和“是否泄露草稿”。修改 ORDER BY 要同时考虑稳定分页，源码最后按 w.id 继续排序。LIKE 的搜索值经过 escapeLike，避免把用户的通配字符无意解释成模式；参数绑定与转义通配符解决的问题不同。

## publicApi

输入 path/RequestInit，默认 GET 使用 no-cache、所有请求 credentials=include。响应解析失败被转换为 null，随后检查 response.ok。这样支持 204，但泛型 T 只是 TypeScript 静态约定，不能替代运行时响应校验；调用方 unwrapItems 提供空数组兜底。

## enqueueGeneration

输入 taskId、chapterId、ownerId 与任务 body。先查询同 taskId 的现存作业：未完成就复用，已完成/失败先移除，再 add chapter.generate。重试最多 3 次，退避从 2 秒指数增长，完成/失败分别有保留时间与数量限制。

修改 taskId 生成规则会改变去重范围；把所有失败自动重试可能重复付费外部调用。已读代码只证明队列层策略，模型响应的恢复和账单补偿还需沿 worker/生成服务继续核对，本次没有远程调用证明。

## 源码定位

- [listPublicBooks · apps/api/src/server.ts:758–774](../../../evidence/story-listing/)
- [publicApi / unwrapItems · components/public-library/library-api.ts:87–99](../../../evidence/story-api/)
- [queueEnabled / enqueueGeneration · apps/api/src/task-queue.ts:60–84](../../../evidence/story-queue/)

## 常见误区

使用 ORM 或参数绑定后，就不需要检查动态排序、发布可见性和业务授权。

## 自检

把 pageSize 上限改成十万可能影响什么？

<details>
<summary>展开参考思路</summary>

查询和序列化内存、数据库负载、前端渲染及拒绝服务风险；应先做有上限的性能实验。

</details>
