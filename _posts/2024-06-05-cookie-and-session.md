---
title: cookie 和 session 的工作机理
date: 2024-06-05 00:21:23 +0800
categories: [前端]
tags: [cookie, session, jwt, localStorage, sessionStorage]
---

## cookie 是什么？

> HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据。
>
> 浏览器会存储 cookie 并在下次向同一服务器再发起请求时携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器——如保持用户的登录状态。
>
> Cookie 主要用于以下三个方面：
>
> - [会话状态管理](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#会话状态管理)
>
>   如用户登录状态、购物车、游戏分数或其他需要记录的信息
>
> - [个性化设置](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#个性化设置)
>
>   如用户自定义设置、主题和其他设置
>
> - [浏览器行为跟踪](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#浏览器行为跟踪)
>
>   如跟踪分析用户行为等
>
> Cookie 曾一度用于客户端数据的存储，因当时并没有其他合适的存储办法而作为唯一的存储手段，但现在推荐使用现代存储 API。由于服务器指定 Cookie 后，浏览器的每次请求都会携带 Cookie 数据，会带来额外的性能开销（尤其是在移动环境下）。新的浏览器 API 已经允许开发者直接将数据存储到本地，如使用 [Web storage API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Storage_API)（`localStorage` 和 `sessionStorage`）或 [IndexedDB](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API) 。

## 为什么需要 cookie？

> HTTP 是无状态的：在同一个连接中，两个执行成功的请求之间是没有关系的。这就带来了一个问题，用户没有办法在同一个网站中进行连贯的交互，比如在电商网站中使用购物车功能。尽管 HTTP 根本上来说是无状态的，但借助 HTTP Cookie 就可使用有状态的会话。
>
> Cookie 使基于[无状态](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview#http_是无状态，有会话的)的 HTTP 协议记录稳定的状态信息成为了可能。

## cookie 的工作方式？

> 服务器收到 HTTP 请求后，服务器可以在响应标头里面添加一个或多个 [`Set-Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie) 选项。浏览器收到响应后通常会保存下 Cookie，并将其放在 HTTP [`Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cookie) 标头内，向同一服务器发出请求时一起发送。你可以指定一个过期日期或者时间段之后，不能发送 cookie。你也可以对指定的域和路径设置额外的限制，以限制 cookie 发送的位置。关于下面提到的头部属性的详细信息，请参考 [`Set-Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie) 文章。

### [`Set-Cookie` 和 `Cookie` 标头](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#set-cookie_和_cookie_标头)

> 这指示服务器发送标头告知客户端存储一对 cookie：
>
> ```http
> HTTP/1.0 200 OK
> Content-type: text/html
> Set-Cookie: yummy_cookie=choco
> Set-Cookie: tasty_cookie=strawberry
> 
> [页面内容]
> ```
>
> 现在，对该服务器发起的每一次新请求，浏览器都会将之前保存的 Cookie 信息通过 [`Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cookie) 请求头部再发送给服务器。
>
> ```http
> GET /sample_page.html HTTP/1.1
> Host: www.example.org
> Cookie: yummy_cookie=choco; tasty_cookie=strawberry
> ```

### [定义 Cookie 的生命周期](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#定义_cookie_的生命周期)

> Cookie 的生命周期可以通过两种方式定义：
>
> - *会话期* Cookie 会在当前的会话结束之后删除。浏览器定义了“当前会话”结束的时间，一些浏览器重启时会使用*会话恢复*。这可能导致会话 cookie 无限延长。
> - *持久性* Cookie 在过期时间（`Expires`）指定的日期或有效期（`Max-Age`）指定的一段时间后被删除。
>
> ```http
> Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
> ```
> > [!NOTE]
> >
> > 当 Cookie 的过期时间（ `Expires`）被设定时，设定的日期和时间只与客户端相关，而不是服务端。

### [限制访问 Cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#限制访问_cookie)

> 有两种方法可以确保 `Cookie` 被安全发送，并且不会被意外的参与者或脚本访问：`Secure` 属性和 `HttpOnly` 属性。
>
> 标记为 `Secure` 的 Cookie 只应通过被 HTTPS 协议加密过的请求发送给服务端。它永远不会使用不安全的 HTTP 发送（本地主机除外），这意味着[中间人](https://developer.mozilla.org/zh-CN/docs/Glossary/MitM)攻击者无法轻松访问它。不安全的站点（在 URL 中带有 `http:`）无法使用 `Secure` 属性设置 cookie。但是，`Secure` 不会阻止对 cookie 中敏感信息的访问。例如，有权访问客户端硬盘（或，如果未设置 `HttpOnly` 属性，则为 JavaScript）的人可以读取和修改它。
>
> JavaScript [`Document.cookie`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie) API 无法访问带有 `HttpOnly` 属性的 cookie；此类 Cookie 仅作用于服务器。例如，持久化服务器端会话的 Cookie 不需要对 JavaScript 可用，而应具有 `HttpOnly` 属性。此预防措施有助于缓解[跨站点脚本（XSS）](https://developer.mozilla.org/zh-CN/docs/Web/Security/Types_of_attacks#cross-site_scripting_xss)攻击。
>
> 示例：
>
> ```http
> Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
> ```

### [定义 Cookie 发送的位置](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#定义_cookie_发送的位置)

> `Domain` 和 `Path` 标识定义了 Cookie 的*作用域*：即允许 Cookie 应该发送给哪些 URL。
>
> #### Domain 属性
>
> `Domain` 指定了哪些主机可以接受 Cookie。如果不指定，该属性默认为同一 [host](https://developer.mozilla.org/zh-CN/docs/Glossary/Host) 设置 cookie，*不包含子域名*。如果指定了 `Domain`，则一般包含子域名。因此，指定 `Domain` 比省略它的限制要少。但是，当子域需要共享有关用户的信息时，这可能会有所帮助。
>
> 例如，如果设置 `Domain=mozilla.org`，则 Cookie 也包含在子域名中（如 `developer.mozilla.org`）。
>
> #### Path 属性
>
> `Path` 属性指定了一个 URL 路径，该 URL 路径必须存在于请求的 URL 中，以便发送 `Cookie` 标头。以字符 `%x2F` (“/”) 作为路径分隔符，并且子路径也会被匹配。
>
> 例如，设置 `Path=/docs`，则以下地址都会匹配：
>
> - `/docs`
> - `/docs/`
> - `/docs/Web/`
> - `/docs/Web/HTTP`
>
> 但是这些请求路径不会匹配以下地址：
>
> - `/`
> - `/docsets`
> - `/fr/docs`
>
> ---
>
> #### JavaScript 通过 Document.cookie 访问 Cookie
>
> 通过 [`Document.cookie`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie) 属性可创建新的 Cookie。如果未设置 `HttpOnly` 标记，你也可以从 JavaScript 访问现有的 Cookie。
>
> JSCopy to Clipboard
>
> ```
> document.cookie = "yummy_cookie=choco";
> document.cookie = "tasty_cookie=strawberry";
> console.log(document.cookie);
> // logs "yummy_cookie=choco; tasty_cookie=strawberry"
> ```
>
> 通过 JavaScript 创建的 Cookie 不能包含 `HttpOnly` 标志。
>
> 请留意在[安全](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#安全)章节提到的安全隐患问题，JavaScript 可以通过跨站脚本攻击（XSS）的方式来窃取 Cookie。

## 与 session 结合使用

**用户身份验证：** 当用户登录时，服务器可以创建 Session 并存储用户登录信息。服务器还可以向客户端发送 Cookie，其中包含 Session ID。当客户端再次访问服务器时，浏览器会发送 Cookie 中的 Session ID，服务器可以使用 Session ID 来检索用户登录信息并验证用户身份。

## 考虑使用 JSON Web Token

相比 cookie-session 的方式，JWT 解决了跨域认证的问题。

### JWT 组成

JWT 由三个部分组成，用点 （ `.` ） 分隔，它们是：

- **头部（Header）**：包含有关令牌本身的信息，例如使用的签名算法和令牌类型。

  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

- **负载（Payload）**：包含有关用户身份或其他声明的信息。

  ```json
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```

- **签名（Signature）**：使用头和负载中包含的信息来验证令牌的完整性和真实性。

  ```json
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret)
  ```

形如 `xxxxx.yyyyy.zzzzz`。

### JWT 工作方式

以用户注册和登录为例。

1. 用户输入用户名和密码并提交登录表单。
2. 服务器验证用户名和密码。
3. 如果验证成功，服务器将创建并签名一个 JWT。该 JWT 将包含有关用户的信息，例如用户 ID 和用户名，以及过期日期和签名密钥。
4. 服务器将 JWT 返回给客户端。
5. 客户端将 JWT 存储在其本地存储或 cookie 中。
6. 客户端在以后的请求中包含 JWT 的授权头。
7. 服务器验证 JWT 并根据其内容授予或拒绝访问。

### 对比 JWT 和 Cookie + Session

| 特性     | Cookie + Session                           | JWT                                    |
| -------- | ------------------------------------------ | -------------------------------------- |
| 存储位置 | 客户端和服务器端                           | 客户端                                 |
| 状态     | 有状态                                     | 无状态                                 |
| 跨域     | 需要额外的配置                             | 开箱即用                               |
| 安全性   | 依赖于 cookie 的安全性，可以被盗窃或篡改   | 使用签名和加密来提高安全性             |
| 可扩展性 | 服务器端需要存储会话数据，这可能会成为瓶颈 | 可扩展，因为服务器端不需要存储会话数据 |

## localStorage / sessionStorage

Web 存储对象 `localStorage` 和 `sessionStorage` 允许我们在浏览器中保存键/值对。

- `key` 和 `value` 都必须为字符串。
- 存储大小限制为 5MB+，具体取决于浏览器。
- 它们不会过期。
- 数据绑定到源（域/端口/协议）。

| `localStorage`                       | `sessionStorage`                                       |
| :----------------------------------- | :----------------------------------------------------- |
| 在同源的所有标签页和窗口之间共享数据 | 在当前浏览器标签页中可见，包括同源的 iframe            |
| 浏览器重启后数据仍然保留             | 页面刷新后数据仍然保留（但标签页关闭后数据则不再保留） |

## 参考

1. [HTTP Cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
2. [HTTP 概述](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview)
3. [Introduction to JSON Web Tokens JSON Web](https://jwt.io/introduction)
