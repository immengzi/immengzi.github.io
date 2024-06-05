---
title: JavaScript 中的深拷贝
date: 2024-06-05 21:10:10 +0800
categories: [JavaScript]
tags: [deepClone, JavaScript, JS]
---

## 什么是深拷贝

对象的**深拷贝**是指其属性与其拷贝的源对象的属性不共享相同的引用（指向相同的底层值）的副本。因此，当你更改源或副本时，可以确保不会导致其他对象也发生更改；也就是说，你不会无意中对源或副本造成意料之外的更改。这种行为与[浅拷贝](https://developer.mozilla.org/zh-CN/docs/Glossary/Shallow_copy)的行为形成对比，在浅拷贝中，对源或副本的更改可能也会导致其他对象的更改（因为两个对象共享相同的引用）。

在 JavaScript 中，标准的内置对象复制操作（[展开语法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_syntax)、[`Array.prototype.concat()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/concat)、[`Array.prototype.slice()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)、[`Array.from()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from)、[`Object.assign()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) 和 [`Object.create()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)）不创建深拷贝（相反，它们创建浅拷贝）。

## JSON.stringify()

如果一个 JavaScript 对象可以被[序列化](https://developer.mozilla.org/zh-CN/docs/Glossary/Serialization)，则存在一种创建深拷贝的方式：使用 [`JSON.stringify()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) 将该对象转换为 JSON 字符串，然后使用 [`JSON.parse()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse) 将该字符串转换回（全新的）JavaScript 对象：

```javascript
let ingredients_list = ["noodles", { list: ["eggs", "flour", "water"] }];
let ingredients_list_deepcopy = JSON.parse(JSON.stringify(ingredients_list));
```

然而，虽然上面代码中的对象足够简单，可以[序列化](https://developer.mozilla.org/zh-CN/docs/Glossary/Serialization)，但许多 JavaScript 对象根本不能序列化——例如，[函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Functions)（带有闭包）、[Symbol](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)、在 [HTML DOM API](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_DOM_API) 中表示 HTML 元素的对象、递归数据以及许多其他对象。在这种情况下，调用 `JSON.stringify()` 来序列化对象将会失败。所以没有办法对这些对象进行深拷贝。

## 考虑循环引用

```javascript
const myDeepClone = (obj, map = new Map()) => {
    if (obj == null || typeof obj !== "object") {
        return obj;
    }
    if (/^(Function|RegExp|Date|Map|Set)$/i.test(obj.constructor.name)) {
        return new obj.constructor(obj);
    }
    if (map.get(obj)) {
        return map.get(obj);
    }
    let cloneObj = Array.isArray(obj) ? [] : {};
    map.set(obj, cloneObj);
    for (key in obj) {
        if (Object.prototype.hasOwnProperty.call(obj, key)) {
            cloneObj[key] = myDeepClone(obj[key], map);
        }
    }
    return cloneObj;
}

var A = {
    a: 1,
    b: [1, 2, 3],
    c: {
        "0": 0
    },
    d: undefined,
    e: null,
    f: new Date()
};
A.A = A;
console.log("A", A);
var B = myDeepClone(A)
console.log("Cloned A:", B);
```

输出：

```sh
PS E:\code\js> node "e:\code\js\deepClone.js"
A <ref *1> {
  a: 1,
  b: [ 1, 2, 3 ],
  c: { '0': 0 },
  d: undefined,
  e: null,
  f: 2024-06-05T20:23:56.343Z,
  A: [Circular *1]
}
Cloned A: <ref *1> {
  a: 1,
  b: [ 1, 2, 3 ],
  c: { '0': 0 },
  d: undefined,
  e: null,
  f: 2024-06-05T20:23:56.343Z,
  A: [Circular *1]
}
```

