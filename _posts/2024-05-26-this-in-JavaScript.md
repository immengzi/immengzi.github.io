---
title: JavaScript中的this指针
date: 2024-05-26 23:57:34 +0800
categories: [JavaScript]
tags: [JS,this]
---

> 从三月中旬开始，一边从头学 JavaScript 一边面试，已遇到了若干次的 this 指针问题，似乎我从来只是靠猜。所以我想通过写笔记的方式来彻底搞懂 JS 中的 this。

## 红宝书（JavaScript高级程序设计）的解释

### 函数内部 this

另一个特殊的对象是 this，它在标准函数和箭头函数中有不同的行为。

在标准函数中，**this 引用的是把函数当成方法调用的上下文对象**，这时候通常称其为 this 值（在网页的全局上下文中调用函数时，this 指向 windows）。来看下面的例子：

```js
window.color = 'red';
let o = {
    color: 'blue'
};

function sayColor() {
    console.log(this.color);
}

sayColor(); // 'red'

o.sayColor = sayColor;
o.sayColor(); // 'blue'
```

定义在全局上下文中的函数 sayColor() 引用了 this 对象。这个 this 到底引用哪个对象必须到函数被调用时才能确定。因此这个值在代码执行的过程中可能会变。如果在全局上下文中调用 sayColor()，这结果会输出"red"，因为 this 指向 window，而 this.color 相当于 window.color。而在把 sayColor() 赋值给 o 之后再调用 o.sayColor()，this 会指向 o，即 this.color 相当于 o.color，所以会显示"blue"。

在箭头函数中，this 引用的是定义箭头函数的上下文。下面的例子演示了这一点。在对 sayColor() 的两次调用中，this 引用的都是 window 对象，因为这个箭头函数是在 window 上下文中定义的：

```js
window.color = 'red';
let o = {
    color: 'blue'
};

let sayColor = () => console.log(this.color);

sayColor(); // 'red'

o.sayColor = sayColor;
o.sayColor(); // 'red'
```

有读者知道，在事件回调或定时回调中调用某个函数时，this 值指向的并非想要的对象。此时将回调函数写成箭头函数就可以解决问题。这是因为箭头函数中的 this 会保留定义该函数时的上下文：

```javascript
function King() {
    this.royaltyName = 'Henry';
    // this 引用 King 的实例
    setTimeout(() => console.log(this.royaltyName), 1000);
}

function Queen() {
    this.royaltyName = 'Elizabeth';
    
    // this 引用 window 对象
    setTimeout(function() { console.log(this.royaltyName); }, 1000);
}

new King(); // Henry
new Queen(); // undefined
```

> 注意：函数名只是保存指针的变量。因此全局定义的 sayColor() 函数和 o.sayColor() 是同一个函数，只不过执行的上下文不同。

### 闭包中的 this 对象

在闭包中使用 this 会让代码变复杂。如果内部函数没有使用箭头函数定义，则 this 对象会在运行时绑定到执行函数的上下文。如果在全局函数中调用，则 this 在非严格模式下等于 window，在严格模式下等于 undefined。如果作为某个对象的方法调用，则 this 等于这个对象。匿名函数在这种情况下不会绑定到某个对象，这就意味着 this 会指向 window，除非在严格模式下 this 是 undefined。不过，由于闭包的写法所致，这个事实有时候没有那么容易看出来。来看下面的例子：

```js
window.identity = 'The Window';

let object = {
    identity: 'My Object',
    getIdentityFunc() {
        return function() {
            return this.identity;
        };
    }
};

console.log(object.getIdentityFunc()()); // 'The Window'
```

这里先创建了一个全局变量 identity，之后又创建一个包含 identity 属性的对象。这个对象还包含一个 getIdentityFunc() 方法，返回一个匿名函数。这个匿名函数返回 this.identity。因为 getIdentityFunc() 返回函数，所以 object.getIdentityFunc()() 会立即调用这个返回的函数，从而得到一个字符串。可是，此时返回的字符串是"The Winodw"，即全局变量 identity 的值。为什么匿名函数没有使用其包含作用域（getIdentityFunc()）的 this 对象呢？

