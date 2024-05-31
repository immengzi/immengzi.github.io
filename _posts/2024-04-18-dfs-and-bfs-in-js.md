---
title: JS 中的 DFS 和 BFS
date: 2024-04-18 23:47:00 +0800
categories: [JavaScript]
tags: [JS, JavaScript, DFS, BFS]
---

在 JavaScript 中，可以通过数组的方法来模拟栈（stack）和队列（queue）的操作：

1. 模拟栈（Stack）

   栈是一种后进先出（LIFO, Last In First Out）的数据结构，可以使用数组的以下方法来模拟：

   - **入栈（push）操作**：使用 `Array.push()` 方法将元素添加到数组的末尾。

   - **出栈（pop）操作**：使用 `Array.pop()` 方法从数组的末尾移除元素，并返回该元素。

2. 模拟队列（Queue）

   队列是一种先进先出（FIFO, First In First Out）的数据结构，可以使用数组的以下方法来模拟：

   - **入队（enqueue）操作**：使用 `Array.push()` 方法将元素添加到数组的末尾。

   - **出队（dequeue）操作**：使用 `Array.shift()` 方法从数组的开头移除元素，并返回该元素。

接下来是具体的 JavaScript 实现代码，展示了如何在实际的数据结构（这里是一个简单的树结构）上应用 DFS 和 BFS 搜索算法。

由于 JavaScript 函数调用会自动使用调用栈，因此在进行深度优先搜索（DFS）时，可以直接通过递归函数调用来利用这一点，无需显式地使用栈结构。而广度优先搜索（BFS）则需要显式地使用队列来控制搜索的顺序，这里通过数组的 `push()` 和 `shift()` 方法模拟队列操作。

```js
const tree = {
    value: 1,
    children: [
        {
            value: 2,
            children: [
                {
                    value: 4,
                    children: []
                },
                {
                    value: 5,
                    children: []
                }
            ]
        },
        {
            value: 3,
            children: [
                {
                    value: 6,
                    children: []
                },
                {
                    value: 7,
                    children: []
                }
            ]
        }
    ]
}

function dfs(obj) {
    console.log(obj.value)
    for (child of obj.children) {
        dfs(child)
    }
}

dfs(tree)

function bfs(obj) {
    let queue = []
    queue.push(obj)
    while (queue.length != 0) {
        let top = queue.shift()
        console.log(top.value)
        for (child of top.children) {
            queue.push(child)
        }
    }
}

bfs(tree)

// output
// 1
// 2
// 4
// 5
// 3
// 6
// 7
// 1
// 2
// 3
// 4
// 5
// 6
// 7
```