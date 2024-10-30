---
title: LeetCode 438. 找到字符串中所有字母异位词
date: 2024-07-28 20:51:23 +0800
categories: [algorithm]
tags: [JavaScript, JS, Heteronym, String]
---

## 题目描述

给定两个字符串 s 和 p，找到 s 中所有 p 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

异位词 指由相同字母重排列形成的字符串（包括相同的字符串）。

示例 1:

输入: s = "cbaebabacd", p = "abc"
输出: [0,6]
解释:
起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。

示例 2:

输入: s = "abab", p = "ab"
输出: [0,1,2]
解释:
起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。

## 整体思路

为了找出满足条件的起始索引，可以遍历起始索引 i 从 0 到 s.length-p.length，逐渐向右扩充边界，第一个考虑的右边界即起始索引，故 r 的初始值应设置为 i-1，最大值为 i-1+p.length。

每次在起始索引为 k 处找到目标子串，则说明 k 到 k-1+p.length 之间的子串已经满足部分条件，仅需判断第 k+p.length 是否满足内部条件，再判断子串整体是否满足条件即可。

判断子串整体是否满足条件，最开始我使用了 js 内置对象 Set，但我忽视了相同字符的情况，于是我改为 String.split('').sort().join('') 对当前子串和目标子串分别完成转换再进行比对，结果是超出时间限制，因为上述方法的复杂度达到了 O(n)+O(nlogn)+O(n)=O(nlogn)。

我浏览了网上比对异位词的方法，得知 .charCodeAt() 这个可爱的方法，它可以获取一个字符的 Unicode 值。为避免数组索引位数过长，一般的做法是减去 'a'.charCodeAt()，可将其预设为 base。

## 首次通过代码

```js
// s = "abab", p = "ab"
s = "cbaebabacd", p = "abc"

var findAnagrams = function (s, p) {
    let targetArray = p.split('');
    let indexArray = [];
    let tmpStr = '';
    const n = s.length;
    for (let i = 0; i <= n - p.length; i++) {
        let r = i - 1 + tmpStr.length;
        while (r + 1 < i + p.length && targetArray.includes(s[r + 1])) {
            tmpStr += s[r + 1];
            r++;
        }

        if (compare(tmpStr, p)) {
            indexArray.push(i);
            tmpStr = tmpStr.slice(1);
            continue;
        }

        tmpStr = '';
    }
    return indexArray;
};

function compare(str1, str2) {
    if (str1.length !== str2.length) {
        return false;
    }

    const base = 'a'.charCodeAt(0);
    let arr1 = Array(26).fill(0);
    let arr2 = Array(26).fill(0);

    for (let i = 0; i < str1.length; i++) {
        arr1[str1.charCodeAt(i) - base]++;
        arr2[str2.charCodeAt(i) - base]++;
    }

    for (let i = 0; i < 26; i++) {
        if (arr1[i] !== arr2[i]) {
            return false;
        }
    }

    return true;
}

console.log(findAnagrams(s, p));
```

## 优化版本

```js
/**
 * @param {string} s
 * @param {string} p
 * @return {number[]}
 */
var findAnagrams = function (s, p) {
    const sLen = s.length;
    const pLen = p.length;
    const base = 'a'.charCodeAt(0);
    const result = [];

    if (sLen < pLen) return [];

    let sCount = Array(26).fill(0);
    let pCount = Array(26).fill(0);

    for (let i = 0; i < pLen; i++) {
        sCount[s.charCodeAt(i) - base]++;
        pCount[p.charCodeAt(i) - base]++;
    }

    if (arraysEqual(sCount, pCount)) {
        result.push(0);
    }

    for (let i = pLen; i < sLen; i++) {
        sCount[s.charCodeAt(i) - base]++;
        sCount[s.charCodeAt(i - pLen) - base]--;

        if (arraysEqual(sCount, pCount)) {
            result.push(i - pLen + 1);
        }
    }

    return result;
};

function arraysEqual(arr1, arr2) {
    return arr1.every((val, index) => val === arr2[index]);
}
```

