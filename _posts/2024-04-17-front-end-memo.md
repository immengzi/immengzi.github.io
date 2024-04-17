---
title: 前端开发备忘录
date: 2024-04-17 14:26:00 +0800
categories: [前端]
tags: [Vue,React,HTTP,CORS,Cookie,Session]
---

## 引言

> 自己总结，试图讲明白，会反过来迫使笔者理解透彻。

最近一个月的求职经历，使我对前端的了解程度逐渐加深。从仅仅为能用框架做个小网站，到现在能串联起来一些语言、平台环境、网络、安全等不同领域的知识。我考虑将它们总结起来，方便自己回顾，也可以给同样对前端开发有兴趣的人参考。

## Javascript

### 作用域

1. var 会引起变量提升

    ```js
    function test() {
        console.log(x) // undefined
        var x = 2
        console.log(x) // 2
    }
    
    test()
    ```

2. 函数声明也可提前

    ```js
    function f() {
      x();
    
      function x() {
        console.log(1);
      }
    }
    
    f(); // 1
    ```

3. 如果将函数赋值给一个变量，则会被视为普通变量，被提升后的值为"undefined"，被调用后引发 TypeError
4. 函数声明的优先级高于变量声明，可以 F12 自己试一下
5. let 也有变量提升，但是"暂时性死区"，const 同理。下面的例子中，如果认为 let 无变量提升，应输出"1"

    ```js
    var x = 1;
    if(true) {
      console.log(x); // Uncaught ReferenceError: Cannot access 'x' before initialization
      let x = 2;
    }
    ```

6. var 没有块级作用域，循环语句可以使用 let/const 避免异步操作的参数错误

7. 作用域链

### this 指向

