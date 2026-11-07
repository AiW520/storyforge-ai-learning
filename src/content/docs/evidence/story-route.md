---
title: "story-route · route"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/server.ts#L410-L423)（远端行 410–423；私有仓库需要访问权限）。


## 结论

原生 Node HTTP 路由先检查同源和限流；readiness 分别检查数据库与队列。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/server.ts`
- 符号：`route`
- 行号：410–423
- 文件 SHA-256：`acd27a1a3dc2da74ab6d33865f41964b454445fa68eb298090adb8e2a020a225`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：410。

````ts
async function route(req: IncomingMessage, res: ServerResponse) {
  const method = req.method || 'GET';
  const url = new URL(req.url || '/', `http://${req.headers.host || 'localhost'}`);
  const path = url.pathname.replace(/\/+$/, '') || '/';
  assertSameOriginRequest(req, path);
  await applyRateLimit(req, path);
  if (method === 'OPTIONS') return sendEmpty(res, 204);
  if (method === 'GET' && (path === '/health' || path === '/health/live')) return sendJson(res, 200, { ok: true, service: 'storyforge-api' });
  if (method === 'GET' && path === '/health/ready') {
    const state = await readiness({
      database: () => postgresAuth ? postgresAuth.ping() : Boolean(db.prepare('SELECT 1 AS ok').get()),
      redis: () => queueReady(),
    });
    return sendJson(res, state.ok ? 200 : 503, { ...state, service: 'storyforge-api' });
````
