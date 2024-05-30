---
title: Vue和React的区别
date: 2024-05-30 16:24:25 +0800
categories: [前端]
tags: [Vue，React]
---

## 组件

### 组件定义

Vue：模板+样式+脚本，和 HTML 基本一致，易理解

React：写 React 就是写 js，有扩充的 JSX 语法

### 组件结构

Vue：`defineProps` API 在 setup 函数中被编译。

```vue
<!-- App.vue -->
<script setup>
import { ref } from 'vue'
import BlogPost from './BlogPost.vue'
  
const posts = ref([
  { id: 1, title: 'My journey with Vue' },
  { id: 2, title: 'Blogging with Vue' },
  { id: 3, title: 'Why Vue is so fun' }
])
</script>

<template>
	<BlogPost
  	v-for="post in posts"
	  :key="post.id"
  	:title="post.title"
	></BlogPost>
</template>

<!--BlogPost.vue-->
<script setup>
defineProps(['title'])
</script>

<template>
  <h4>{{ title }}</h4>
</template>
```

React 组件是一个函数，它的属性就是函数的参数。

### 组件事件

Vue：通过 `this.$emit('xxx')` 触发，或使用 `defineEmits` 定义

React：执行一个函数

```react
// src/components/TodoInput.js
function TodoInput(props) {
  const { addTodo } = props // 解构出事件 addTodo
  function addTodoHandler() {
    addTodo('some text') // 执行事件 addTodo ，随便传入参数
  }
  return (
      <div>
      	<p onClick={addTodoHandler}>
            todo input
          </p>
      </div>
  )
}
export default TodoInput

// src/App.js
import TodoInput from './components/TodoInput'
import TodoList from './components/TodoList'

function App() {
  function addTodo(t) {
    console.log('addTodo: ', t)
  }
  return (
    <div>
      <TodoInput addTodo={addTodo} />
      <TodoList foo="hello foo" />
    </div>
  )
}
export default App
```

### 子组件

Vue：<slot> 插槽

React：`props.children`

## 模板

Vue：template 模板语法

React：JSX 语法

### 插值

Vue：`{{name}}`

React：`{name}`

### 属性

Vue：`v-bind`/`:xxx`

React：仍然使用插值语法形如 `{name}`

### 自定义事件

Vue：使用 `@xxx` 语法

React：仍然使用插值语法形如 `{addTodo}`

### 样式

Vue：`class="a b"`

React：`className="a b"`

对于动态 class，Vue 有更多预设的 API，React 仍然是插值。

### DOM事件

Vue：使用 `@xxx` 语法形如 `{@click}`

React：参考 HTML 事件模型的语法，习惯使用驼峰命名法

### 条件渲染

Vue：`v-if`，`v-else`，`v-show`

React：js 表达式

```react
{ flag && <SomeComponent /> }
{ flag ? <SomeCompnent/> : <OtherComponent/> }
```

### 列表渲染

Vue：`v-for`

React：使用 js 数组的 map 方法加插值语法

```react
function TodoList(props) {
  const { list = [] } = props
  return (
    <ul>
      {list.map((item) => (
        <li key={item.id}>{item.text}</li>
      ))}
    </ul>
  )
}
export default TodoList
```

### 整体对比

1. JSX 语法相对较为简洁
   - `{xxx}` 大括号里面是 js 的变量或表达式，可实现所有动态功能，包括判断和循环
   - `onXxx` 是 DOM 事件的写法
   - `{x}` 是动态的，`"x"` 是静态的
2. Vue template 的规则更丰富和语义化

## 状态和响应式

Vue：Vue3 使用 `ref` 和 `reactive` 两个函数

React：`useState` API

### 值类型

Vue：Vue3 在 js 中修改或使用 ref 方法的响应式数据时，要使用 `.value` 属性（本人亲测不好使）

React：React 修改数据不是响应式的，而是命令式的，形如 `setXxx`

```react
// src/components/TodoInput.js

import { useState } from 'react'

function TodoInput(props) {
  const { addTodo } = props
  function addTodoHandler() { addTodo('some text') }

  const [count, setCount] = useState(0)
  function increase() {
    setCount(count + 1) //【注意】这里不能写 count++ ，必须执行 setCount 函数，并传入最新的值。原因：在 React 中，状态应该是不可变的。count++ 实际上是在修改原始的状态，而 count+1 是表达式，明确表示将当前的 count 值增加 1，并将这个新值设置为状态。
  }
  return (
    <div>
      <button onClick={increase}>{count}</button>
      <p onClick={addTodoHandler}>todo input</p>
    </div>
  )
}
export default TodoInput
```

## 参考

1. [只会 Vue 不会 React ？22 点证明 React 比 Vue3 更简单](https://juejin.cn/post/7344536653463207973)
2. [组件基础#传递 props](https://cn.vuejs.org/guide/essentials/component-basics.html#passing-props)
3. [将 Props 传递给组件](https://zh-hans.react.dev/learn/passing-props-to-a-component)
