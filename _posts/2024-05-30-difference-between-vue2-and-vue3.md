---
title: Vue2 和 Vue3 的区别
date: 2024-05-30 09:22:54 +0800
categories: [Vue]
tags: [Vue, Vue2, Vue3]
---

## 响应式原理/双向数据绑定

vue2：ES5 [Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 对属性进行劫持，结合发布订阅模式的方式来实现的。

具体流程是：

1. 组件实例化时，Vue 遍历组件 data 对象中的所有属性，并使用 defineProperty() 为每个属性创建 getter 和 setter 函数
2. 当模板中访问到某个数据时，Vue 通过 getter 进行依赖收集，即将当前组件的渲染函数或计算属性添加到这个数据属性的依赖列表中
3. 当数据发生变化时，setter 被触发，Vue 会通知所有依赖这个数据属性的地方，告诉它们数据已经改变
4. 依赖的触发会导致视图的更新，或者是相关计算属性的重新计算，Vue 以此确保了 UI 总是与数据保持同步

vue3：ES6 [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 对数据代理。

具体流程是：

1. Vue 通过 Proxy 包裹组件的 data 返回的对象，以及其他响应式状态如 props、computed 等，任何对这些对象的操作都可以被拦截（基础是 ES6 的 [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)，已经非常强大）
2. 当读取数据属性时，Proxy 的 get 拦截器会被调用，Vue 会进行依赖收集，即将当前的副作用函数（例如渲染函数或计算属性）记录为这个属性的依赖
3. 当修改数据属性时，Proxy 的 set 拦截器会被调用，Vue 会触发所有依赖于这个属性的副作用函数，使得它们重新运行，从而更新视图或计算属性的值

二者区别：

- 监听数据：defineProperty 需要知道具体属性且只能监听某个属性；Proxy 直接监听整个对象
- 对原对象的影响：defineProperty 是通过在原对象身上新增或修改属性增加描述符的方式实现的监听效果，一定会修改原数据；而 Proxy 只是原对象的代理，Proxy 会返回一个代理对象不会在原对象上进行改动，对原数据无污染
- 对数组的监听：因为数组 length 的特殊性（length 的描述符 configurable 和 enumerable 为 false，并且*妄图*修改 configurable 为 True 时 JS 会直接报错：VM305:1 Uncaught TypeError: Cannot redefine property: length），defineproperty 无法监听数组长度变化，Vue 只能通过重写数组方法的方式变现达成监听的效果，但仅重写数组方法还是不能解决修改数组下标时监听的问题，只能再使用自定义 set 函数的方式，但是对 Proxy 代理对象进行操作时，所有的操作都会被捕捉，包括数组的方法和 length 操作
- defineProperty 只能监听到 value 的 get/set 操作，但是 Proxy 有更全面和丰富的拦截器（除 `[[getOwnPropertyNames]]` 以外所有JS 的对象操作）

## TypeScript支持

Vue3 由 TypeScript 重写，相对于 Vue2 有更好的 TypeScript 支持。

## API 类型

Vue2 是选项式 API（Options API），一个逻辑会散乱在文件不同位置（data、props、computed、watch、生命周期钩子等），导致代码的可读性变差。当需要修改某个逻辑时，需要上下来回跳转文件位置。

Vue3 组合式API（Composition API）则很好地解决了这个问题，可将同一逻辑的内容写到一起，增强了代码的可读性、内聚性，其还提供了较为完美的逻辑复用性方案。

## 生命周期钩子函数

基本一致，Vue3 的钩子函数前面多了"on"标识。明显的区别在于，组合式 API 在 setup 函数执行时创建组件，选项式 API 在 beforeCreate 和 created 之间创建组件。Vue3 需要先导入生命周期钩子。

![组件生命周期图示](https://cn.vuejs.org/assets/lifecycle_zh-CN.W0MNXI0C.png)

## 多根节点/Fragments

在 Vue2 中，模板中只能有一个根节点，Vue3 支持多个根节点

```vue
// 以下代码在 Vue2 中会报错，需要在外面包裹一层 <div>
<template>
<header></header>
<main></main>
<footer></footer>
</template>
```

## 参考

1. [defineProperty 和 Proxy区别](https://segmentfault.com/a/1190000041084082)
