---
title: "story-origin · assertSameOriginRequest"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/security.ts#L51-L75)（远端行 51–75；私有仓库需要访问权限）。


## 结论

非只读请求核对浏览器来源，但无来源元数据请求及内部/回调路径有例外。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/security.ts`
- 符号：`assertSameOriginRequest`
- 行号：51–75
- 文件 SHA-256：`c141b9839298575259274fae82a36547a329fdee8253acc3ec4c6394f581f842`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：51。

````ts
export function assertSameOriginRequest(req: IncomingMessage, path: string) {
  const method = String(req.method || 'GET').toUpperCase();
  if (['GET', 'HEAD', 'OPTIONS'].includes(method) || path.startsWith('/api/billing/webhooks/') || path.startsWith('/api/internal/')) return;
  const origin = firstHeader(req.headers.origin);
  const referer = firstHeader(req.headers.referer);
  const fetchSite = firstHeader(req.headers['sec-fetch-site']).toLowerCase();
  if (!origin && !referer && !fetchSite) return;
  if (fetchSite === 'cross-site') throw securityError(403, '跨站写请求已被拒绝');
  const candidate = origin || referer;
  if (!candidate) throw securityError(403, '缺少同源校验信息');
  let requestOrigin: string;
  try { requestOrigin = new URL(candidate).origin; } catch { throw securityError(403, 'Origin 或 Referer 无效'); }
  const configured = `${process.env.APP_ORIGINS || ''},${process.env.CORS_ORIGIN || ''}`.split(',').map(item => item.trim()).filter(Boolean);
  const trustProxy = process.env.TRUST_PROXY === 'true';
  const forwardedProto = trustProxy ? firstHeader(req.headers['x-forwarded-proto']).split(',')[0]?.trim() : '';
  const protocol = forwardedProto || (process.env.NODE_ENV === 'production' ? 'https' : 'http');
  const host = (trustProxy ? firstHeader(req.headers['x-forwarded-host']).split(',')[0]?.trim() : '') || firstHeader(req.headers.host);
  const allowed = new Set(configured.map(item => { try { return new URL(item).origin; } catch { return ''; } }).filter(Boolean));
  if (process.env.NODE_ENV !== 'production' && !configured.length) {
    allowed.add('http://localhost:3000');
    allowed.add('http://127.0.0.1:3000');
  }
  if (host) allowed.add(`${protocol}://${host}`);
  if (!allowed.has(requestOrigin)) throw securityError(403, '跨站写请求已被拒绝');
}
````
