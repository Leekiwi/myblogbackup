---
title: ES6中简约强大数组操作组合
date: 2018-09-07 18:25:20
tags: javascript
categories: ES6
---

## ES6 数组新增方法

### reduce

`array.reduce(callback[, initialValue])`

<!-- more -->

数组求和

```js
const numbers = [10, 20, 30, 40]
numbers.reduce((prev, cur, index, arr) => {
  console.log('prev: ' + prev + '; ' + 'cur: ' + cur + ';')
  return prev + cur
})
```

```js
prev: 10
cur: 20
prev: 30
cur: 30
prev: 60
cur: 40
```

这第二个参数就是设置 prev 的初始类型和初始值，比如为 0，就表示 prev 的初始值为 number 类型，值为 0，因此，reduce 的最终结果也会是 number 类型。

```js
const numbers = [10, 20, 30, 40]
numbers.reduce((prev, cur, index, arr) => {
  console.log('prev: ' + prev + '; ' + 'cur: ' + cur + ';')
  return prev + cur
}, 0)
```

```js
prev: 0
cur: 10
prev: 10
cur: 20
prev: 30
cur: 30
prev: 60
cur: 40
```

参考：

https://www.zhangxinxu.com/wordpress/2013/04/es5%E6%96%B0%E5%A2%9E%E6%95%B0%E7%BB%84%E6%96%B9%E6%B3%95/

https://segmentfault.com/a/1190000005921341

https://segmentfault.com/a/1190000013972464

https://segmentfault.com/a/1190000013121115
