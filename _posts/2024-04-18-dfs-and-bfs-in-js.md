---
title: JS中的DFS和BFS
date: 2024-04-18 23:47:00 +0800
categories: [JavaScript]
tags: [JS,dfs,bfs]
---

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
