---
title: "story-bookstore · Bookstore · useEffect"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/components/public-library/bookstore.tsx#L26-L55)（远端行 26–55；私有仓库需要访问权限）。


## 结论

书城通过过滤和分页拉取公开书籍，清理 AbortController 并呈现加载/错误状态。

## 源码定位

- 项目：`storyforge`
- 相对路径：`components/public-library/bookstore.tsx`
- 符号：`Bookstore · useEffect`
- 行号：26–55
- 文件 SHA-256：`37b6cc81c943da714d4548b6c4fccf56b9911122d0d4fc0eaeadc26b54527c9f`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：26。

````tsx
  const [totalPages, setTotalPages] = useState(1);
  const [loading, setLoading] = useState(true);
  const [rankingLoading, setRankingLoading] = useState(true);
  const [error, setError] = useState("");

  useEffect(() => {
    const frame = requestAnimationFrame(() => setRecent(readRecentHistory().slice(0, 4)));
    return () => cancelAnimationFrame(frame);
  }, []);

  useEffect(() => {
    const controller = new AbortController();
    const params = new URLSearchParams({ genre, search, sort, page: String(page), pageSize: "12" });
    queueMicrotask(() => { if (!controller.signal.aborted) { setLoading(true); setError(""); } });
    publicApi<PagedResult<PublicBook> | PublicBook[]>(`/api/public/books?${params}`, { signal: controller.signal })
      .then((data) => {
        setBooks(unwrapItems(data));
        setTotalPages(Array.isArray(data) ? 1 : Math.max(1, Number(data.total_pages || Math.ceil(Number(data.total || 0) / Number(data.page_size || 12)) || 1)));
      })
      .catch((cause) => { if (!controller.signal.aborted) { setBooks([]); setError(cause instanceof Error ? cause.message : "故事库暂时无法访问"); } })
      .finally(() => { if (!controller.signal.aborted) setLoading(false); });
    return () => controller.abort();
  }, [genre, page, search, sort]);

  useEffect(() => {
    const controller = new AbortController();
    queueMicrotask(() => { if (!controller.signal.aborted) setRankingLoading(true); });
    publicApi<PagedResult<PublicBook> | PublicBook[]>(`/api/public/rankings?type=${rankingType}`, { signal: controller.signal })
      .then((data) => setRanking(unwrapItems(data).slice(0, 8)))
      .catch(() => { if (!controller.signal.aborted) setRanking([]); })
````
