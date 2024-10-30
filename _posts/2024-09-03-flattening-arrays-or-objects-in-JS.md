---
title: JS 中拍平数组/对象
date: 2024-09-03 14:51:02 +0800
categories: [JavaScript]
tags: [JavaScript, JS, arr, obj, flat]
---

## JS 数组拍平

```javascript
function flat(arr) {
    let newArr = [];

    for (let i = 0; i < arr.length; i++) {
        let ele = arr[i];
        if (Array.isArray(ele)) {
            newArr = newArr.concat(flat(ele.slice()));
        } else {
            newArr.push(ele);
        }
    }

    return newArr;
}

const arr = [1, 2, [3, 4, [5]]];

console.log(flat(arr));
// [ 1, 2, 3, 4, 5 ]
```

## JS 对象拍平


```javascript
const origin_obj = {
    a: {
        b: 1,
        c: 2,
        d: { e: 5 },
    },
    b: [1, 3, { a: 2, b: 3 }],
    c: 3
}

function flatten(obj) {
    if (typeof obj != 'object') {
        throw new Error('TypeError');
    }
    const flat_obj = {};
    const flat = (obj, pre = '', mid_str = '.') => {
        for (let key in obj) {
            if (typeof obj[key] == 'object' && typeof obj[key] != null) {
                flat(obj[key], pre + key + mid_str);
            } else {
                flat_obj[pre + key] = obj[key];
            }
        }
    }

    flat(obj);
    return flat_obj;
}

console.log(flatten(origin_obj));

// output:
// {
//     'a.b': 1,
//     'a.c': 2,
//     'a.d.e': 5,
//     'b.0': 1,
//     'b.1': 3,
//     'b.2.a': 2,
//     'b.2.b': 3,
//     c: 3
//   }
```

