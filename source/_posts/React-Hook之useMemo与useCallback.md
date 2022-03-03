---
title: React Hook之useMemo与useCallback
date: 2022-03-03 17:15:36
categories: React
tags:
---
### 一、useMemo:
用法：

```JavaScript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```
返回一个memoized值，即有记忆的值。可以类比成vue中的computed属性，只有相关依赖项发生改变才会重新计算。
该方法有两个参数，第一个参数是一个函数，第二个参数是一个依赖项数组，他仅会在某个依赖项改变时才会重新计算memoized值。这种优化有助于避免在每次渲染时都进行高开销的计算。
注意：
①、传入useMemo的函数在初次渲染期间就会执行，因此不要在函数内部执行与渲染无关的操作。
②、如果没有传入依赖项数组，useMemo会在每次渲染时都计算新的值。
下面举个例子，页面中有count跟val两个变量，有两个按钮分别操作count跟val的值，还有一个方法用于计算count与数值10的和。代码如下：
<!-- more -->

```JavaScript
import { useState } from 'react';

export default function UseMemo () {
	const [count, setCount] = useState(0)
	const [val, setVal] = useState(66)

	const getNum = () => {
		console.log('getNum')
		return 10 + count
	}

	return (
		<>
			<p>结果：{getNum()}</p>
			<button onClick={() => setCount(count + 1)}>+1</button>
			<button onClick={() => setVal(val + 10)}>+10</button>
		</>
	)
}
```
分别改变count跟val的值发现getNum方法每次都会重新执行一遍，但是按理说改变val的值跟getNum方法没有毛关系，不应该执行才对啊，因为它只跟count的值有关系。如此一来，这样不就会引发性能问题吗？所以这时候useMemo就派上用场了。
使用useMemo来解决这个问题，仅当count改变时getNum才重新计算。代码如下：

```JavaScript
import { useState, useMemo } from 'react';

export default function UseMemo () {
	const [count, setCount] = useState(0)
	const [val, setVal] = useState(66)

	const getNum = useMemo(() => {
		console.log('getNum')
		return 34 + count
	}, [count])

	return (
		<>
			<p>结果：{getNum}</p>
			<button onClick={() => setCount(count + 1)}>+1</button>
			<button onClick={() => setVal(val + 10)}>+10</button>
		</>
	)
}
```
现在再分别改变count与val的值发现只有count值改变时getNum才会去重新执行。
注意：使用useMemo方法后getNum是该方法返回的一个值，因此使用时只需要使用getNum就行了。
不使用useMemo时，getNum是一个方法，使用时需要带上括号。

### 二、useCallback：
用法：

```JavaScript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```
返回的是一个memoized回调函数。
该方法有俩参数，一个是函数，一个是依赖项数组，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。
看官方文档是一脸懵逼，不知道有啥用，这里举例说，还是上面的例子，页面有变量count跟val，有俩按钮分别操作这俩变量，同时还有一个子组件Child，希望改变count时子组件会重新渲染，改变val时子组件不会重新渲染，这是useCallback就有用武之地了。
UseCallback.js代码如下：

```JavaScript
import { useCallback, useState } from 'react';
import Child from './Child.js'

export default function UseCallback () {
	const [count, setCount] = useState(6)
	const [val, setVal] = useState(1)
	
	const getNum = useCallback(() => {
         /*
         这里是需要缓存的函数，只有当count变化时，该函数才会更新。
        */
	}, [count])


	return (
		<>
			<p>count值：{count}</p>
			<p>val值：{val}</p>
			<Child getNum={getNum} />
			<button onClick={() => setCount(count + 10)}> + </button>
			<button onClick={() => setVal(val + 1)}> + </button>
		</>
	)
}
```
Child.js代码如下：

```JavaScript
export default function Child () {
	return (
		<>
			<div>hello world!{new Date().getTime()}</div>
		</>
	)
}
```
这里为了证明子组件有没有重新渲染，我在页面中打印出了当前时间距离1970年1月1日的时间戳。
现在点击俩按钮分别更改count和val的值，发现子组件都会重新渲染，因为页面上显示的值在不断变化。这是为啥，这里缺少一个东西，那就是memo，在导出Child组件时需要使用memo将其包裹起来，而memo则需要先导入。修改后的代码如下：

```JavaScript
import { memo } from 'react'

export default memo(function Child () {
	return (
		<>
			<div>hello world!{new Date().getTime()}</div>
		</>
	)
})
```
现在在分别点击俩按钮改变count和val的值，发现只有count改变时，子组件才会重新渲染，改变val的值时子组件不会重新渲染。
如果改变useCallback依赖项数组为[count, val]，会发现无论是改变count和val，子组件都会重新渲染。
使用useCallback时，子组件一定要使用memo进行包裹，其作用是对组件前后两次进行浅对比，阻止不必要的更新，否则会无效。

### useMemo与useCallback对比：
通过上面会发现useMemo和useCallback差不多，都是接收俩参数，第一个为回调函数，第二个为依赖项数组。
共同点：
仅当依赖项发生改变时，才会重新计算结果，也就是起到缓存作用。

区别：
1、useMemo的计算结果是return回来的值，主要用于缓存计算结果的值，应用场景是只有某一依赖项改变时才去重新计算。
2、useCallback的计算结果是函数，主要用于缓存函数，应用场景：需要缓存的函数，因为在函数式组件中，任何一个state变化时，整个组件都会重新渲染，而有时候只是父组件中数据变化了，子组件并没有变化，这时应该把子组件缓存起来，提高性能，避免资源浪费。