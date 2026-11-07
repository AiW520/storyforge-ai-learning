---
title: "源码版本与本地差异"
---

本课程依据已审阅的本地快照编写，包含未提交修改。远端仓库为 `AiW520/storyforge-ai`，核对提交为 `f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a`。**这不是远端 main 的无差异说明书。**

## 差异对学习的影响

被引用业务文件与远端一致。package.json 本地声明 Next 与 eslint-config-next 16.4.1，远端声明 16.2.12；sharp 本地为 0.35.5，远端为 0.35.3。运行时版本与安装行为需以对应版本的锁文件和安装验证为准。

## 如何使用版本证据

先阅读真实流程，再进入证据页。标记“与远端文件一致”的条目可对照固定提交；文件有差异时，只在片段逐行唯一匹配后提供远端行号。其余实现只代表本地快照。哈希（SHA-256）是内容指纹，文件改动后指纹会变化。

| 被引用文件 | 核对结果 |
|---|---|
| `apps/api/src/server.ts` | 与远端文件一致 |
| `components/public-library/bookstore.tsx` | 与远端文件一致 |
| `components/public-library/book-detail.tsx` | 与远端文件一致 |
| `components/public-library/library-api.ts` | 与远端文件一致 |
| `prisma/schema.prisma` | 与远端文件一致 |
| `apps/api/src/auth.ts` | 与远端文件一致 |
| `apps/api/src/security.ts` | 与远端文件一致 |
| `apps/api/src/task-queue.ts` | 与远端文件一致 |
| `apps/api/src/security.test.ts` | 与远端文件一致 |
| `package.json` | 本地文件存在差异 |

## 本地版本

- `storyforge`：提交 `f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a`；快照 `a6fb802783e06dc33f0025e8e7ca482591438c8249ddedbc4651f09286a89746`；未提交修改：True。

文件差异可能发生在引用片段之外；不能用“文件不同”推断所有流程都改变。教程的测试结果只属于注明的本地版本，不代表线上生产环境。

[逐文件哈希报告](/reports/remote-review.json) · [分析与验证](../coverage/)
