---
title: "story-security-tests · security tests"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/security.test.ts#L1-L31)（远端行 1–31；私有仓库需要访问权限）。


## 结论

三项测试覆盖私网地址、本地模型精确白名单和图片签名；已在副本执行通过。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/security.test.ts`
- 符号：`security tests`
- 行号：1–31
- 文件 SHA-256：`cf96f8469e1ca980bfd98d39179163c50bf1d6c4c27ee962e99da1ccf8e808d3`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：1。

````ts
import assert from 'node:assert/strict';
import test from 'node:test';
import { isAllowedPrivateModelUrl, isPrivateOrReservedIp, validateOutboundBaseUrl, verifiedImageMime } from './security.ts';

test('出站地址拒绝私网、回环、链路本地和 IPv6 内网', () => {
  for (const ip of ['0.0.0.0','10.0.0.1','100.64.0.1','127.0.0.1','169.254.169.254','172.16.0.1','192.168.1.1','224.0.0.1','::1','fc00::1','fe80::1','::ffff:127.0.0.1']) {
    assert.equal(isPrivateOrReservedIp(ip), true, ip);
  }
  assert.equal(isPrivateOrReservedIp('8.8.8.8'), false);
  assert.throws(() => validateOutboundBaseUrl('https://127.0.0.1/v1'));
  assert.throws(() => validateOutboundBaseUrl('https://[::1]/v1'));
  assert.throws(() => validateOutboundBaseUrl('http://api.example.com/v1'));
  assert.equal(validateOutboundBaseUrl('https://api.example.com/v1/'), 'https://api.example.com/v1');
});

test('本地模型仅接受超级管理员配置的精确 Base URL 白名单', () => {
  const original = process.env.AI_LOCAL_BASE_URL_ALLOWLIST;
  process.env.AI_LOCAL_BASE_URL_ALLOWLIST = 'http://127.0.0.1:11434/v1';
  try {
    assert.equal(isAllowedPrivateModelUrl('http://127.0.0.1:11434/v1/'), true);
    assert.equal(isAllowedPrivateModelUrl('http://127.0.0.1:11435/v1'), false);
    assert.equal(isAllowedPrivateModelUrl('http://127.0.0.1:11434/admin'), false);
  } finally { if (original == null) delete process.env.AI_LOCAL_BASE_URL_ALLOWLIST; else process.env.AI_LOCAL_BASE_URL_ALLOWLIST = original; }
});

test('图片 MIME 必须与真实文件签名一致', () => {
  const png = new Uint8Array([0x89,0x50,0x4e,0x47,0x0d,0x0a,0x1a,0x0a,0,0,0,0]);
  assert.equal(verifiedImageMime(png, 'image/png'), 'image/png');
  assert.throws(() => verifiedImageMime(png, 'image/jpeg'));
  assert.throws(() => verifiedImageMime(new TextEncoder().encode('<html>metadata</html>'), 'image/png'));
});
````
