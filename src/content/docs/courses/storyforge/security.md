---
title: "05 · 安全验证与动手实验"
sidebar:
  order: 5
---

> **阅读版本：** 本课程含本地未提交修改，请先查看[源码版本与远端差异](../../../source-version/)。


> **学完能做什么：** 用实际测试区分地址校验、图片签名与全链路安全。

## 必要概念

SSRF（Server-Side Request Forgery，服务端请求伪造）利用服务端网络权限访问不该访问的目标；CSRF（Cross-Site Request Forgery，跨站请求伪造）利用浏览器自动携带的身份凭据。

## 安全矩阵

| 主题 | 状态 | 具体边界 |
| --- | --- | --- |
| 密码/会话 | 已发现实现 | Argon2 验证、随机会话、数据库摘要、HttpOnly/Lax、生产 Secure |
| 对象权限 | 已发现实现 | 书架使用登录用户编号，加入前验证作品可见性 |
| SQL 注入 | 已发现实现 | 本课 SQLite 查询参数绑定，排序枚举 |
| CSRF | 已发现部分实现 | assertSameOriginRequest；无来源元信息、内部与支付回调有豁免，需核对各入口专属认证 |
| SSRF | 已发现实现并有局部测试 | 协议、凭据、私网、DNS、连接代理；白名单放行必须严格控制 |
| 上传 | 已发现局部实现 | verifiedImageMime 核对签名；完整上传大小、对象访问与文件生命周期另需审计 |
| XSS、全局日志、依赖 | 尚未全面确认 | 不把 React 默认转义推广到所有富文本和 HTML 展示 |
| TLS、代理与 Cookie 域 | 依赖部署 | TRUST_PROXY、允许来源与网络拓扑需要一致 |

## 已执行的隔离验证

复制 security.ts/security.test.ts 到实验目录，在 Node 22.18 使用 `node --experimental-strip-types --test apps/api/src/security.test.ts`，**3 项通过**：私网/IPv6/HTTP 拒绝、精确本地模型白名单、图片签名与声称类型一致。依赖解析复用了当前机器已有包，不能称为干净环境全量安装验证。没有启动业务 API，也未执行生产集成测试。

## 实验一：白名单边界

在副本测试中把允许地址设为 `http://127.0.0.1:11434/v1`，分别断言该地址及尾斜杠通过、不同端口和 /admin 不通过；补一个带用户名密码的 URL，validateOutboundBaseUrl 应拒绝。运行上述测试命令，预期新断言通过。恢复仅撤销副本测试改动；环境变量在 finally 恢复。

## 实验二：可见性和重复收藏

用合成数据库夹具建立用户、草稿作品与未发布章节。草稿不应进入公开列表；仅作品发布而章节未发布仍不进列表；两者发布后出现。收藏两次只保留一条用户/作品关系，再删除两次应仍无关系。对 SQLite 与 Postgres 各运行同一组断言，发现行为差异时先记录，不自动改生产业务。

## 工程维护

根 npm test 先跑 lib 的 Vitest，再跑 API 测试；Postgres 集成用例需要专用数据库。重点观察发布过滤、双存储行为、生成作业幂等和 Cookie 回归。大型 server.ts 提高导航成本，可按稳定领域边界渐进拆分，但本次不修改原项目。性能关注 LIKE 搜索、子查询计数与分页深度；没有负载实测，不提供虚构指标。

## 源码定位

- [loginUser / createSession / currentUser · apps/api/src/auth.ts:34–73](../../../evidence/story-auth/)
- [assertSameOriginRequest · apps/api/src/security.ts:51–75](../../../evidence/story-origin/)
- [validateOutboundBaseUrl / secureFetch · apps/api/src/security.ts:11–46](../../../evidence/story-outbound/)
- [security tests · apps/api/src/security.test.ts:1–31](../../../evidence/story-security-tests/)
- [addSqliteBookshelf / removeSqliteBookshelf · apps/api/src/server.ts:793–805](../../../evidence/story-shelf-sql/)
- [BookshelfEntry / ReadingProgress · prisma/schema.prisma:199–232](../../../evidence/story-schema/)

## 常见误区

三项安全单元测试通过就能宣布完整生产安全或图片生成成功。

## 自检

同源校验允许没有浏览器来源头的请求，是否能当作身份认证？

<details>
<summary>展开参考思路</summary>

不能；该检查只补浏览器跨站边界，身份仍由会话与业务权限控制。

</details>
