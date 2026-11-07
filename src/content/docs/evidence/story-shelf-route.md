---
title: "story-shelf-route · bookshelfMatch"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/server.ts#L522-L539)（远端行 522–539；私有仓库需要访问权限）。


## 结论

登录用户可读取书架、加入返回 201、删除返回 204，按存储模式选择实现。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/server.ts`
- 符号：`bookshelfMatch`
- 行号：522–539
- 文件 SHA-256：`acd27a1a3dc2da74ab6d33865f41964b454445fa68eb298090adb8e2a020a225`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：522。

````ts
  if (path === '/api/account/bookshelf' && method === 'GET') {
    const actor = requireSignedIn(user);
    return sendJson(res, 200, { items: postgresPublication ? await postgresPublication.listBookshelf(actor.id) : listSqliteBookshelf(actor.id) });
  }
  const bookshelfMatch = path.match(/^\/api\/account\/bookshelf\/([^/]+)$/);
  if (bookshelfMatch && method === 'POST') {
    const actor = requireSignedIn(user);
    const entry = postgresPublication
      ? await postgresPublication.addBookshelf(actor.id, bookshelfMatch[1])
      : addSqliteBookshelf(actor.id, bookshelfMatch[1]);
    if (!entry) throw new HttpError(404, '公开作品不存在');
    return sendJson(res, 201, entry);
  }
  if (bookshelfMatch && method === 'DELETE') {
    const actor = requireSignedIn(user);
    if (postgresPublication) await postgresPublication.removeBookshelf(actor.id, bookshelfMatch[1]);
    else removeSqliteBookshelf(actor.id, bookshelfMatch[1]);
    return sendEmpty(res, 204);
````
