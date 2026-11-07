---
title: "story-listing · listPublicBooks"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/server.ts#L758-L774)（远端行 758–774；私有仓库需要访问权限）。


## 结论

列表限制查询长度、分页和排序，SQLite 使用已发布快照字段及参数化查询。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/server.ts`
- 符号：`listPublicBooks`
- 行号：758–774
- 文件 SHA-256：`acd27a1a3dc2da74ab6d33865f41964b454445fa68eb298090adb8e2a020a225`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：758。

````ts
async function listPublicBooks(res: ServerResponse, url: URL) {
  const search = String(url.searchParams.get('query') || url.searchParams.get('search') || '').trim().slice(0, 120);
  const genre = String(url.searchParams.get('genre') || '').trim().slice(0, 80);
  const sort = enumValue(url.searchParams.get('sort'), ['hot', 'new', 'updated', 'favorite'], 'updated') as 'hot' | 'new' | 'updated' | 'favorite';
  const page = positiveQueryInt(url.searchParams.get('page'), 1, 100);
  const pageSize = positiveQueryInt(url.searchParams.get('pageSize'), 20, 50);
  res.setHeader('Cache-Control', MUTABLE_PUBLIC_CACHE_CONTROL);
  if(!postgresPublication){
    const where=["w.publication_status='published'","w.published_at IS NOT NULL","EXISTS(SELECT 1 FROM chapters c WHERE c.work_id=w.id AND c.published_at IS NOT NULL)"];const params:unknown[]=[];
    if(search){where.push('(LOWER(w.published_title) LIKE ? OR LOWER(w.published_synopsis) LIKE ?)');const value=`%${escapeLike(search.toLowerCase())}%`;params.push(value,value);}
    if(genre){where.push('LOWER(w.published_genre)=?');params.push(genre.toLowerCase());}
    const clause=where.join(' AND ');const total=Number(one(`SELECT COUNT(*) AS total FROM works w WHERE ${clause}`,...params)?.total||0);const order=sort==='new'?'w.published_at DESC':sort==='updated'?'w.last_published_at DESC':'w.featured DESC,w.last_published_at DESC';
    const items=all(`SELECT w.id,w.published_title AS title,w.published_genre AS genre,w.published_synopsis AS synopsis,w.published_cover_url AS cover_url,w.published_at,w.last_published_at AS updated_at,w.featured,(SELECT COUNT(*) FROM chapters c WHERE c.work_id=w.id AND c.published_at IS NOT NULL) AS chapter_count,(SELECT COALESCE(SUM(c.published_word_count),0) FROM chapters c WHERE c.work_id=w.id AND c.published_at IS NOT NULL) AS word_count,0 AS favorite_count FROM works w WHERE ${clause} ORDER BY ${order},w.id LIMIT ? OFFSET ?`,...params,pageSize,(page-1)*pageSize).map(item=>({...item,author:{id:'',display_name:'平台编辑部'}}));
    return sendJson(res,200,{items,page,page_size:pageSize,total,total_pages:Math.ceil(total/pageSize)});
  }
  return sendJson(res, 200, await postgresPublication.listPublicBooks({ search, genre, sort, page, pageSize }));
}
````
