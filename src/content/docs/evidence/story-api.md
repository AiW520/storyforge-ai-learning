---
title: "story-api · publicApi / unwrapItems"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/components/public-library/library-api.ts#L87-L99)（远端行 87–99；私有仓库需要访问权限）。


## 结论

统一携带 Cookie；响应 JSON 解析失败返回 null，204 空响应不会触发 JSON 异常。

## 源码定位

- 项目：`storyforge`
- 相对路径：`components/public-library/library-api.ts`
- 符号：`publicApi / unwrapItems`
- 行号：87–99
- 文件 SHA-256：`c8dca297aab1e2c17584f24c262892a6cd72f09760da51a405c60f1ef8be71bd`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：87。

````ts
const API_BASE = (process.env.NEXT_PUBLIC_API_BASE_URL || "").replace(/\/$/, "");

export async function publicApi<T>(path: string, init?: RequestInit): Promise<T> {
  const method = init?.method?.toUpperCase() || "GET";
  const cache = init?.cache ?? (method === "GET" ? "no-cache" : undefined);
  const response = await fetch(`${API_BASE}${path}`, { ...init, ...(cache ? { cache } : {}), credentials: "include" });
  const payload = await response.json().catch(() => null) as ({ error?: string; message?: string } & T) | null;
  if (!response.ok) throw new PublicApiError(payload?.error || payload?.message || `请求失败（${response.status}）`, response.status);
  return payload as T;
}

export function unwrapItems<T>(payload: PagedResult<T> | T[]): T[] {
  return Array.isArray(payload) ? payload : Array.isArray(payload?.items) ? payload.items : [];
````
