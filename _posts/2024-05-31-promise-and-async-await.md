---
title: 【MDN】Promise&Async/Await
date: 2024-05-31 11:00:48 +0800
categories: [JavaScript]
tags: [Promise, Async, Await]
---

> 学习 JS 近三个月，从看不懂基本回调函数的实现，到慢慢了解 Promise 和 Async/Await，再到在项目中实际使用它们，我自认为我对异步回调函数已经"差不多了解"了。但是面试的时候被问及一些细节，我的大脑就一片空白。今日认真阅读 MDN 文档中与之相关的部分，并记录下我认为值得再回顾的内容。有感兴趣的朋友也可以收藏备忘。

## Promise 是什么？

[`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 是一个对象，它表示异步操作最终的完成（或失败）以及其结果值。

> 一个 **`Promise`** 是一个代理，它代表一个在创建 promise 时不一定已知的值。它允许你将处理程序与异步操作的最终成功值或失败原因关联起来。这使得异步方法可以像同步方法一样返回值：异步方法不会立即返回最终值，而是返回一个 *promise*，以便在将来的某个时间点提供该值。
>
> 一个 `Promise` 必然处于以下几种状态之一：
>
> - *待定（pending）*：初始状态，既没有被兑现，也没有被拒绝。
> - *已兑现（fulfilled）*：意味着操作成功完成。
> - *已拒绝（rejected）*：意味着操作失败。
> - 如果一个 Promise 已经被兑现或拒绝，即不再处于待定状态，那么则称之为已*敲定（settled）*。
>
> 你还会听到使用*已解决*（resolved）这个术语来描述 Promise——这意味着该 Promise 已经敲定（settled），或为了匹配另一个 Promise 的最终状态而被“锁定（lock-in）”，进一步解决或拒绝它都没有影响。原始 Promise 提案中的 [States and fates](https://github.com/domenic/promises-unwrapping/blob/master/docs/states-and-fates.md) 文档包含了更多关于 Promise 术语的细节。在口语中，“已解决”的 Promise 通常等价于“已兑现”的 Promise，但是正如“States and fates”所示，已解决的 Promise 也可以是待定或拒绝的。例如：
>
> ```javascript
> new Promise((resolveOuter) => {
>   resolveOuter(
>     new Promise((resolveInner) => {
>       setTimeout(resolveInner, 1000);
>     }),
>   );
> });
> ```
>
> 此 Promise 在创建时已经被解决（因为 `resolveOuter` 是同步调用的），但它是用另一个 Promise 解决的，因此在内部 Promise 兑现的 1 秒之后才会*被兑现*。在实践中，“解决”过程通常是在幕后完成的，不可观察，只有其兑现或拒绝是可观察的。

## 为什么需要 Promise？

### 异步的基本实现：回调函数

> **回调函数**是作为参数传递到另一个函数中，然后在外部函数内调用以完成某种例行程序或操作的函数。
>
> 基于回调的 API 的使用者需要编写一个被传递到 API 中的函数。API 的提供者（称为*调用方*）接受该函数，并在调用方的主体内的某个时刻回调（或者说，执行）该函数。调用方负责将正确的参数传递给回调函数。调用方也可能期望从回调函数中获得特定的返回值，用于指示调用方的进一步行为。
>
> 回调可以通过两种方式进行调用：*同步*和*异步*。同步回调在外部函数调用后立即调用，没有中间的异步任务；异步回调在某个稍后的时间点调用，通常是在一个[异步](https://developer.mozilla.org/zh-CN/docs/Glossary/Asynchronous)操作完成后。
>
> 考虑以下示例：
>
> ```javascript
> let value = 1;
> 
> doSomething(() => {
>    value = 2;
> });
> 
> console.log(value);
> 
> function doSomething(callback) {
>   callback();  // 同步调用
> }
> 
> function doSomething(callback) {
>   setTimeout(callback, 1000);  // 异步调用，延迟 1000 毫秒执行
> }
> ```
>
> 如果 `doSomething` 同步调用回调，则最后一条语句将记录 `2`，因为 `value = 2` 是同步执行的；如果回调是异步的，最后一条语句将记录 `1`，因为 `value = 2` 将在 `console.log` 语句之后执行。
>
> 同步回调的示例包括传递给 [`Array.prototype.map()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map)、[`Array.prototype.forEach()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) 等的回调。异步回调的示例包括传递给 [`setTimeout()`](https://developer.mozilla.org/zh-CN/docs/Web/API/setTimeout) 和 [`Promise.prototype.then()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 的回调。

### Promise [链式调用](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises#链式调用)

#### 是什么？

> [`Promise.prototype.then()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)、[`Promise.prototype.catch()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch) 和 [`Promise.prototype.finally()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally) 方法用于将进一步的操作与已敲定的 Promise 相关联。由于这些方法返回 Promise，因此它们可以被链式调用。

##### Promise.prototype.then()

> [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 实例的 **`then()`** 方法最多接受两个参数：用于 `Promise` 兑现和拒绝情况的回调函数。它立即返回一个等效的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象，允许你链接到其他 Promise 方法，从而实现[链式调用](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises#链式调用)。
>
> Promise.prototype.then() 立即返回一个新的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象，该对象始终处于待定状态，无论当前 Promise 对象的状态如何。
>
> `onFulfilled` 和 `onRejected` 处理函数之一将被执行，以处理当前 Promise 对象的兑现或拒绝。即使当前 Promise 对象已经敲定，这个调用也总是异步发生的。返回的 Promise 对象（称之为 `p`）的行为取决于处理函数的执行结果，遵循一组特定的规则。如果处理函数：
>
> - 返回一个值：`p` 以该返回值作为其兑现值。
> - 没有返回任何值：`p` 以 `undefined` 作为其兑现值。
> - 抛出一个错误：`p` 抛出的错误作为其拒绝值。
> - 返回一个已兑现的 Promise 对象：`p` 以该 Promise 的值作为其兑现值。
> - 返回一个已拒绝的 Promise 对象：`p` 以该 Promise 的值作为其拒绝值。
> - 返回另一个待定的 Promise 对象：`p` 保持待定状态，并在该 Promise 对象被兑现/拒绝后立即以该 Promise 的值作为其兑现/拒绝值。
>

##### Promise.prototype.catch()

> 此方法是 [`Promise.prototype.then(undefined, onRejected)`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 的一种简写形式。即`.catch()` 其实就是一个没有为 Promise 兑现时的回调函数留出空位的 `.then()`。
>
> Promise.prototype.catch() 返回一个新的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)，无论当前的 promise 状态如何，这个新的 promise 在返回时总是处于待定（pending）状态。如果调用了 `onRejected`，则返回的 promise 将根据此调用的返回值进行兑现，或者使用此调用引发的错误进行拒绝。如果当前的 promise 已兑现，则 `onRejected` 不会被调用，并且返回的 promise 具有相同的兑现值。
>
> `catch()` 方法内部会调用当前 promise 对象的 `then()` 方法，并将 `undefined` 和 `onRejected` 作为参数传递给 `then()`。该调用的返回值直接被返回。如果你对这些方法进行封装，这一点是可以观察到的。
>
> ```javascript
> // 重写原本的 Promise.prototype.then/catch 方法，只是为了添加一些日志
> ((Promise) => {
>   const originalThen = Promise.prototype.then;
>   const originalCatch = Promise.prototype.catch;
> 
>   Promise.prototype.then = function (...args) {
>     console.log("在 %o 上调用 .then 方法，参数为：%o", this, args);
>     return originalThen.apply(this, args);
>   };
>   Promise.prototype.catch = function (...args) {
>     console.error("在 %o 上调用 .catch 方法，参数为：%o", this, args);
>     return originalCatch.apply(this, args);
>   };
> })(Promise);
> 
> // 对已经解决的 Promise 调用 catch
> Promise.resolve().catch(function XXX() {});
> 
> // 输出：
> // 在 Promise{} 上调用 .catch，参数为：Arguments{1} [0: function XXX()]
> // 在 Promise{} 上调用 .then，参数为：Arguments{2} [0: undefined, 1: function XXX()]
> ```

##### Promise.prototype.finally()

> [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 实例的 **`finally()`** 方法用于注册一个在 promise 敲定（兑现或拒绝）时调用的函数。它会立即返回一个等效的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象，这可以允许你[链式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises#链式调用)调用其他 promise 方法。
>
> ```javascript
> finally(onFinally)
> ```
>
> [`onFinally`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally#onfinally)
>
> 一个当 promise 敲定时异步执行的函数。它的返回值将被忽略，除非返回一个被拒绝的 promise。调用该函数时不带任何参数。
>
> 立即返回一个新的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)。无论当前 promise 的状态如何，此新的 promise 在返回时始终处于待定（pending）状态。如果 `onFinally` 抛出错误或返回被拒绝的 promise，则新的 promise 将使用该值进行拒绝。否则，新的 promise 将以与当前 promise 相同的状态敲定（settled）。

#### 为什么？

> 连续执行两个或者多个异步操作是一个常见的需求，在上一个操作执行成功之后，开始下一个的操作，并带着上一步操作所返回的结果。在旧的回调风格中，这种操作会导致经典的[回调地狱](http://callbackhell.com/)（很难直观地理解）。
>
> ```javascript
> doSomething(function (result) {
> doSomethingElse(result, function (newResult) {
>  doThirdThing(newResult, function (finalResult) {
>    console.log(`得到最终结果：${finalResult}`);
>  }, failureCallback);
> }, failureCallback);
> }, failureCallback);
> ```
>
> 有了 Promise，我们就可以通过一个 Promise 链来解决这个问题。这就是 Promise API 的优势，因为回调函数是附加到返回的 Promise 对象上的，而不是传入一个函数中。
>
> 见证奇迹的时刻：`then()` 函数会返回一个和原来不同的**新的 Promise**：
>
> ```javascript
> const promise = doSomething();
> const promise2 = promise.then(successCallback, failureCallback);
> ```
>
> 第二个 promise（`promise2`）不仅表示 `doSomething()` 函数的完成，也代表了你传入的 `successCallback` 或者 `failureCallback` 的完成，这两个函数也可以是返回 Promise 对象的异步函数。这样的话，在 `promise2` 上新增的排在该 promise 后面的回调函数会通过 `successCallback` 或 `failureCallback` 返回。
>
> ......
>
> ```javascript
> doSomething()
> .then((result) => doSomethingElse(result))
> .then((newResult) => doThirdThing(newResult))
> .then((finalResult) => {
>  console.log(`得到最终结果：${finalResult}`);
> })
> .catch(failureCallback);
> ```
>
> `doSomethingElse` 和 `doThirdThing` 可以返回任何值——如果它们返回的是 Promise，那么会首先等待这个 Promise 的敲定，然后**下一个回调函数会接收到它的兑现值，而不是 Promise 本身**。在 `then` 回调中始终返回 Promise 是非常重要的，即使 Promise 总是兑现为 `undefined`。如果上一个处理器启动了一个 Promise 但并没有返回它，那么就没有办法再追踪它的敲定状态了，这个 Promise 就是“漂浮”的。

## async/await

`async`/`await` 基于 promise，使用 [`async`/`await`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function) 可以帮助你编写更直观、更类似同步代码的代码。下面是使用 `async`/`await` 的相同示例：

```javascript
async function logIngredients() {
  const url = await doSomething();
  const res = await fetch(url);
  const data = await res.json();
  listOfIngredients.push(data);
  console.log(listOfIngredients);
}
```

### async function

> `async function` 声明创建一个 [`AsyncFunction`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction) 对象。每次调用异步函数时，都会返回一个新的 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象，该对象将会被解决为异步函数的返回值，或者被拒绝为异步函数中未捕获的异常。
>
> 异步函数可以包含零个或者多个 [`await`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await) 表达式。await 表达式通过暂停执行使返回 promise 的函数表现得像同步函数一样，直到返回的 promise 被兑现或拒绝。返回的 promise 的解决值会被当作该 await 表达式的返回值。使用 `async`/`await` 关键字就可以使用普通的 `try`/`catch` 代码块捕获异步代码中的错误。
>
> ......
>
> 每个 await 表达式之后的代码可以被认为存在于 `.then` 回调中。通过这种方式，可以通过函数的每个可重入步骤来逐步构建 promise 链。而返回值构成了链中的最后一个环。
>
> 在接下来的示例中，我们依次 await 两个 promise，整个 `foo` 函数的执行将会被分为三个阶段。
>
> 1. `foo` 函数的第一行将会同步执行，其中 await 配置了待定的 promise。然后 `foo` 的进程将被暂停，并将控制权交还给调用 `foo` 的函数。
> 2. 一段时间后，当第一个 promise 被兑现或拒绝的时候，控制权将重新回到 `foo` 内。第一个 promise 的兑现结果（如果没有被拒绝的话）将作为 await 表达式的返回值。在这里 `1` 被赋值给 `result1`。程序继续执行，并计算第二个 await 表达式。同样的，`foo` 的进程将被暂停，并交出控制权。
> 3. 一段时间后，当第二个 promise 被兑现或拒绝的时候，控制权将重新回到 `foo`。第二个 promise 的兑现结果将作为第二个 await 表达式的返回值。在这里 `2` 被赋值给 `result2`。程序继续执行到返回表达式（如果有的话）。默认的返回值 `undefined` 将作为当前 promise 的兑现值被返回。
>
> ```javascript
> async function foo() {
>   const result1 = await new Promise((resolve) =>
>     setTimeout(() => resolve("1")),
>   );
>   const result2 = await new Promise((resolve) =>
>     setTimeout(() => resolve("2")),
>   );
> }
> foo();
> ```
>
> 注意：promise 链不是一次就构建好的，相反，promise 链是随着控制权依次在异步函数中交出并返回而分阶段构建的。因此在处理并发异步操作时，我们必须小心错误处理。

### await 和并发执行

> ```javascript
> function resolveAfter2Seconds() {
>   console.log("开始较慢兑现的 promise");
>   return new Promise((resolve) => {
>     setTimeout(() => {
>       resolve("slow");
>       console.log("较慢兑现的 promise 完成了");
>     }, 2000);
>   });
> }
> 
> function resolveAfter1Second() {
>   console.log("开始较快兑现的 promise");
>   return new Promise((resolve) => {
>     setTimeout(() => {
>       resolve("fast");
>       console.log("较快兑现的 promise 完成了");
>     }, 1000);
>   });
> }
> 
> async function sequentialStart() {
>   console.log("== sequentialStart 开始 ==");
> 
>   // 1. 启动一个计时器，并在计时器完成后打印结果
>   const slow = resolveAfter2Seconds();
>   console.log(await slow);
> 
>   // 2. 等待前一个计时器完成后，启动下一个计时器
>   const fast = resolveAfter1Second();
>   console.log(await fast);
> 
>   console.log("== sequentialStart 结束 ==");
> }
> 
> async function sequentialWait() {
>   console.log("== sequentialWait 开始 ==");
> 
>   // 1. 启动两个计时器，互不等待
>   const slow = resolveAfter2Seconds();
>   const fast = resolveAfter1Second();
> 
>   // 2. 等待较慢的计时器完成后，打印结果
>   console.log(await slow);
>   // 3. 等待较快的计时器完成后，打印结果
>   console.log(await fast);
> 
>   console.log("== sequentialWait 结束 ==");
> }
> 
> async function concurrent1() {
>   console.log("== concurrent1 开始 ==");
> 
>   // 1. 并发启动两个计时器，并等待它们完成
>   const results = await Promise.all([
>     resolveAfter2Seconds(),
>     resolveAfter1Second(),
>   ]);
>   // 2. 同时打印两个计时器的结果
>   console.log(results[0]);
>   console.log(results[1]);
> 
>   console.log("== concurrent1 完成 ==");
> }
> 
> async function concurrent2() {
>   console.log("== concurrent2 开始 ==");
> 
>   // 1. 并发启动两个计时器，并在其中任意一个完成后立即打印对应结果
>   await Promise.all([
>     (async () => console.log(await resolveAfter2Seconds()))(),
>     (async () => console.log(await resolveAfter1Second()))(),
>   ]);
>   console.log("== concurrent2 结束 ==");
> }
> 
> sequentialStart(); // 2 秒后，打印“slow”，然后再过 1 秒，打印“fast”
> 
> // 等待上面的代码执行完毕
> setTimeout(sequentialWait, 4000); // 2 秒后，打印“slow”，然后打印“fast”
> 
> // 再次等待
> setTimeout(concurrent1, 7000); // 跟 sequentialWait 一样
> 
> // 再次等待
> setTimeout(concurrent2, 10000); // 1 秒后，打印“fast”，然后过 1 秒，打印“slow”
> ```
>
> 在 `sequentialStart` 中，程序执行第一个 `await` 时暂停 2 秒，然后又为第二个 `await` 暂停了 1 秒。直到第一个计时器结束后，第二个计时器才被创建，因此程序需要 3 秒执行完毕。
>
> 在 `sequentialWait` 中，两个计时器都被创建并用 `await` 进行等待。这两个计时器并行运行，这意味着代码运行时间缩短到 2 秒，而不是 3 秒，即较慢的计时器的时间。然而，`await` 调用仍旧是顺序执行的，这意味着第二个 `await` 会等待第一个执行完。在这个例子中，较快的计时器的结果会在较慢的计时器之后被处理。
>
> 在 `concurrentStart` 中，两个计时器被同时创建，然后执行 `await`。这两个计时器同时运行，这意味着程序完成运行只需要 2 秒，而不是 3 秒，即较慢的计时器的时间。
>
> 如果你希望在并发执行的两个或多个任务完成后安全地执行其他任务，那么在这些任务开始前，你必须等待对 [`Promise.all()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) 或 [`Promise.allSettled()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled) 的调用。

### [使用异步函数重写 promise 链](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function#使用异步函数重写_promise_链)

> 返回 [`Promise`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)的 API 将会产生一个 promise 链，它将函数肢解成许多部分。例如下面的代码：
>
> ```javascript
>function getProcessedData(url) {
>     return downloadData(url) // 返回一个 promise
>        .catch((e) => downloadFallbackData(url)) // 返回一个 promise
>        .then((v) => processDataInWorker(v)); // 返回一个 promise
> }
> ```
> 
> 可以使用单个异步函数重写，如下所示：
>
> ```javascript
>async function getProcessedData(url) {
>     let v;
>     try {
>        v = await downloadData(url);
>     } catch (e) {
>        v = await downloadFallbackData(url);
>     }
>     return processDataInWorker(v);
> }
> ```
> 
> 或者，你可以使用 `catch()` 链式调用 promise：
>
> ```javascript
>async function getProcessedData(url) {
>     const v = await downloadData(url).catch((e) => downloadFallbackData(url));
>     return processDataInWorker(v);
> }
> ```
> 
> 以上两个重写版本中，请注意在 `return` 关键字之后没有 `await` 语句，这也是有效的。

`async` /异步函数总是返回一个 promise。如果一个异步函数的返回值本身不是 promise，那么它将会被隐式地包装在一个 promise 中。看起来像是被包装在了一个 [`Promise.resolve`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve) 中，但它们不是等价的。二者区别详见 [async function - JavaScript | MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function#:~:text=%E5%8D%B3%E4%BD%BF%E5%BC%82%E6%AD%A5%E5%87%BD%E6%95%B0%E7%9A%84%E8%BF%94%E5%9B%9E%E5%80%BC%E7%9C%8B%E8%B5%B7%E6%9D%A5%E5%83%8F%E6%98%AF%E8%A2%AB%E5%8C%85%E8%A3%85%E5%9C%A8%E4%BA%86%E4%B8%80%E4%B8%AA%20Promise.resolve%20%E4%B8%AD%EF%BC%8C%E4%BD%86%E5%AE%83%E4%BB%AC%E4%B8%8D%E6%98%AF%E7%AD%89%E4%BB%B7%E7%9A%84%E3%80%82)。

## 错误处理

### Promise 链

> ```javascript
> doSomething()
>   .then((result) => doSomethingElse(result))
>   .then((newResult) => doThirdThing(newResult))
>   .then((finalResult) => console.log(`得到最终结果：${finalResult}`))
>   .catch(failureCallback);
> ```
>

### async/await

> ```javascript
> async function foo() {
>   try {
>     const result = await doSomething();
>     const newResult = await doSomethingElse(result);
>     const finalResult = await doThirdThing(newResult);
>     console.log(`得到最终结果：${finalResult}`);
>   } catch (error) {
>     failureCallback(error);
>   }
> }
> ```
>

### 嵌套

> 嵌套是一种可以限制 `catch` 语句的作用域的控制结构写法。明确来说，嵌套的 `catch` 只会捕获其作用域及以下的错误，而不会捕获链中更高层的错误。如果使用正确，可以实现细粒度的错误恢复。**简洁的 Promise 链式编程最好保持扁平化，不要嵌套 Promise，因为嵌套经常会是粗心导致的。**
> ```javascript
> doSomethingCritical()
> .then((result) =>
>  doSomethingOptional()
>    .then((optionalResult) => doSomethingExtraNice(optionalResult))
>    .catch((e) => {}),
> ) // 即便可选操作失败了，也会继续执行
> .then(() => moreCriticalStuff())
> .catch((e) => console.log(`严重失败：${e.message}`));
> 
> 
> // async/await 版
> async function main() {
> try {
>  const result = await doSomethingCritical();
>  try {
>    const optionalResult = await doSomethingOptional(result);
>    await doSomethingExtraNice(optionalResult);
>  } catch (e) {
>    // 忽略可选步骤的失败并继续执行。
>  }
>  await moreCriticalStuff();
> } catch (e) {
>  console.error(`严重失败：${e.message}`);
> }
> }
> ```

### [Catch 的后续链式操作](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Using_promises#catch_的后续链式操作)

> ```javascript
> new Promise((resolve, reject) => {
>   console.log("初始化");
> 
>   resolve();
> })
>   .then(() => {
>     throw new Error("有哪里不对了");
> 
>     console.log("执行「这个」");
>   })
>   .catch(() => {
>     console.log("执行「那个」");
>   })
>   .then(() => {
>     console.log("执行「这个」，无论前面发生了什么");
>   });
> 
> 
> // async/await 版本
> async function main() {
>   try {
>     await doSomething();
>     throw new Error("有哪里不对了");
>     console.log("执行「这个」");
>   } catch (e) {
>     console.error("执行「那个」");
>   }
>   console.log("执行「这个」，无论前面发生了什么");
> }
> ```
> ```
> 初始化
> 执行「那个」
> 执行「这个」，无论前面发生了什么
> ```

## Promise 组合

以下各种 Promise 组合的静态方法均接受一个 Promise 可迭代对象作为输入，并返回一个 Promise。以 Promise.all() 为例：

```javascript
Promise.all([func1(), func2(), func3()]).then(([result1, result2, result3]) => {
/* 使用 result1、result2 和 result3 */
});
```

### Promise.all()

> 当所有输入的 Promise 都被**兑现**（fulfilled）时，返回的 Promise 也将被兑现（即使传入的是一个空的可迭代对象），并返回一个包含所有兑现值的数组。
>
> 如果数组中的某个 Promise 被拒绝，`Promise.all()` 就会立即拒绝返回的 Promise，并终止其他操作。
>
> 相比之下，Promise.allSettled() 方法返回的 Promise 会等待所有输入的 Promise 完成，不管其中是否有 Promise 被拒绝。如果你需要获取输入可迭代对象中每个 Promise 的最终结果，则应使用 allSettled() 方法。

### Promise.allSettled()

> 当所有输入的 Promise 都已**敲定**（settled）时（包括传入空的可迭代对象时），返回的 Promise 将被兑现，并带有描述每个 Promise 结果的对象数组。
>
> 等待所有操作完成后再处理返回的 Promise。

### Promise.any()

> 当输入的任何一个 Promise 兑现时，这个返回的 Promise 将会兑现，并返回第一个兑现的值。当所有输入 Promise 都被拒绝（包括传递了空的可迭代对象）时，它会以一个包含拒绝原因数组的 [`AggregateError`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/AggregateError) 拒绝。
>
> 与 [`Promise.all()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all) 返回一个兑现值*数组*不同的是，我们只会得到一个兑现值（假设至少有一个 Promise 被兑现）。
>
> 与 [`Promise.race()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race) 返回第一个*敲定*（无论是兑现还是拒绝）的值不同的是，该方法返回第一个*兑现*的值。该方法忽略所有被拒绝的 Promise，直到第一个被兑现的 Promise。

### Promise.race()

> 这个返回的 promise 会随着第一个 promise 的敲定（settled）而敲定（settled）。

## Promise 时序

> 在设计异步 API 的上下文中，这意味着回调在某些情况下是同步调用的，但在其他情况下是异步调用的，这为调用者带来的歧义。
>
> ```javascript
> let value = 1;
> 
> doSomething(() => {
> value = 2;
> });
> 
> console.log(value);
> 
> function doSomething(callback) {
> callback();  // 同步调用
> }
> 
> function doSomething(callback) {
> setTimeout(callback, 1000);  // 异步调用，延迟 1000 毫秒执行
> }
> 
> function doSomething(callback) {
> if (Math.random() > 0.5) { // Zalgo 状态: 同步异步混乱
>  callback();
> } else {
>  setTimeout(() => callback(), 1000);
> }
> }
> ```
>
> 另一方面，Promise 是一种[控制反转](https://zh.wikipedia.org/wiki/控制反转)的形式——API 的实现者不控制回调何时被调用。相反，维护回调队列并决定何时调用回调的工作被委托给了 Promise 的实现者，这样一来，API 的使用者和开发者都会自动获得强大的语义保证，包括：
>
> - 被添加到 [`then()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 的回调永远不会在 JavaScript 事件循环的[当前运行完成](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Event_loop#执行至完成)之前被调用。
>
> - 即使异步操作已经完成（成功或失败），在这之后通过 [`then()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 添加的回调函数也会被调用。
>
> - 通过多次调用 [`then()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/then) 可以添加多个回调函数，它们会按照插入顺序进行执行。
>
> 传入 `then()` 的函数不会立即运行，而是被放入微任务队列中，这意味着它会在稍后运行（仅在创建该函数的函数退出后，且 JavaScript 执行堆栈为空时），也就是在控制权返回事件循环之前。

事件循环相关：

- [并发模型与事件循环](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Event_loop)
- [深入：微任务与 Javascript 运行时环境](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API/Microtask_guide/In_depth)
