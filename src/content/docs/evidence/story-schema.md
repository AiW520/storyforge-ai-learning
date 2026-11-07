---
title: "story-schema · BookshelfEntry / ReadingProgress"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/prisma/schema.prisma#L199-L232)（远端行 199–232；私有仓库需要访问权限）。


## 结论

Prisma 声明书架复合主键、索引和阅读进度关系；SQLite 实现需另核对。

## 源码定位

- 项目：`storyforge`
- 相对路径：`prisma/schema.prisma`
- 符号：`BookshelfEntry / ReadingProgress`
- 行号：199–232
- 文件 SHA-256：`84b8fe4d94cee125ef15fdcb8f66e476d737d5672ab53272966ce26aefc3c7bf`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：199。

````prisma
model BookshelfEntry {
  userId    String   @map("user_id")
  workId    String   @map("work_id")
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  work      Work     @relation(fields: [workId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now()) @map("created_at")

  @@id([userId, workId])
  @@index([workId, createdAt(sort: Desc)])
  @@index([userId, createdAt(sort: Desc)])
  @@map("bookshelf_entries")
}

model ReadingProgress {
  userId           String    @map("user_id")
  workId           String    @map("work_id")
  chapterId        String?   @map("chapter_id")
  user             User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  work             Work      @relation(fields: [workId], references: [id], onDelete: Cascade)
  chapter          Chapter?  @relation(fields: [chapterId], references: [id], onDelete: SetNull)
  progress         Float     @default(0)
  totalReadSeconds Int       @default(0) @map("total_read_seconds")
  completed        Boolean   @default(false)
  completedAt      DateTime? @map("completed_at")
  lastEventHash    String    @default("") @map("last_event_hash") @db.VarChar(64)
  lastMetricDate   DateTime? @map("last_metric_date") @db.Date
  createdAt        DateTime  @default(now()) @map("created_at")
  updatedAt        DateTime  @updatedAt @map("updated_at")

  @@id([userId, workId])
  @@index([workId, completed])
  @@index([userId, updatedAt(sort: Desc)])
  @@index([chapterId])
  @@map("reading_progress")
````
