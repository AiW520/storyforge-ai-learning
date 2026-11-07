---
title: "00 · 创作和阅读为什么分开"
sidebar:
  order: 0
---

> **阅读版本：** 本课程含本地未提交修改，请先查看[源码版本与远端差异](../../source-version/)。


> **学完能做什么：** 从真实入口说明系统给作者与读者分别提供什么。

## 必要概念

发布快照（published snapshot）是面向读者的已公开内容；作者正在编辑的草稿不应因为共用一个作品编号而直接成为公开版本。

## 用途与用户

StoryForge 包含小说创作、角色与章节管理、模型任务、公开书城和个人书架。作者在创作界面编辑，读者浏览已发布作品；管理端处理平台配置。课程从不需要付费模型的读者流程入门，再理解后台生成模块的边界。

## 最短与深入路线

最短：总览 → 环境 → 三条读者流程 → 可见性实验。深入：Next 页面与组件 → publicApi → 原生 HTTP route → SQLite/Postgres 适配 → auth/security → task-queue 与 worker → 生成和发布扩展阅读。

先修：Promise、React useEffect、HTTP 状态码与 SQL。学完能解释一本书为什么出现在书城，书架状态如何变化，以及为何数据库切换不能只改一个连接字符串。

## 已核对边界

public-library 组件与 server.ts 的书城/书架链路已逐段阅读；认证、同源和队列入口有独立证据。模型生成的全部 prompt、质量评审、计费与图片对象存储未穷尽分析，也未调用真实模型。本课程不把 package.json 中每个依赖都称为已运行能力。

## 源码定位

- [Bookstore · useEffect · components/public-library/bookstore.tsx:26–55](../../evidence/story-bookstore/)
- [bookshelfMatch · apps/api/src/server.ts:522–539](../../evidence/story-shelf-route/)
- [addSqliteBookshelf / removeSqliteBookshelf · apps/api/src/server.ts:793–805](../../evidence/story-shelf-sql/)
- [queueEnabled / enqueueGeneration · apps/api/src/task-queue.ts:60–84](../../evidence/story-queue/)
- [scripts / dependencies · package.json:5–38](../../evidence/story-manifest/)

## 常见误区

Next.js 依赖存在，就认定所有后端逻辑都在 app/api。这个项目的主业务 API 在 apps/api/src/server.ts。

## 自检

公开列表为什么读取 published_title 而不是直接读取 title？

<details>
<summary>展开参考思路</summary>

它读取已发布快照，避免草稿编辑立即改变读者看到的内容。

</details>
