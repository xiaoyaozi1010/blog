---
title: 核心JavaScript性能测试
tags: [Javascript, 性能]
---

__运行环境：chrome  56.0.2924.87 (64-bit) MacOs__
## 1. parseInt 与 + 操作性能比较：
```js
var count = 10000000;
var arr = [];
for(var i = 0; i < count; i++){
   arr.push(i + '');
}
console.time();
for(var i = 0, l = arr.length; i < l; i++){
   parseInt(arr[i]);
   // +(arr[i]);
}
console.timeEnd();
```
结果：

<!-- more -->

> parseInt: 660ms左右

> '+' : 420ms左右

## join与for += 操作性能比较：
```js
var count = 10000000;
var arr = [];
var str = '';
console.time();
for(var i = 0; i < count; i++){
//    arr.push(i);
    str += i;
}
// console.time();
// arr.join();
console.timeEnd();
```

> += : 1500ms左右；

> join: 1400ms左右；

*todo:计算平均值*