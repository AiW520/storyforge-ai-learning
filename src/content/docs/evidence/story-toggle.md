---
title: "story-toggle · toggleBookshelf"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/components/public-library/book-detail.tsx#L96-L105)（远端行 96–105；私有仓库需要访问权限）。


## 结论

忙碌时阻止重复点击；成功后切换状态，401 提示登录，finally 解除忙碌。

## 源码定位

- 项目：`storyforge`
- 相对路径：`components/public-library/book-detail.tsx`
- 符号：`toggleBookshelf`
- 行号：96–105
- 文件 SHA-256：`fc2e60407e3245d99af751332cd137956c7af45db5362d86a2ab6d0ec0a73382`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：96。

````tsx
  const toggleBookshelf = async () => {
    if (bookshelfBusy) return;
    setBookshelfBusy(true); setNotice("");
    try {
      if (inBookshelf) await publicApi(`/api/account/bookshelf/${encodeURIComponent(bookId)}`, { method: "DELETE" });
      else await publicApi(`/api/account/bookshelf/${encodeURIComponent(bookId)}`, { method: "POST" });
      setInBookshelf(!inBookshelf); setNotice(inBookshelf ? "已从书架移除" : "已加入我的书架");
    } catch (cause) {
      setNotice(cause instanceof PublicApiError && cause.status === 401 ? "登录作者账户后即可使用云端书架" : cause instanceof Error ? cause.message : "书架操作失败");
    } finally { setBookshelfBusy(false); }
````
