---
title: javascript之recude方法
date: 2020-06-13 09:53:17
categories: JavaScript
tags: 
---
以前看到reduce方法，总是看得我头皮发麻，今天无意间又遇到他了，于是学习了下，接触之后，觉得这个方法还挺好用的，在很多地方都可以派上用场，比如，数组中元素求和、数组去重、求数组中的最大值或最小值等等都可以用到它。

　　reduce() 方法接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终计算为一个值。

　　语法：`array.reduce(function(total, currentValue, currentIndex, arr), initialValue)`

　　可以看出它接收一个回调函数和一个初始值。

　　其中total为初始值或者计算后的返回值（必须）、currentValue为当前元素（必须）、currentIndex为当前元素索引（可选）、arr为当前元素所属的对象（可选）、initialValue为传递给函数的初始值

<!--more-->

　　案例1：数组去重
// 数组去重
```javascript
var arr = [12, 34, 34, 342, 345, 34, 123, 345, 45, 12]
var newArr = arr.reduce(function (prev, next) {
    prev.indexOf(next) == -1 && prev.push(next)
    return prev
}, [])
console.log(arr)  // [12, 34, 34, 342, 345, 34, 123, 345, 45, 12]
console.log(newArr) // [12, 34, 342, 345, 123, 45]
```
　　初始化一个空数组，判断下一个元素是否在当前数组中，不存在则添加到当前数组中。

　　案例2：数组中元素求和
// 数组求和
```javascript
var arr = [1, 2, 3, 4, 5]
var total = arr.reduce(function (prev, next) {
    return prev + next
}, 0)
console.log(total)
```
　　将0当做reduce回调函数中的初始值，然后依次累加


　　案例3：求数组中最大值或最小值：
// 获取数组中最大值
```javascript
var arr = [134798, 3478973, 12, 345, 355, 425, 1342356, 3425566, 7908798]
var max = arr.reduce(function (prev, next) {
    return Math.max(prev, next)    // Math.min(prev, next)
}, 0)
console.log(max)
```

　　tips：**initialValue为传递给函数的初始值，假如该值不存在时，则回调函数的初始值为数组中的第一项，即回调函数中的prev值**