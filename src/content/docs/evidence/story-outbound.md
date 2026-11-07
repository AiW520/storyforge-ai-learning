---
title: "story-outbound · validateOutboundBaseUrl / secureFetch"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/security.ts#L11-L46)（远端行 11–46；私有仓库需要访问权限）。


## 结论

URL 结构、协议、DNS 与私网范围检查用于限制出站请求，白名单是显式例外。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/security.ts`
- 符号：`validateOutboundBaseUrl / secureFetch`
- 行号：11–46
- 文件 SHA-256：`c141b9839298575259274fae82a36547a329fdee8253acc3ec4c6394f581f842`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：11。

````ts
export function validateOutboundBaseUrl(value: string, options: OutboundOptions = {}) {
  let url: URL;
  try { url = new URL(value); } catch { throw securityError(400, 'base_url 无效'); }
  if (url.username || url.password) throw securityError(400, 'base_url 不得包含凭据');
  if (url.search || url.hash) throw securityError(400, 'base_url 不得包含查询参数或片段');
  if (url.protocol !== 'https:' && url.protocol !== 'http:') throw securityError(400, 'base_url 仅支持 HTTP(S)');
  if (url.protocol === 'http:' && !options.allowPrivateNetwork) throw securityError(400, 'base_url 必须使用 HTTPS；本地 HTTP 仅允许超级管理员白名单');
  if (isBlockedHostname(url.hostname) && !options.allowPrivateNetwork) throw securityError(400, 'base_url 不得指向本机或内部网络');
  if (isIP(stripIpv6Brackets(url.hostname)) && isPrivateOrReservedIp(stripIpv6Brackets(url.hostname)) && !options.allowPrivateNetwork) {
    throw securityError(400, 'base_url 不得指向私网、回环、链路本地或保留地址');
  }
  return value.replace(/\/+$/, '');
}

export async function assertSafeOutboundUrl(value: string | URL, options: OutboundOptions = {}) {
  const url = value instanceof URL ? value : new URL(value);
  if (url.username || url.password || !['https:', 'http:'].includes(url.protocol)) throw securityError(400, '上游地址不安全');
  if (url.protocol === 'http:' && !options.allowPrivateNetwork) throw securityError(400, '上游地址必须使用 HTTPS');
  const hostname = stripIpv6Brackets(url.hostname);
  if (isBlockedHostname(hostname) && !options.allowPrivateNetwork) throw securityError(400, '上游地址不得指向本机或内部网络');
  const addresses = isIP(hostname) ? [{ address: hostname }] : await lookup(hostname, { all: true, verbatim: true }).catch(() => []);
  if (!addresses.length) throw securityError(502, '上游地址无法解析');
  if (!options.allowPrivateNetwork && addresses.some(item => isPrivateOrReservedIp(item.address))) {
    throw securityError(400, '上游地址解析到私网、回环、链路本地或保留地址');
  }
}

export async function secureFetch(input: string | URL, init: RequestInit = {}, options: OutboundOptions = {}): Promise<Response> {
  const url = input instanceof URL ? input : new URL(input);
  await assertSafeOutboundUrl(url, options);
  return undiciFetch(url, { ...init, dispatcher: options.allowPrivateNetwork ? privateAgent : publicAgent } as Parameters<typeof undiciFetch>[1]) as unknown as Promise<Response>;
}

export function isAllowedPrivateModelUrl(value: string) {
  const candidate = normalizeAllowlistUrl(value);
  if (!candidate) return false;
````
