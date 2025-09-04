---
title: NextAuth 身份验证功能详解
published: 2025-01-04
description: ''
image: ''
tags: [next]
category: 'next'
draft: false 
lang: ''
---

# NextAuth 身份验证功能详解
### 一句话概括
**NextAuth.js 是一个专门为 Next.js 应用程序设计的、功能完整且易于集成的身份认证库。**

它帮你处理用户登录、注册、登出等一系列繁琐且容易出错的认证流程，让你能快速、安全地为你的 Next.js 应用添加认证功能。

### 它解决了什么问题？
想象一下你要为你的网站添加一个“使用 GitHub 登录”的功能，你需要：

1. 去 GitHub 注册一个 OAuth 应用，拿到 `Client ID` 和 `Client Secret`。
2. 在你的服务器上创建 API 路由来处理登录请求。
3. 将用户重定向到 GitHub 的授权页面。
4. 处理 GitHub 回调返回的授权码（code），并用它去交换访问令牌（access token）。
5. 用访问令牌获取用户在 GitHub 上的个人信息（如邮箱、用户名）。
6. 在你的数据库中查找或创建该用户。
7. 创建会话（Session）或 JWT Token 来维持用户的登录状态。
8. 安全地管理用户登出。

这个过程非常复杂，且任何一步出错都可能引入安全漏洞。

**NextAuth.js 的作用就是把这整个复杂的过程封装起来，你只需要进行简单的配置，它就能帮你自动完成所有这些步骤。**

### 简单的示例
通过固定的用户名和密码，验证下是否可以通过 `next-auth` 实现登录。

1. `npm install next-auth` 安装 next-auth，当前安装版本为 `^4.24.11`。  
2. 在 `/app/api/auth/[...nextauth]/route.ts` 下添加路由处理程序。

```typescript
import NextAuth from "next-auth";

const handler = NextAuth({
  providers: [],
};);

export { handler as GET, handler as POST };
```

2. 通过一小段代码配置 Credentials 登录，验证 nextAuth。

```typescript
import CredentialsProvider from "next-auth/providers/credentials";

const authOptions = {
  providers: [
    CredentialsProvider({
      name: "Credentials",
      // 1. 配置登录表单的字段
      credentials: {
        username: { label: "Username", type: "text", placeholder: "jsmith" },
        password: { label: "Password", type: "password" },
      },
      //   2. 自定义登录逻辑
      authorize: async (credentials) => {
        if (!credentials) {
          return null;
        }
        if (
          credentials.username !== "xiaoming" &&
          credentials.password !== "123123"
        ) {
          return null;
        }
        return {
          id: "1",
          ...credentials,
        };
      },
    }),
  ],
};
```