[彻底搞懂JavaScript中的this指向问题](https://zhuanlan.zhihu.com/p/42145138)，这篇文章确实讲的很好，我就不赘述了。

> 想要理解this，你可以先记住以下两点：
>
> **1：this永远指向一个对象；**
>
> **2：this的指向完全取决于函数调用的位置；**
>
> 针对以上的第一点特别好理解，不管在什么地方使用this，它必然会指向某个对象；确定了第一点后，也引出了一个问题，就是this使用的地方到底在哪里，而第二点就解释了这个问题，但关键是在JavaScript语言之中，一切皆对象，运行环境也是对象，所以函数都是在某个对象下运行，而this就是函数运行时所在的对象（环境）。这本来并不会让我们糊涂，但是JavaScript支持运行环境动态切换，也就是说，this的指向是动态的，很难事先确定到底指向哪个对象，这才是最让我们感到困惑的地方。
> ...
> 因为函数在js中既可以当做值传递和返回，也可当做对象和构造函数，所有函数在运行时需要确定其当前的运行环境，this就出生了，所以，this会根据运行环境的改变而改变，同时，函数中的this也只能在运行时才能最终确定运行环境；

### 继承与原型链

当谈到继承时，JavaScript 只有一种结构：对象。每个对象（object）都有一个私有属性指向另一个名为**原型**（prototype）的对象。原型对象也有一个自己的原型，层层向上直到一个对象的原型为 `null`。根据定义，`null` 没有原型，并作为这个**原型链**（prototype chain）中的最后一个环节。

### 判断数据类型的方法

1. `typeof(obj)`

   这种方法并不是万能的，它不能判断出 null 和数组。判断 null 可以使用 `if (obj === null)`，判断数组可以使用 `Array.isArray(obj)`

2. `instanceof`

3. `constructor`

4. `Object.prototype.toString`

### 闭包

> 闭包是一个函数以及其捆绑的周边环境状态（词法环境）的引用的组合。

闭包的作用：

1. 模拟私有方法
2. 延长变量的生命周期

### 回调/异步的实现

#### 回调函数

> 被作为实参传入另一函数，并在该外部函数内被调用，用以来完成某些任务的函数，称为回调函数。

```js
function loadScript(src, callback) {
  let script = document.createElement('script');
  script.src = src;
  script.onload = () => callback(script);
  document.head.append(script);
}

loadScript('https://cdnjs.cloudflare.com/ajax/libs/lodash.js/3.2.0/lodash.js', script => {
  alert(`酷，脚本 ${script.src} 加载完成`);
  alert( _ ); // _ 是所加载的脚本中声明的一个函数
});
```

#### Promise

```js
function loadScript(src) {
  return new Promise(function(resolve, reject) {
    // 当 promise 被构造完成时，自动执行此函数
    let script = document.createElement('script');
    script.src = src;

    script.onload = () => resolve(script);
    script.onerror = () => reject(new Error(`Script load error for ${src}`));

    document.head.append(script);
  });
}

let promise = loadScript("https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.11/lodash.js");

promise.then(
  script => alert(`${script.src} is loaded!`),
  error => alert(`Error: ${error.message}`)
);

promise.then(script => alert('Another handler...'));
// 实际上我们极少遇到一个 promise 需要多个处理程序的情况。使用链式调用的频率更高
```

```js
// Promise 链
loadScript("/article/promise-chaining/one.js")
	.then(script => loadScript("/article/promise-chaining/two.js"))
	.then(script => loadScript("/article/promise-chaining/three.js"))
	.then(script => {
    	one()
    	two()
    	three()
})
```

#### Async/Await

> async/await 是以更舒适的方式使用 promise 的一种特殊语法，同时它也非常易于理解和使用。

```js
async function f() {

  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });

  let result = await promise; // 等待，直到 promise resolve (*)

  alert(result); // "done!"
}

f();
```

```js
async function showAvatar() {

  // 读取我们的 JSON
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();

  // 读取 github 用户信息
  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  // 显示头像
  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  img.className = "promise-avatar-example";
  document.body.append(img);

  // 等待 3 秒
  await new Promise((resolve, reject) => setTimeout(resolve, 3000));

  img.remove();

  return githubUser;
}

showAvatar();
```

### 事件循环机制

常见的宏任务：

- setTimeout/setInterval
- I/O
- UI 渲染

常见的微任务：

- .then()/.catch()/.finally()
- await
- MutaionObserver
- process.nextTick

#### 经典题目1

```js
console.log('script start')

setTimeout(function () {
  console.log('setTimeout')
}, 0)

Promise.resolve()
  .then(function () {
    console.log('promise1')
  })
  .then(function () {
    console.log('promise2')
  })

console.log('script end')
```

1. 整体 script 作为第一个宏任务进入主线程，输出`script start`
2. 遇到 setTimeout，setTimeout 为宏任务，加入宏任务队列
3. 遇到 Promise，其 then 回调函数加入到微任务队列；第二个 then 回调函数也加入到微任务队列
4. 继续往下执行，输出`script end`
5. 检测微任务队列，输出`promise1`、`promise2`
6. 进入下一轮循环，执行 setTimeout 中的代码，输出`setTimeout`

最后执行结果为：

```
script start
script end
promise1
promise2
setTimeout
```

#### 经典题目2

```js
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2')
}
console.log('script start')
setTimeout(function () {
  console.log('setTimeout')
}, 0)
async1()
new Promise(function (resolve) {
  console.log('promise1')
  resolve()
}).then(function () {
  console.log('promise2')
})
console.log('script end')
```

```
script start
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout
```

## 浏览器

### 开发者工具

### 同源策略

同源策略（Same-Origin Policy，SOP）主要是为了保护客户端用户的安全。它是一种重要的安全策略，用于Web浏览器中，防止不同源的文档或脚本执行这些文档中的操作。这个策略确保了一个源的脚本在没有明确授权的情况下，无法读取或者操作另一个源的数据。

> 如果两个 URL 的[协议](https://developer.mozilla.org/zh-CN/docs/Glossary/Protocol)、[端口](https://developer.mozilla.org/zh-CN/docs/Glossary/Port)（如果有指定的话）和[主机](https://developer.mozilla.org/zh-CN/docs/Glossary/Host)都相同的话，则这两个 URL 是*同源*的。

**服务端也可以从中受益，比如减少因客户端安全问题导致的服务端资源滥用，但同源策略的直接和主要目的是保护客户端用户。**

情境：在线银行服务和恶意网站

假设您登录了您的在线银行账户，该网站位于 `https://www.mybank.com`。同时，在另一个浏览器标签中，您不小心访问了一个恶意网站 `https://www.badwebsite.com`。这个恶意网站试图通过JavaScript代码获取您在银行网站上的敏感信息，比如账户余额或个人信息。

恶意尝试：

1. 恶意读取尝试：
   - 恶意网站上的脚本尝试通过发送一个XHR（XMLHttpRequest）请求到 `https://www.mybank.com` 来获取您的银行账户信息。
   - JavaScript代码尝试：`fetch('https://www.mybank.com/api/account').then(response => response.json()).then(data => console.log(data));`
2. 恶意操作尝试：
   - 恶意脚本尝试提交一个表单到银行网站，以尝试进行未授权的资金转账。
   - JavaScript代码尝试：`document.createElement('form').submit.call(document.getElementById('transferForm'));`

同源策略的保护机制：

- 限制读取：当 `badwebsite.com` 的脚本尝试从 `mybank.com` 读取信息时，浏览器会阻止这种跨源HTTP请求，因为源不匹配（即协议、域名和端口号三者之一或全部不一致）。浏览器不会执行这些请求，也不会把任何响应数据透露给恶意脚本。
- 限制DOM访问：即使恶意网站的脚本可以在用户的浏览器上加载并运行，同源策略也会阻止它访问或修改 `mybank.com` 打开的网页的DOM（文档对象模型）。这意味着恶意脚本无法通过任何方式操作银行网站的表单或读取其数据。

### 解决跨域问题

#### 跨源资源共享（CORS）

> **跨源资源共享**（[CORS](https://developer.mozilla.org/zh-CN/docs/Glossary/CORS)，或通俗地译为跨域资源共享）是一种基于 [HTTP](https://developer.mozilla.org/zh-CN/docs/Glossary/HTTP) 头的机制，该机制通过允许服务器标示除了它自己以外的其他[源](https://developer.mozilla.org/zh-CN/docs/Glossary/Origin)（域、协议或端口），使得浏览器允许这些源访问加载自己的资源。跨源资源共享还通过一种机制来检查服务器是否会允许要发送的真实请求，该机制通过浏览器发起一个到服务器托管的跨源资源的“预检”请求。在预检中，浏览器发送的头中标示有 HTTP 方法和真实请求中会用到的头。
>
> 跨源 HTTP 请求的一个例子：运行在 `https://domain-a.com` 的 JavaScript 代码使用 [`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest) 来发起一个到 `https://domain-b.com/data.json` 的请求。
>
> 出于安全性，浏览器限制脚本内发起的跨源 HTTP 请求。例如，`XMLHttpRequest` 和 [Fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API) 遵循[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)。这意味着使用这些 API 的 Web 应用程序只能从加载应用程序的同一个域请求 HTTP 资源，除非响应报文包含了正确 CORS 响应头。

```
header('Access-Control-Allow-Origin:*');//允许所有来源访问
header('Access-Control-Allow-Method:POST,GET');//允许访问的方式
```

#### 反向代理

在浏览器和服务器之间加一个代理服务器，转发请求和响应

#### 发送JSONP请求替代XHR请求

JSONP 利用 Script 标签请求资源可以跨域的特点解决跨域问题

只支持 get 请求，不支持 post 请求，导致数据不安全，不推荐使用

#### 客户端浏览器解除跨域限制

设置 chrome.exe --disable-web-security

简单粗暴，不安全，不推荐

### Cookie

HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据。浏览器会存储 cookie 并在下次向同一服务器再发起请求时携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器——如保持用户的登录状态。Cookie 使基于[无状态](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview#http_是无状态，有会话的)的 HTTP 协议记录稳定的状态信息成为了可能。

Cookie 主要用于以下三个方面：

- [会话状态管理](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#会话状态管理)

  如用户登录状态、购物车、游戏分数或其他需要记录的信息

- [个性化设置](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#个性化设置)

  如用户自定义设置、主题和其他设置

- [浏览器行为跟踪](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies#浏览器行为跟踪)

  如跟踪分析用户行为等

服务器收到 HTTP 请求后，服务器可以在响应标头里面添加一个或多个 [`Set-Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie) 选项。浏览器收到响应后通常会保存下 Cookie，并将其放在 HTTP [`Cookie`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cookie) 标头内，向同一服务器发出请求时一起发送。你可以指定一个过期日期或者时间段之后，不能发送 cookie。你也可以对指定的域和路径设置额外的限制，以限制 cookie 发送的位置。

```
Set-Cookie: <name>=<value>[; <Max-Age>=<age>]
`[; expires=<date>][; domain=<domain_name>]
[; path=<some_path>][; secure][; HttpOnly]
```

**Secure**

标记为 `Secure` 的 Cookie 只应通过被 HTTPS 协议加密过的请求发送给服务端。它永远不会使用不安全的 HTTP 发送（本地主机除外），这意味着[中间人](https://developer.mozilla.org/zh-CN/docs/Glossary/MitM)攻击者无法轻松访问它。不安全的站点（在 URL 中带有 `http:`）无法使用 `Secure` 属性设置 cookie。但是，`Secure` 不会阻止对 cookie 中敏感信息的访问。例如，有权访问客户端硬盘（或，如果未设置 `HttpOnly` 属性，则为 JavaScript）的人可以读取和修改它。

**HTTPOnly**

如果HTTP响应头中包含了HttpOnly标志（可选的），那么无法通过客户端脚本 [`Document.cookie`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie) 访问该cookie（前提是浏览器支持这个标志）。此类 Cookie 仅作用于服务器。例如，持久化服务器端会话的 Cookie 不需要对 JavaScript 可用，而应具有 `HttpOnly` 属性。因此，即使存在跨站脚本攻击（XSS）漏洞，并且用户不小心访问了利用这一漏洞的链接，浏览器（主要是Internet Explorer）也不会向第三方泄露cookie。

### sessionStorage

- 页面会话在浏览器打开期间一直保持，并且重新加载或恢复页面仍会保持原来的页面会话。
- **在新标签或窗口打开一个页面时会复制顶级浏览会话的上下文作为新会话的上下文，这点和 session cookie 的运行方式不同。**
- 打开多个相同的 URL 的 Tabs 页面，会创建各自的 `sessionStorage`。
- 关闭对应浏览器标签或窗口，会清除对应的 `sessionStorage`。

### localStorage

只读的`localStorage` 属性允许你访问一个[`Document`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document) 源（origin）的对象 [`Storage`](https://developer.mozilla.org/zh-CN/docs/Web/API/Storage)；存储的数据将保存在浏览器会话中。

## 网络

### HTTP/HTTPS

加了一层 SSL/TLS 安全协议

### 从 URL 到渲染好的页面

1. URL 解析得到主机名
2. 缓存判断
3. DNS 解析
4. 建立 TCP 连接
5. 发送 HTTP 请求
6. 响应请求
7. 断开 TCP 连接
8. 浏览器解析页面
   1. 将 HTML 解析生成 DOM 树
   2. 将 CSS 解析生成 CSS 规则（rules）树
   3. 结合 DOM 树和 CSS 规则树生成渲染（render）树
   4. 根据渲染树计算每个节点的信息（布局）
   5. 根据计算后的信息绘制页面

### JavaScript 在浏览器中处理的主要流程

1. 解析 HTML： 当浏览器解析 HTML 时，遇到 `<script>` 标签会根据标签的属性不同有不同的处理方式。如果 `<script>` 标签有 `async` 或 `defer` 属性，浏览器会修改脚本的加载和执行方式。
2. 脚本加载：
   - 同步脚本： 默认情况下，浏览器会阻塞 HTML 的解析来加载和执行脚本，直到脚本执行完毕后继续解析 HTML。
   - 异步脚本（`async`）： 脚本会在被下载的过程中继续解析 HTML，但一旦脚本下载完成，浏览器会暂停 HTML 解析，立即执行 JavaScript，执行完毕后继续 HTML 解析。
   - 延迟脚本（`defer`）： 脚本会在 HTML 全部解析完毕后、触发 `DOMContentLoaded` 事件之前执行。
3. 执行 JavaScript： 在 JavaScript 执行阶段，脚本会通过修改 DOM、注册事件、调用 Web API 等操作与网页交互。这可能会改变 DOM 结构，触发重渲染。
4. DOM 更新： 如果 JavaScript 修改了 DOM 或者 CSSOM，浏览器会重新计算渲染树，更新布局（Layout），并重绘受影响的部分。
5. 事件处理： JavaScript 还可以注册事件处理函数，响应用户的交互，如点击、滚动等。这些事件可以异步触发，并可能导致 DOM 的修改、页面的重新渲染。
6. 异步操作： JavaScript 通过异步回调和 Promise 可以在不阻塞页面渲染的情况下执行长时间运行的任务，例如从服务器获取数据。

### TCP

1. TCP 三次握手

   确认双方的接收能力和发送能力是否正常、指定自己的初始化序列号为后面的可靠性传送做准备。具体一些：

   - 避免历史连接
   - 同步双方初始序列号
   - 避免资源浪费

2. TCP 四次挥手

   - 关闭连接时，客户端向服务端发送 `FIN` 时，仅仅表示客户端不再发送数据了但是还能接收数据。
   - 服务端收到客户端的 `FIN` 报文时，先回一个 `ACK` 应答报文，而服务端可能还有数据需要处理和发送，等服务端不再发送数据时，才发送 `FIN` 报文给客户端来表示同意现在关闭连接。

### HTTP 缓存机制

之前按照个别面经网站去理解，可能因为被压缩了某些助于理解的东西，导致面试时容易被问住。翻来覆去，还是 MDN 文档讲得好。

[HTTP 缓存 - MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Caching)

## 参考文章

1. [JS作用域和变量提升看这一篇就够了](https://juejin.cn/post/6844904161855684616)
1. [彻底搞懂JavaScript中的this指向问题](https://zhuanlan.zhihu.com/p/42145138)
1. [深入理解 JavaScript 之事件循环(Event Loop)](https://github.com/Jacky-Summer/personal-blog/blob/master/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3JavaScript%E7%B3%BB%E5%88%97/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%20JavaScript%20%E4%B9%8B%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF(Event%20Loop).md)