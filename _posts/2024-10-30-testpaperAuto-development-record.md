---
title: TestpaperAuto 开发实录
date: 2024-10-30 14:52:57 +0800
categories: [Nextjs]
tags: [JavaScript, JS, theme, jwt, access-token, refresh-token, Nextjs, React, Zustand, Context, Zod, Jose, Custom Hook, MongoDB]
---

## 项目背景

针对期末复习阶段往年试卷答案缺失、OCR 识别质量差的痛点，开发一站式试卷识别与答案生成解决方案

## 核心功能

- [ ] 整合高精度 OCR 与 GPT 模型，实现试卷的智能识别与答案生成
- [ ] 基于 Token 的计费系统，支持用户充值和额度管理
- [x] 完整的用户系统，包含注册、登录、找回密码等功能
- [ ] 个人中心展示识别历史记录，支持答案的再次查看

## 目录结构

```
├─public
│  └─icon
│          square-pen-light.ico
│          square-pen.ico
│
└─src
    ├─app
    │  │  init.ts
    │  │  layout.tsx
    │  │  page.tsx
    │  │
    │  ├─(routes)
    │  │  │  layout.tsx
    │  │  │
    │  │  ├─(auth)
    │  │  │  │  layout.tsx
    │  │  │  │
    │  │  │  ├─forgetpwd
    │  │  │  │      page.tsx
    │  │  │  │
    │  │  │  ├─login
    │  │  │  │      page.tsx
    │  │  │  │
    │  │  │  └─register
    │  │  │          page.tsx
    │  │  │
    │  │  ├─(protected)
    │  │  │  └─dashboard
    │  │  │      ├─history
    │  │  │      │      page.tsx
    │  │  │      │
    │  │  │      └─profile
    │  │  │              page.tsx
    │  │  │
    │  │  └─(public)
    │  │      ├─help
    │  │      │      page.tsx
    │  │      │
    │  │      └─play
    │  │              page.tsx
    │  │
    │  └─api
    │      ├─answer
    │      │      route.ts
    │      │
    │      ├─auth
    │      │  ├─login
    │      │  │      route.ts
    │      │  │
    │      │  ├─logout
    │      │  │      route.ts
    │      │  │
    │      │  ├─register
    │      │  │      route.ts
    │      │  │
    │      │  └─validate
    │      │          route.ts
    │      │
    │      └─ocr
    │              route.ts
    │
    ├─components
    │  ├─layout
    │  │      Footer.tsx
    │  │      Navbar.tsx
    │  │
    │  └─ui
    │          Alert.tsx
    │          AuthInput.tsx
    │          Spinner.tsx
    │
    ├─context
    │      ThemeContext.js
    │
    ├─hooks
    │      use-alert.ts
    │      use-auth.ts
    │
    ├─lib
    │  ├─config
    │  │      auth.ts
    │  │      routes.ts
    │  │
    │  ├─constants
    │  │      config.ts
    │  │
    │  └─types
    │          file.ts
    │          IAlert.ts
    │          index.ts
    │          IUser.ts
    │          jwt-payload.ts
    │
    ├─server
    │  ├─db
    │  │  │  index.ts
    │  │  │
    │  │  ├─config
    │  │  │      connection.ts
    │  │  │
    │  │  └─models
    │  │          fileModel.ts
    │  │          index.ts
    │  │          recordModel.ts
    │  │          userModel.ts
    │  │
    │  ├─middleware
    │  │      api-handler.ts
    │  │      jwt.ts
    │  │      validate.ts
    │  │
    │  └─repositories
    │          users-repo.ts
    │
    ├─store
    │  │  index.ts
    │  │
    │  └─slices
    │          alert-slice.ts
    │          auth-slice.ts
    │
    └─styles
            globals.css
```

## 项目架构

以 Login 为例：

<pre class="mermaid">
sequenceDiagram
    Login Page->>Custom Hook: useAuth()
    Custom Hook->>Zustand store: setLoading(true)
    Custom Hook->>API: fetch('/api/auth/login')
    API->>Repository: usersRepository.findByEmail(email)
    Repository->>MongoDB: User.findOne({email})
    MongoDB->>Repository: User
    Repository->>API: User
    API->>Custom Hook: response
    Custom Hook->>Zustand store: setUser(result.data.user)
    Zustand store->>Page: router.push(returnUrl)
    Custom Hook->>Zustand store: setLoading(false)
