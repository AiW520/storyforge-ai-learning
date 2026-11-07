---
title: "story-auth · loginUser / createSession / currentUser"
---

> **版本核对：** 与远端文件一致。本页行号对应已审阅的本地快照。 引用片段与远端一致：[固定提交源码](https://github.com/AiW520/storyforge-ai/blob/f20fbf55e5f2cfb22a43996ce027dfc8035f5c6a/apps/api/src/auth.ts#L34-L73)（远端行 34–73；私有仓库需要访问权限）。


## 结论

校验密码和用户状态，会话保存摘要，Cookie HttpOnly/SameSite=Lax 且生产 Secure。

## 源码定位

- 项目：`storyforge`
- 相对路径：`apps/api/src/auth.ts`
- 符号：`loginUser / createSession / currentUser`
- 行号：34–73
- 文件 SHA-256：`86411ab4b3259efb415034b9dc78b226b8953b4ce1ff875b4ea9e720ed97390d`

工作区可能有未提交修改；以本页文件哈希为精确依据。提交版本和范围见[覆盖报告](../../coverage/)。

## 核对片段

以下是已审阅的必要片段，不包含完整文件。代码仅作为文本展示。起始行：34。

````ts
export async function loginUser(store: AuthStore, input: { email: string; password: string }) {
  const user = await store.findUserByEmail(normalizeEmail(input.email));
  const valid = user?.password_hash && await verify(String(user.password_hash), input.password).catch(() => false);
  if (!user || !valid) throw authError(401, '邮箱或密码错误');
  const status = String(user.status || (user.disabled_at ? 'disabled' : 'active'));
  if (status !== 'active') throw authError(403, status === 'banned' ? '账户已被封禁，请联系管理员' : '账户当前不可用，请联系管理员');
  if (emailVerificationRequired() && !user.email_verified_at) throw authError(403, '请先验证邮箱后再登录');
  return publicUser(user);
}

export function emailVerificationRequired() {
  return process.env.AUTH_REQUIRE_EMAIL_VERIFICATION === 'true' || (process.env.NODE_ENV === 'production' && process.env.AUTH_REQUIRE_EMAIL_VERIFICATION !== 'false');
}

export async function createSession(store: AuthStore, res: ServerResponse, userId: string) {
  const token = randomBytes(32).toString('base64url');
  const now = new Date();
  const expires = new Date(now.getTime() + SESSION_DAYS * 86_400_000);
  await store.createSession({ id: randomUUID(), user_id: userId, token_hash: tokenHash(token), expires_at: expires.toISOString(), created_at: now.toISOString() });
  setCookie(res, `${COOKIE_NAME}=${token}; Path=/; HttpOnly; SameSite=Lax; Max-Age=${SESSION_DAYS * 86_400}${process.env.NODE_ENV === 'production' ? '; Secure' : ''}`);
}

export async function logoutUser(store: AuthStore, req: IncomingMessage, res: ServerResponse) {
  const token = readCookie(req, COOKIE_NAME);
  if (token) await store.deleteSessionByHash(tokenHash(token));
  setCookie(res, `${COOKIE_NAME}=; Path=/; HttpOnly; SameSite=Lax; Max-Age=0${process.env.NODE_ENV === 'production' ? '; Secure' : ''}`);
}

export async function currentUser(store: AuthStore, req: IncomingMessage): Promise<AuthUser | null> {
  await store.deleteExpiredSessions(new Date().toISOString());
  const token = readCookie(req, COOKIE_NAME);
  if (!token) return null;
  const session = await store.findSessionByHash(tokenHash(token));
  if (!session || new Date(String(session.expires_at)).getTime() <= Date.now()) return null;
  const user = await store.findUserById(String(session.user_id));
  if (!user || String(user.status || (user.disabled_at ? 'disabled' : 'active')) !== 'active') {
    await store.deleteSessionByHash(tokenHash(token));
    return null;
  }
  return publicUser(user);
````
