---
title: 浏览器缓存机制
date: 2024-06-04 22:41:33 +0800
categories: [前端]
tags: [cache, Browser, http]
---

[HTTP缓存机制](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching) 是浏览器缓存机制的基础。

## 浏览器缓存的过程

> 强制缓存优先于协商缓存进行，若强制缓存（Expires 和 Cache-Control）生效则直接使用缓存，若不生效则进行协商缓存（Last-Modified / If-Modified-Since 和 Etag / If-None-Match），协商缓存由服务器决定是否使用缓存，若协商缓存失效，那么代表该请求的缓存失效，重新获取请求结果，再存入浏览器缓存中；生效则返回304，继续使用缓存，主要过程如下：
> 
> ![all](https://heyingye.github.io/2018/04/16/%E5%BD%BB%E5%BA%95%E7%90%86%E8%A7%A3%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9A%84%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6/img/all.jpg)

## [强制重新验证](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching#强制重新验证)

1. `no-cache`

   通过在响应中添加 `Cache-Control: no-cache` 以及 `Last-Modified` 和 `ETag`——如下所示——如果请求的资源已更新，客户端将收到 `200 OK` 响应，否则，如果请求的资源尚未更新，则会收到 `304 Not Modified` 响应。

2. `max-age=0` 和 `must-revalidate` 的组合与 `no-cache` 具有相同的含义。

   ```http
   Cache-Control: max-age=0, must-revalidate
   ```

   `max-age=0` 意味着响应立即过时，而 `must-revalidate` 意味着一旦过时就不得在没有重新验证的情况下重用它——因此，结合起来，语义似乎与 `no-cache` 相同。

## [不使用缓存](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching#不使用缓存)

`no-cache` 指令不会阻止响应的存储，而是阻止在没有重新验证的情况下重用响应。

如果你不希望将响应存储在任何缓存中，请使用 `no-store`。

```http
Cache-Control: no-store
```

不建议随意授予 `no-store`，因为你失去了 HTTP 和浏览器所拥有的许多优势，包括浏览器的后退/前进缓存。

## [常见的缓存模式](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching#常见的缓存模式)

如果服务实现了 cookie 或其他登录方式，并且内容是为每个用户个性化的，那么也必须提供 `private`，以防止与其他用户共享：

```http
Cache-Control: no-cache, private
```

## 参考

1. [彻底理解浏览器的缓存机制](https://heyingye.github.io/2018/04/16/%E5%BD%BB%E5%BA%95%E7%90%86%E8%A7%A3%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9A%84%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6/)
1. [HTTP 缓存 - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching)
