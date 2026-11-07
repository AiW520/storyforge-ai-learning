---
title: "story-queue · queueEnabled / enqueueGeneration"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/task-queue.ts#L60-L84)（远端行 60–84；私有仓库需要访问权限）。


## 结论

REDIS_URL 决定队列开关，任务使用 taskId 去重，三次尝试和指数退避。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/task-queue.ts`
- 符号：`queueEnabled / enqueueGeneration`
- 行号：60–84
- 文件 SHA-256：`d6c442358b826b6ff1431e433823ede9d3ac141a2df0e7db6c26802b46189b50`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：60。

````ts
export function queueEnabled() { return Boolean(process.env.REDIS_URL?.trim()); }

export async function queueReady() {
  if (!queueEnabled()) return true;
  getQueue();
  const client = connection;
  if (!client) return false;
  return (await client.ping()) === 'PONG';
}

export async function enqueueGeneration(job: GenerationJob) {
  const queue = getQueue();
  const existing = await queue.getJob(job.taskId);
  if (existing) {
    const state = await existing.getState();
    if (state === 'completed' || state === 'failed') await existing.remove();
    else return existing;
  }
  return queue.add('chapter.generate', job, {
    jobId: job.taskId,
    attempts: 3,
    backoff: { type: 'exponential', delay: 2_000 },
    removeOnComplete: { age: 86_400, count: 1000 },
    removeOnFail: { age: 604_800, count: 5000 },
  });
````