1. 打开 [http://localhost:3000/api/auth/signin](http://localhost:3000/api/auth/signin) 可跳转至 `next-auth`为我们提供的默认 `signin` 页面。![](./image1.png)
2. 输入用户名和密码验证是否符合登录逻辑吧。

# 通过 NextAuth 实现 gitlab 登录完整步骤指南
#### 在 GitLab 上创建应用程序
1. **登录你的 GitLab 账户**。
2. **打开 Applications** 。

![](./image2.png)

3. **新建应用，填写表单：**
    - **Name**: 给你的应用取个名字，例如 `My Next.js App`。
    - **Redirect URI**: 这是最关键的一步。填写 NextAuth.js 处理回调的地址，通常是：  
    - **http://localhost:3000/api/auth/callback/gitlab** (开发环境)
    - **https://你的生产域名/api/auth/callback/gitlab** (生产环境)
    - **Scopes**: 勾选所需的权限。对于基本的登录功能，通常只需要 `read_user` 和 `openid`。如果你需要用户的邮箱，请确保也勾选 `email`。
4. 点击 **Save application**。
5. 保存成功后，页面会显示你的 **Application ID** 和 **Secret**。**将它们复制下来**！！，配置到文件中。

#### 在 Next.js 项目中配置 NextAuth.js
1. **配置 GitLab Provider**  
将以下代码复制到 `[...nextauth]/routes.js` 文件中，并填入你的 GitLab Application ID 和 Secret。

```javascript
const handler = NextAuth({
  providers: [
    // 1. 配置 GitLab 作为认证提供程序
    GitlabProvider({
      clientId:
        "dc8474b6ce1e190171b19a46aad49d0a8cdb13642a25e70f521feab2197cf9fa",
      clientSecret:
        "gloas-dbeca1d5a90804b7aafe7aef2321b03e39fc79614c385714d17d177b9e3b982f",
    }),
  ],
  // 2. 配置 NextAuth 主题
  theme: {
    colorScheme: "dark",
    brandColor: "#FC6D26",
  },
});

```

2. 打开 [http://localhost:3000/api/auth/signin](http://localhost:3000/api/auth/signin)，可看到界面已变成 gitlab 登录按钮。

![](./image3.png)

#### 使用 `useSession()` 检查是否登陆成功
1. 文档 [https://next-auth.js.org/getting-started/client#usesession](https://next-auth.js.org/getting-started/client#usesession)
2. 因为 useSession 只能用在客户端组件中，创建 `userInfo.tsx`客户端组件使用 useSession。

```javascript
"use client";
import { useSession, SessionProvider } from "next-auth/react";

export function Component() {
  const { data: session, status } = useSession();
  console.log("%c Line:6 🍅 session", "color:#42b983", session);
  return (
    <div className="h-screen flex items-center justify-center">
      <div>Signed in as {session.user.name}</div>
    </div>
  );
}

export default function SessionProviderUserInfo() {
  return (
    // useSession 需包裹在 SessionProvider 内使用
    <SessionProvider>
      <Component />
    </SessionProvider>
  );
}
```

3. 在 page.ts 中使用 `<UserInfo>`验证用户信息。

```javascript
// pages/index.js
import { useSession, signIn, signOut } from "next-auth/react";

export default function HomePage() {
  // useSession Hook 获取会话数据和状态
  const { data: session, status } = useSession();
  const loading = status === "loading";

  if (loading) {
    return <p>Loading...</p>;
  }

  return (
    <div>
      {session ? (
        <>
          {/* 已登录状态 */}
          <p>欢迎，{session.user.name}！</p>
          <p>你的邮箱：{session.user.email}</p>
          <button onClick={() => signOut()}>退出登录</button>
        </>
      ) : (
        <>
          {/* 未登录状态 */}
          <p>请先登录</p>
          {/* 调用 signIn('gitlab') 会重定向到 GitLab 登录页面 */}
          <button onClick={() => signIn("gitlab")}>
            使用 GitLab 登录
          </button>
        </>
      )}
    </div>
  );
}
```

#### 通过 adapter 实现自动存储功能
1. 安装 `drizzle-adapter`。

```typescript
pnpm install @auth/drizzle-adapter
```

2. 更新 schema

```typescript
import {
  boolean,
  timestamp,
  pgTable,
  text,
  primaryKey,
  integer,
} from "drizzle-orm/pg-core";

export const users = pgTable("user", {
  id: text("id")
    .primaryKey()
    .$defaultFn(() => crypto.randomUUID()),
  name: text("name"),
  email: text("email").unique(),
  emailVerified: timestamp("emailVerified", { mode: "date" }),
  image: text("image"),
});

export const accounts = pgTable(
  "account",
  {
    userId: text("userId")
      .notNull()
      .references(() => users.id, { onDelete: "cascade" }),
    type: text("type").notNull(),
    provider: text("provider").notNull(),
    providerAccountId: text("providerAccountId").notNull(),
    refresh_token: text("refresh_token"),
    access_token: text("access_token"),
    expires_at: integer("expires_at"),
    token_type: text("token_type"),
    scope: text("scope"),
    id_token: text("id_token"),
    session_state: text("session_state"),
  },
  (account) => [
    {
      compoundKey: primaryKey({
        columns: [account.provider, account.providerAccountId],
      }),
    },
  ]
);

export const sessions = pgTable("session", {
  sessionToken: text("sessionToken").primaryKey(),
  userId: text("userId")
    .notNull()
    .references(() => users.id, { onDelete: "cascade" }),
  expires: timestamp("expires", { mode: "date" }).notNull(),
});

export const verificationTokens = pgTable(
  "verificationToken",
  {
    identifier: text("identifier").notNull(),
    token: text("token").notNull(),
    expires: timestamp("expires", { mode: "date" }).notNull(),
  },
  (verificationToken) => [
    {
      compositePk: primaryKey({
        columns: [verificationToken.identifier, verificationToken.token],
      }),
    },
  ]
);

export const authenticators = pgTable(
  "authenticator",
  {
    credentialID: text("credentialID").notNull().unique(),
    userId: text("userId")
      .notNull()
      .references(() => users.id, { onDelete: "cascade" }),
    providerAccountId: text("providerAccountId").notNull(),
    credentialPublicKey: text("credentialPublicKey").notNull(),
    counter: integer("counter").notNull(),
    credentialDeviceType: text("credentialDeviceType").notNull(),
    credentialBackedUp: boolean("credentialBackedUp").notNull(),
    transports: text("transports"),
  },
  (authenticator) => [
    {
      compositePK: primaryKey({
        columns: [authenticator.userId, authenticator.credentialID],
      }),
    },
  ]
);

```

3. 运行 `npx drizzle-kit push`创建数据库。
4. 运行 `npx drizzle-kit studio`打开GUI界面，可看到数据库创建成功。
5. 但是数据库此时无数据，需要我们给 `nextauth` 集成 `adpter`。打开 `src\app\api\auth\[...nextauth]\route.ts`添加以下代码。

```typescript
...
import { db } from "@/server/db/db";
const handler = NextAuth({
  adapter: DrizzleAdapter(db),
  ...
});
```

6. 打开 [http://localhost:3000/api/auth/signin](http://localhost:3000/api/auth/signin) 重新登陆后，查看 [https://local.drizzle.studio/](https://local.drizzle.studio/)，可看到在表中已经有数据生成了~

![](./image4.png)

# 通过 getServerSession 在服务端获取 Session
前面我们使用的 `useSession`是在客户端获取 session，接下来演示如何通过 `getServerSession`在服务端获取 session。

文档：[https://next-auth.js.org/configuration/nextjs#in-app-router](https://next-auth.js.org/configuration/nextjs#in-app-router)

创建 `src\server\auth\index.ts`文件，放置 `authOptions` 及 `getServerSession`。

```typescript
import GitlabProvider from "next-auth/providers/gitlab";
import { DrizzleAdapter } from "@auth/drizzle-adapter";
import { db } from "@/server/db/db";
import { getServerSession as nextAuthGetServerSession } from "next-auth";
import type { NextAuthOptions } from "next-auth";

export const authOptions: NextAuthOptions = {
  adapter: DrizzleAdapter(db),
  providers: [
    GitlabProvider({
      clientId:
        "dc8474b6ce1e190171b19a46aad49d0a8cdb13642a25e70f521feab2197cf9fa",
      clientSecret:
        "gloas-dbeca1d5a90804b7aafe7aef2321b03e39fc79614c385714d17d177b9e3b982f",
    }),
  ],
  // 2. 配置 NextAuth 主题
  theme: {
    colorScheme: "dark",
    brandColor: "#FC6D26",
  },
};

export const getServerSession = function () {
  return nextAuthGetServerSession(authOptions);
};
```

修改 `src\app\api\auth\[...nextauth]\route.ts` 文件。

```typescript
import NextAuth from "next-auth";
import { authOptions } from "@/server/auth";

const handler = NextAuth(authOptions);

export { handler as GET, handler as POST };

```

在 page 中使用 `gerServerSession`。

```tsx
import { getServerSession } from "@/server/auth";
import { redirect } from "next/navigation";

...
  const session = await getServerSession();
  console.log("%c Line:26 🍇 session", "color:#ffdd4d", session);

  if (!session?.user) {
    redirect("/api/auth/signin");
  }
...
```

打开页面，如果自动登录成功可以清除浏览器缓存进行验证，因为前文我们登录 gitlab 后储存了 session 信息在postgres 内，所以不要忘了打开docker postgres镜像哦~

清除缓存后，可以看到此时无登录信息，跳转到 signin 页面。

![](./image5.png)

在 signin 页面登陆成功后，服务端输出 session 信息。

![](./image6.png)