前面介绍过，每个函数在被调用时都会自动创建两个特殊变量：this 和 arguments。内部函数永远不可能直接访问外部函数的这两个变量。但是，如果把 this 保存到闭包可以访问的另一个变量中，则是行得通的。比如：

```js
window.identity = 'The Window';

let object = {
    identity: 'My Object',
	getIdentityFunc() {
 		let that = this; // 手动加粗
 		return function() {
 			return that.identity; // 手动加粗
        };
    }
};

console.log(object.getIdentityFunc()()); // 'My Object' 
```

这里加粗的代码展示了与前面那个例子的区别。在定义匿名函数之前，先把外部函数的 this 保存到变量 that 中。然后在定义闭包时，就可以让它访问 that，因为这是包含函数中名称没有任何冲突的一个变量。即使在外部函数返回之后，that 仍然指向 object，所以调用 object.getIdentityFunc()() 就会返回"My Object"。

> 注意：this 和 arguments 都是不能直接在内部函数中访问的。如果想访问包含作用域中的 arguments 对象，则同样需要将其引用先保存到闭包能访问的另一个变量中。

在一些特殊情况下，this 值可能并不是我们所期待的值。比如下面这个修改后的例子：

```js
window.identity = 'The Window';
let object = {
	identity: 'My Object',
	getIdentity () { // 手动加粗
		return this.identity; // 手动加粗
    }
}; 
```

getIdentity() 方法就是返回 this.identity 的值。以下是几种调用 object.getIdentity() 的方式及返回值：

```js
object.getIdentity(); // 'My Object'
(object.getIdentity)(); // 'My Object'
(object.getIdentity = object.getIdentity)(); // 'The Window'
```

第一行调用 object.getIdentity() 是正常调用，会返回"My Object"，因为 this.identity 就是 object.identity。第二行在调用时把 object.getIdentity 放在了括号里。虽然加了括号之后看起来是对一个函数的引用，但 this 值并没有变。这是因为按照规范，object.getIdentity 和 (object.getIdentity) 是相等的。第三行执行了一次赋值，然后再调用赋值后的结果。因为赋值表达式的值是函数本身，this 值不再与任何对象绑定，所以返回的是"The Window"。

一般情况下，不大可能像第二行和第三行这样调用对象上的方法。但通过这个例子，我们可以知道，即使语法稍有不同，也可能影响 this 的值。

---

## 现代 JavaScript 教程