</pre>
<script src="
https://cdn.jsdelivr.net/npm/mermaid@11.3.0/dist/mermaid.min.js
"></script>

## 技术亮点

### 前端架构

- 采用 Next.js App Router 构建全栈应用，实现页面零配置 SSR
- 基于路由组规范实现 (auth)、(protected)、(public) 三层访问权限控制
- 使用 zustand 管理全局状态，实现主题切换、用户认证等功能
- 封装 useAlert、useAuth 等自定义 Hook，统一管理 API 

Zustand，通过 create 创建 store，类型是 state & actions，e.g. user & setUser

#### 为什么自定义 Hook？

在很多页面都可能会访问当前用户状态，在组件内部，我们的关注点是做什么，因此我们提取逻辑到自定义 Hook

#### 验证模式

```tsx
export const RegisterSchema = z.object({
    email: z.string()
        .email('Invalid email address')
        .min(1, 'Email is required'),
    username: z.string()
        .min(1, 'Username must be at least 1 characters')
        .max(50, 'Username must be less than 50 characters')
        .regex(/^[a-zA-Z0-9_-]+$/, 'Username can only contain letters, numbers, underscores and hyphens'),
    password: z.string()
        .min(6, 'Password must be at least 6 characters')
        .max(100, 'Password must be less than 100 characters')
        .regex(
            /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d\w\W]{6,}$/,
            'Password must contain at least one uppercase letter, one lowercase letter, and one number'
        )
})
```

#### 主题切换

参考：

