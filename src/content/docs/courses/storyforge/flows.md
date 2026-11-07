---
title: "03 · 搜书、收藏和取消收藏"
sidebar:
  order: 3
---

> **阅读版本：** 本课程含本地未提交修改，请先查看[源码版本与远端差异](../../../source-version/)。


> **学完能做什么：** 沿真实页面函数追踪三条业务链，识别空数据和登录失效。

## 必要概念

取消请求（abort）用于避免页面卸载后的陈旧结果更新；它与服务端事务回滚不是同一件事。

## 一：查询公开书籍

读者改搜索/分类/排序/页码 → Bookstore.useEffect 构造 query、genre、sort、page、pageSize=12 → publicApi `GET /api/public/books?...`，携带 Cookie 但公开读取不要求登录 → route 同源/限流检查后匹配公开入口 → listPublicBooks 限制查询长度、排序枚举和分页 → SQLite 参数查询或 Postgres 适配 → HTTP 200 返回 items/page/page_size/total/total_pages → setBooks、setTotalPages，加载结束。

合成空响应：`{"items":[],"page":1,"page_size":12,"total":0,"total_pages":0}`。页面的页数兜底不表示后端有数据。请求失败显示错误，组件清理时 AbortController 取消请求，不能把取消提示成业务故障。

## 二：加入个人书架

点击加入 → BookDetail.toggleBookshelf，若 busy 则返回 → publicApi `POST /api/account/bookshelf/demo-book`（示例编号，无请求正文）→ 浏览器 Cookie → requireSignedIn → 按驱动 addBookshelf → SQLite boundedId、核对发布状态及章节、INSERT OR IGNORE → HTTP 201 `{ "work_id":"demo-book","added_at":"合成时间" }` → inBookshelf 变 true、notice 显示加入成功、按钮解除 busy。

未登录 401，页面提示登录作者账户；公开作品不可见返回 404，状态不切换。重复收藏在 SQLite 由关系唯一性与 INSERT OR IGNORE 处理，不生成两份书架记录。并非每个写接口都继承这种性质。

## 三：移出书架

已在书架时点击 → 同一 toggleBookshelf → `DELETE /api/account/bookshelf/demo-book` → Cookie/requireSignedIn → removeBookshelf 或 removeSqliteBookshelf，以 user_id/work_id DELETE → HTTP 204 无正文 → publicApi 的 JSON catch 得到 null，因 response.ok 不抛错 → inBookshelf 变 false，显示移除成功。

这里不能伪造 `{success:true}` 响应，也不能要求 204 一定可解析 JSON。网络错误/401 保持原状态并展示 notice；finally 解除 busy。业务失败与网络超时都不会把客户端状态提前反转。

## 示例范围

以上编号、时间和数据均为合成说明。课程不连生产 API，不展示 Cookie 内容。读者链路只用常规 HTTP；后台创作队列在下一章说明，不把它硬塞入书架流程。

## 源码定位

- [Bookstore · useEffect · components/public-library/bookstore.tsx:26–55](../../../evidence/story-bookstore/)
- [publicApi / unwrapItems · components/public-library/library-api.ts:87–99](../../../evidence/story-api/)
- [listPublicBooks · apps/api/src/server.ts:758–774](../../../evidence/story-listing/)
- [toggleBookshelf · components/public-library/book-detail.tsx:96–105](../../../evidence/story-toggle/)
- [bookshelfMatch · apps/api/src/server.ts:522–539](../../../evidence/story-shelf-route/)
- [addSqliteBookshelf / removeSqliteBookshelf · apps/api/src/server.ts:793–805](../../../evidence/story-shelf-sql/)

## 常见误区

收到 204 后还要求 response.json 必须成功，导致后端已删除但界面报错。

## 自检

收藏失败时为什么不能执行 setInBookshelf(!inBookshelf)？

<details>
<summary>展开参考思路</summary>

该状态应反映成功的服务端结果，否则页面与数据库不一致。

</details>
