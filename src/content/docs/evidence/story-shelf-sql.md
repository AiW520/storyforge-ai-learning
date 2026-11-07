---
title: "story-shelf-sql · addSqliteBookshelf / removeSqliteBookshelf"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/server.ts#L793-L805)（远端行 793–805；私有仓库需要访问权限）。


## 结论

SQLite 分支只收藏可见已发布作品，以 user_id/work_id 插入和删除。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/server.ts`
- 符号：`addSqliteBookshelf / removeSqliteBookshelf`
- 行号：793–805
- 文件 SHA-256：`acd27a1a3dc2da74ab6d33865f41964b454445fa68eb298090adb8e2a020a225`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：793。

````ts
function addSqliteBookshelf(userId: string, rawWorkId: string) {
  const workId = boundedId(rawWorkId);
  const visible = one(`SELECT id FROM works w
    WHERE w.id=? AND w.publication_status='published' AND w.published_at IS NOT NULL
      AND EXISTS(SELECT 1 FROM chapters c WHERE c.work_id=w.id AND c.published_at IS NOT NULL)`, workId);
  if (!visible) return undefined;
  db.prepare('INSERT OR IGNORE INTO bookshelf_entries(user_id,work_id,created_at) VALUES(?,?,?)').run(userId, workId, timestamp());
  const entry = one('SELECT created_at FROM bookshelf_entries WHERE user_id=? AND work_id=?', userId, workId);
  return entry ? { work_id: workId, added_at: entry.created_at } : undefined;
}

function removeSqliteBookshelf(userId: string, rawWorkId: string) {
  db.prepare('DELETE FROM bookshelf_entries WHERE user_id=? AND work_id=?').run(userId, boundedId(rawWorkId));
````
