---
title: "01 · 独立 API 与前端的启动"
sidebar:
  order: 1
---

> **阅读版本：** 本课程含本地未提交修改，请先查看[源码版本与远端差异](../../../source-version/)。


> **学完能做什么：** 区分最小 SQLite 学习环境与 PostgreSQL/Redis 的扩展环境。

## 必要概念

环境变量（environment variable）在启动或构建时传入配置；NEXT_PUBLIC_ 前缀的配置会进入浏览器，绝不能存服务端密钥。

## 版本与配置

apps/api/package.json 声明 Node ≥22.6，入口通过 --experimental-strip-types 运行 TypeScript；根 package 声明 Next 16.4.1、React ^19.2.0、Prisma ^7.9.0。具体最低版本仍需结合解析依赖核对，本次使用 Node 22.18 只执行安全单元测试，没有验证完整 Next 构建。

| 配置 | 职责与实验值原则 |
| --- | --- |
| DATABASE_DRIVER | sqlite（默认）或 postgres |
| DATABASE_PATH | 副本专用 SQLite 文件；相对路径以 API 进程 cwd 解析 |
| DATABASE_URL | 仅 PostgreSQL 模式，必须指向实验库 |
| HOST / PORT | API 默认 127.0.0.1 / 8787 |
| NEXT_PUBLIC_API_BASE_URL | 浏览器调用 API 的地址；不能放密钥 |
| APP_ORIGINS / CORS_ORIGIN | 允许的前端来源，注意 localhost 与 127.0.0.1 不同 |
| REDIS_URL | 配置后启用 BullMQ，需要另起 worker |

## 参考启动步骤（完整服务未执行）

复制允许的源码、包清单和锁文件到副本，创建全新 .env（不要复制现有值）。根目录 `npm ci`；API 如有独立锁文件，在 apps/api 中执行对应安装命令。先读安装生命周期脚本。

```sh
npm run dev:api
# 另一个终端
npm run dev
```

SQLite 模式的初始化在 server.ts；Postgres 模式另需 `npm run infra:up`、`npm run db:generate`、`npm run db:migrate:deploy`。这组命令会创建/修改数据库，只在专用实验环境执行。不要对真实数据直接运行 seed。

浏览器访问前端书城；新库无公开作品时出现空状态是正常结果。`/health/live` 只证明进程可用，`/health/ready` 检查数据库及配置启用的队列。停止前端/API用 Ctrl+C，专用 compose 用 `npm run infra:down`。

401 → 检查 Cookie 与登录；403 → 核对 Origin、代理信任与账号状态；空书城 → 确认作品和章节均已发布；数据库驱动错误 → DATABASE_DRIVER 只能是 sqlite/postgres。不要通过关闭鉴权或把私网白名单设为任意地址来排错。

## 源码定位

- [scripts / dependencies · package.json:5–38](../../../evidence/story-manifest/)
- [route · apps/api/src/server.ts:410–423](../../../evidence/story-route/)
- [loginUser / createSession / currentUser · apps/api/src/auth.ts:34–73](../../../evidence/story-auth/)
- [queueEnabled / enqueueGeneration · apps/api/src/task-queue.ts:60–84](../../../evidence/story-queue/)

## 常见误区

Redis 可连就表示 worker 已经处理任务。健康检查与消费链路需要分别验证。

## 自检

为什么登录接口成功后书架仍可能 401？

<details>
<summary>展开参考思路</summary>

检查浏览器是否在正确域名保存并发送 Cookie、API 基址及 credentials 设置。

</details>