> # 对象方法，"this"
>
> 通常创建对象来表示真实世界中的实体，如用户和订单等：
>
> ```javascript
> let user = {
>   name: "John",
>   age: 30
> };
> ```
>
> 并且，在现实世界中，用户可以进行 **操作**：从购物车中挑选某物、登录和注销等。
>
> 在 JavaScript 中，行为（action）由属性中的函数来表示。
>
> ## [方法示例](https://zh.javascript.info/object-methods#fang-fa-shi-li)
>
> 刚开始，我们来教 `user` 说 hello：
>
> ```javascript
> let user = {
>   name: "John",
>   age: 30
> };
> 
> user.sayHi = function() {
>   alert("Hello!");
> };
> 
> user.sayHi(); // Hello!
> ```
>
> 这里我们使用函数表达式创建了一个函数，并将其指定给对象的 `user.sayHi` 属性。
>
> 随后我们像这样 `user.sayHi()` 调用它。用户现在可以说话了！
>
> 作为对象属性的函数被称为 **方法**。
>
> 所以，在这我们得到了 `user` 对象的 `sayHi` 方法。
>
> 当然，我们也可以使用预先声明的函数作为方法，就像这样：
>
> ```javascript
> let user = {
>   // ...
> };
> 
> // 首先，声明函数
> function sayHi() {
>   alert("Hello!");
> }
> 
> // 然后将其作为一个方法添加
> user.sayHi = sayHi;
> 
> user.sayHi(); // Hello!
> ```
>
> **面向对象编程**
>
> 当我们在代码中用对象表示实体时，就是所谓的 [面向对象编程](https://en.wikipedia.org/wiki/Object-oriented_programming)，简称为 “OOP”。
>
> OOP 是一门大学问，本身就是一门有趣的科学。怎样选择合适的实体？如何组织它们之间的交互？这就是架构，有很多关于这方面的书，例如 E. Gamma、R. Helm、R. Johnson 和 J. Vissides 所著的《设计模式：可复用面向对象软件的基础》，G. Booch 所著的《面向对象分析与设计》等。
>
> ### [方法简写](https://zh.javascript.info/object-methods#fang-fa-jian-xie)
>
> 在对象字面量中，有一种更短的（声明）方法的语法：
>
> ```javascript
> // 这些对象作用一样
> 
> user = {
>   sayHi: function() {
>     alert("Hello");
>   }
> };
> 
> // 方法简写看起来更好，对吧？
> let user = {
>   sayHi() { // 与 "sayHi: function(){...}" 一样
>     alert("Hello");
>   }
> };
> ```
>
> 如上所示，我们可以省略 `"function"`，只写 `sayHi()`。
>
> 说实话，这种表示法还是有些不同。在对象继承方面有一些细微的差别（稍后将会介绍），但目前它们并不重要。在几乎所有的情况下，更短的语法是首选的。
>
> ## [方法中的 “this”](https://zh.javascript.info/object-methods#fang-fa-zhong-de-this)
>
> 通常，对象方法需要访问对象中存储的信息才能完成其工作。
>
> 例如，`user.sayHi()` 中的代码可能需要用到 `user` 的 name 属性。
>
> **为了访问该对象，方法中可以使用 `this` 关键字。**
>
> `this` 的值就是在点之前的这个对象，即调用该方法的对象。
>
> 举个例子：
>
> ```javascript
> let user = {
>   name: "John",
>   age: 30,
> 
>   sayHi() {
>     // "this" 指的是“当前的对象”
>     alert(this.name);
>   }
> 
> };
> 
> user.sayHi(); // John
> ```
>
> 在这里 `user.sayHi()` 执行过程中，`this` 的值是 `user`。
>
> 技术上讲，也可以在不使用 `this` 的情况下，通过外部变量名来引用它：
>
> ```javascript
> let user = {
>   name: "John",
>   age: 30,
> 
>   sayHi() {
>     alert(user.name); // "user" 替代 "this"
>   }
> 
> };
> ```
>
> ……但这样的代码是不可靠的。如果我们决定将 `user` 复制给另一个变量，例如 `admin = user`，并赋另外的值给 `user`，那么它将访问到错误的对象。
>
> 下面这个示例证实了这一点：
>
> ```javascript
> let user = {
>   name: "John",
>   age: 30,
> 
>   sayHi() {
>     alert( user.name ); // 导致错误
>   }
> 
> };
> 
> 
> let admin = user;
> user = null; // 重写让其更明显
> 
> admin.sayHi(); // TypeError: Cannot read property 'name' of null
> ```
>
> 如果我们在 `alert` 中以 `this.name` 替换 `user.name`，那么代码就会正常运行。
>
> ## [“this” 不受限制](https://zh.javascript.info/object-methods#this-bu-shou-xian-zhi)
>
> 在 JavaScript 中，`this` 关键字与其他大多数编程语言中的不同。JavaScript 中的 `this` 可以用于任何函数，即使它不是对象的方法。
>
> 下面这样的代码没有语法错误：
>
> ```javascript
> function sayHi() {
>   alert( this.name );
> }
> ```
>
> `this` 的值是在代码运行时计算出来的，它取决于代码上下文。
>
> 例如，这里相同的函数被分配给两个不同的对象，在调用中有着不同的 “this” 值：
>
> ```javascript
> let user = { name: "John" };
> let admin = { name: "Admin" };
> 
> function sayHi() {
>   alert( this.name );
> }
> 
> // 在两个对象中使用相同的函数
> user.f = sayHi;
> admin.f = sayHi;
> 
> // 这两个调用有不同的 this 值
> // 函数内部的 "this" 是“点符号前面”的那个对象
> user.f(); // John（this == user）
> admin.f(); // Admin（this == admin）
> 
> admin['f'](); // Admin（使用点符号或方括号语法来访问这个方法，都没有关系。）
> ```
>
> 这个规则很简单：如果 `obj.f()` 被调用了，则 `this` 在 `f` 函数调用期间是 `obj`。所以在上面的例子中 this 先是 `user`，之后是 `admin`。
>
> **在没有对象的情况下调用：`this == undefined`**
>
> 我们甚至可以在没有对象的情况下调用函数：
>
> ```javascript
> function sayHi() {
>   alert(this);
> }
> 
> sayHi(); // undefined
> ```
>
> 在这种情况下，严格模式下的 `this` 值为 `undefined`。如果我们尝试访问 `this.name`，将会报错。
>
> 在非严格模式的情况下，`this` 将会是 **全局对象**（浏览器中的 `window`，我们稍后会在 [全局对象](https://zh.javascript.info/global-object) 一章中学习它）。这是一个历史行为，`"use strict"` 已经将其修复了。
>
> 通常这种调用是程序出错了。如果在一个函数内部有 `this`，那么通常意味着它是在对象上下文环境中被调用的。
>
> **解除 `this` 绑定的后果**
>
> 如果你经常使用其他的编程语言，那么你可能已经习惯了“绑定 `this`”的概念，即在对象中定义的方法总是有指向该对象的 `this`。
>
> 在 JavaScript 中，`this` 是“自由”的，它的值是在调用时计算出来的，它的值并不取决于方法声明的位置，而是取决于在“点符号前”的是什么对象。
>
> 在运行时对 `this` 求值的这个概念既有优点也有缺点。一方面，函数可以被重用于不同的对象。另一方面，更大的灵活性造成了更大的出错的可能。
>
> 这里我们的立场并不是要评判编程语言的这个设计是好是坏。而是要了解怎样使用它，如何趋利避害。
>
> ## [箭头函数没有自己的 “this”](https://zh.javascript.info/object-methods#jian-tou-han-shu-mei-you-zi-ji-de-this)
>
> 箭头函数有些特别：它们没有自己的 `this`。如果我们在这样的函数中引用 `this`，`this` 值取决于外部“正常的”函数。
>
> 举个例子，这里的 `arrow()` 使用的 `this` 来自于外部的 `user.sayHi()` 方法：
>
> ```javascript
> let user = {
>   firstName: "Ilya",
>   sayHi() {
>     let arrow = () => alert(this.firstName);
>     arrow();
>   }
> };
> 
> user.sayHi(); // Ilya
> ```
>
> 这是箭头函数的一个特性，当我们并不想要一个独立的 `this`，反而想从外部上下文中获取时，它很有用。在后面的 [深入理解箭头函数](https://zh.javascript.info/arrow-functions) 一章中，我们将深入介绍箭头函数。
>
> ## [总结](https://zh.javascript.info/object-methods#zong-jie)
>
> - 存储在对象属性中的函数被称为“方法”。
> - 方法允许对象进行像 `object.doSomething()` 这样的“操作”。
> - 方法可以将对象引用为 `this`。
>
> `this` 的值是在程序运行时得到的。
>
> - 一个函数在声明时，可能就使用了 `this`，但是这个 `this` 只有在函数被调用时才会有值。
> - 可以在对象之间复制函数。
> - 以“方法”的语法调用函数时：`object.method()`，调用过程中的 `this` 值是 `object`。
>
> 请注意箭头函数有些特别：它们没有 `this`。在箭头函数内部访问到的 `this` 都是从外部获取的。

## 练习题

[【建议👍】再来40道this面试题酸爽继续(1.2w字用手整理)](https://juejin.cn/post/6844904083707396109)，这份练习题很丰富，可以看看。