- [Implement Dark/Light mode: How to fix the flicker of incorrect theme?](https://dev.to/tusharshahi/react-nextjs-dark-mode-theme-switcher-how-i-fixed-my-flicker-problem-5b54)
- [Lazy initial state](https://legacy.reactjs.org/docs/hooks-reference.html#lazy-initial-state)
- [Text content does not match server-rendered HTML](https://nextjs.org/docs/messages/react-hydration-error)
- [cookies](https://nextjs.org/docs/app/api-reference/functions/cookies)

问题和解决：

- 使用 `localStorage` 保存主题，会因为 `localStorage` 在服务器端不可用，导致[`hydration 错误`](https://nextjs.org/docs/messages/react-hydration-error)
  要解决此问题，需要使用 Client 端和 Server 均可访问的数据存储。比如 `cookie`

  > cookies 是一个异步函数，允许您在服务器组件中读取 HTTP 传入请求的 cookies，并在服务器操作或路由处理程序中读写出站请求的 cookies。

- 通过 `useEffect`，会因为默认主题和本地主题不同而发生闪屏
  全局入口 `layout.tsx`，获取 cookie 的值，然后再进行渲染

  ```tsx
  const cookieStore = await cookies();
  const theme = cookieStore.get('theme')?.value ?? 'dark';
  ```

### 后端设计

- 实现基于 JWT 的双 Token(access/refresh) 认证机制，提升安全性
- 使用 zod 进行请求参数校验，确保数据完整性
- 设计 API 中间件链，统一处理异常、认证和参数验证
- 采用 MongoDB 存储用户数据和识别记录，优化数据库连接复用

#### 介绍双 Token 机制

先介绍 JWT。JSON Web Token，是一个标准，将信息以 JSON 对象的形式安全传输。可以使用密钥对 JWT 进行签名

应用场景：单点登录（Single sign-on，SSO），开销小，能轻松跨域（cookie 可以跨域，sessionStorage 和 localStorage 受到[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)限制）

单点登录的优点：

- 不存储用户密码，降低访问第三方网站的风险
- 减少相同身份重复输入的时间
- 降低 IT 成本

JWT 3 个部分

header，payload，signature，分别经过 Base64Url 编码形成 JWT 对应的部分

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

```
{
  "ISSUER": "TestpaperAuto",
  "AUDIENCE": "College student",
  "exp": 2024-10-31T05:22:34.234Z
}
```

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

签名用于验证消息在此过程中未更改，并且，对于使用私钥签名的令牌，它还可以验证 JWT 的发送者是否是它所说的人

如果令牌在 `Authorization` 标头中发送，则跨域资源共享 （CORS） 不会成为问题，因为它不使用 Cookie

再拓展一下 cookie，sessionStorage 和 localStorage 的区别：

|              | cookie                                                       | sessionStorage                                               | localStorage                                                 |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 作用域       | 通过设置 Domain 和 Path                                      | 须同一窗口/标签页                                            | 同源                                                         |
| 大小         | 4KB                                                          | 5MB                                                          | 5MB                                                          |
| 服务器通信   | 发送给服务器                                                 | 仅在浏览器存储                                               | 仅在浏览器存储                                               |
| 数据有效时间 | 设置失效时间，默认 session                                   | 仅在当前的浏览器窗口关闭前有效                               | 始终有效，除非删除缓存或者手动设置为空                       |
| 设置方式     | 服务端写入                                                   | 浏览器写入                                                   | 浏览器写入                                                   |
| 安全性       | 不安全                                                       |                                                              |                                                              |
| 场景         | 需要发送到服务器的数据如 SessionID/token<br />- 广告追踪时记录用户的广告点击信息和来源渠道<br />- 新闻网站记录用户已读文章,防止重复推荐 | 临时会话数据<br />- 缓存当前会话的未发送消息草稿<br />- 多步骤表单保存用户在每个步骤中填写的数据 | 本地永久性数据<br />- 用户个性化配置，如主题颜色<br />- 文章编辑器定期自动保存用户正在编辑的文章内容 |

再聊聊跨域

同源：主机、协议、端口相同

同源策略限制了一些跨域访问：

1. Ajax 请求限制

   Async JavaScript and xml，不允许使用 XMLHttpRequest 或 Fetch API 发起跨域请求，不能读取跨域的响应数据

2. DOM 操作限制
   不能获取跨域 iframe 中的 DOM，不能操作跨域窗口的 DOM 元素，不能访问跨域窗口的 JavaScript 对象

3. 访问本地存储限制

   cookie，sessionStorage，localStorage 都不能访问

发起跨域 Ajax 请求时的具体过程：

简单请求：

```javascript
// 请求方法是以下之一:
- GET
- HEAD 
- POST

// Content-Type 是以下之一:
- text/plain
- multipart/form-data
- application/x-www-form-urlencoded
```

简单请求的处理流程：

```javascript
// 1. 发起请求
fetch('https://api.example.com/data', {
  method: 'GET'
});

// 2. 浏览器自动在请求头中添加 Origin
Origin: https://example.com

// 3. 服务器响应，需要包含:
Access-Control-Allow-Origin: https://example.com  
// 或 
Access-Control-Allow-Origin: *
```

预检请求的情况(Preflight Request) 不满足简单请求条件时(如 PUT、DELETE 方法，或包含自定义请求头)，会先发送预检请求：

```javascript
// 1. 首先发送 OPTIONS 预检请求
OPTIONS /data HTTP/1.1
Origin: https://example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: X-Custom-Header

// 2. 服务器需要返回许可:
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: PUT, POST, GET
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Max-Age: 86400  // 预检请求缓存时间

// 3. 预检通过后,才发送实际请求
PUT /data HTTP/1.1
Origin: https://example.com
X-Custom-Header: value
```

如果服务器响应未包含正确的 CORS 头：

```javascript
// 浏览器会阻止请求，控制台报错:
Access to fetch at 'https://api.example.com/data' from origin 
'https://example.com' has been blocked by CORS policy: 
No 'Access-Control-Allow-Origin' header is present on the 
requested resource.
```

携带身份凭证的请求：

```javascript
// 发起请求时设置:
fetch('https://api.example.com/data', {
  credentials: 'include'  // 携带 cookies 等凭证
});

// 服务器必须设置:
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: https://example.com  
// 注意: 这里不能用 * 通配符
```

常见的解决 CORS 的方案：

```javascript
// 1. 服务端配置 Access-Control-Allow-Origin
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type');
  next();
});

// 2. 使用代理服务器转发请求
// nginx 配置示例:
location /api {
  proxy_pass http://api.example.com;
}

// 3. JSONP(仅支持 GET 请求)
function jsonp(url, callback) {
  const script = document.createElement('script');
  script.src = `${url}?callback=${callback}`;
  document.body.appendChild(script);
}
```

双 Token 机制：

```javascript
// AccessToken - 访问令牌
{
  "type": "access",
  "userId": "12345",
  "permissions": ["read", "write"],
  "exp": 1800  // 较短的过期时间，如 30 分钟
}

// RefreshToken - 刷新令牌
{
  "type": "refresh", 
  "userId": "12345",
  "exp": 604800  // 较长的过期时间，如 7 天
}
```

登录后生成 accessToken 和 refreshToken，在响应中设置对应的 cookie。每次访问 app 会获取和验证 accessToken，并返回用户非敏感信息；
如果 accessToken 失效，则获取和验证 refreshToken，通过 cookie 设置新的 accessToken，并返回响应。登出时清除 accessToken 和 refreshToken

#### 使用 zod 进行请求参数校验

```tsx
export const RegisterSchema = z.object({
    email: z.string()
        .email('Invalid email address')
        .min(1, 'Email is required'),
    username: z.string()
        .min(1, 'Username must be at least 1 characters')
        .max(50, 'Username must be less than 50 characters')
        .regex(/^[a-zA-Z0-9_-]+$/, 'Username can only contain letters, numbers, underscores and hyphens'),
    password: z.string()
        .min(6, 'Password must be at least 6 characters')
        .max(100, 'Password must be less than 100 characters')
        .regex(
            /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d\w\W]{6,}$/,
            'Password must contain at least one uppercase letter, one lowercase letter, and one number'
        )
})
```

#### API 中间件链

统一处理异常、认证和参数验证

#### 为什么选择 MongoDB？

1. 数据结构需求
   - 数据模式经常变化，可以随时添加新字段，不需要改表结构
   - 存在非结构化或半结构化数据
   - 快速开发迭代
2. 性能需求
   - 需要高并发读写
   - 查询模式主要是文档级操作
3. 开发效率
   - MongoDB 与 JavaScript/Node.js 天然契合

#### 优化数据库连接复用

根组件加载后执行数据库初始化：

```tsx
export async function init() {
    try {
        await dbConnect();
        console.log('Database initialized successfully');
    } catch (error) {
        console.error('Failed to initialize database:', error);
        // 生产环境出错直接退出应用
        if (process.env.NODE_ENV === 'production') {
            process.exit(1);
        }
    }
}
```

```tsx
// Connection.ts
import mongoose from 'mongoose';

declare global {
    let mongoose: {
        conn: typeof mongoose | null;
        promise: Promise<typeof mongoose> | null;
    };
}

// 在全局对象上初始化mongoose属性
if (!global.mongoose) {
    global.mongoose = {
        conn: null,
        promise: null
    };
}

const MONGODB_URI = process.env.MONGODB_URI!;

const MAX_POOL_SIZE = 10;

async function dbConnect() {
    // 如果已经存在连接，直接返回
    if (global.mongoose.conn) {
        console.log('Using existing connection');
        return global.mongoose.conn;
    }

    // 如果正在建立连接，返回promise
    if (global.mongoose.promise) {
        console.log('Using existing connection promise');
        return global.mongoose.promise;
    }

    // 创建新连接
    global.mongoose.promise = mongoose.connect(MONGODB_URI, {
        maxPoolSize: MAX_POOL_SIZE,
        minPoolSize: 5,
        connectTimeoutMS: 10000,
        socketTimeoutMS: 45000,
    });

    try {
        global.mongoose.conn = await global.mongoose.promise;

        // 监听连接事件
        mongoose.connection.on('connected', () => {
            console.log('MongoDB connected');
        });

        mongoose.connection.on('error', (err) => {
            console.log('MongoDB connection error:', err);
            global.mongoose.conn = null;
            global.mongoose.promise = null;
        });

        mongoose.connection.on('disconnected', () => {
            console.log('MongoDB disconnected');
            global.mongoose.conn = null;
            global.mongoose.promise = null;
        });

        // 处理进程退出
        const cleanup = async () => {
            try {
                await mongoose.connection.close();
                global.mongoose.conn = null;
                global.mongoose.promise = null;
                process.exit(0);
            } catch (err) {
                console.error('Error during cleanup:', err);
                process.exit(1);
            }
        };

        process.on('SIGINT', cleanup);
        process.on('SIGTERM', cleanup);

        console.log('New database connection established');
        return global.mongoose.conn;

    } catch (error) {
        global.mongoose.conn = null;
        global.mongoose.promise = null;
        console.error('MongoDB connection error:', error);
        throw error;
    }
}

// 导出清理函数供外部使用
export const closeConnection = async () => {
    if (global.mongoose.conn) {
        await mongoose.connection.close();
        global.mongoose.conn = null;
        global.mongoose.promise = null;
    }
};

export default dbConnect;
```

