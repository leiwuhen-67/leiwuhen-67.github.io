---
title: React Hook之useReducer使用及跨组件通信
date: 2022-03-03 14:28:31
categories: React
tags:
---
语法：

```JavaScript
const [state, dispatch] = useReducer(reducer, initialArg, init);
```
useReducer是useState的替代方案。在某些场景下，useReducer会比useState更使用，例如state逻辑较复杂，里面包含多个子组件，或者下一个state依赖于之前的state等。并且，使用sueReducer还能给那些会触发深更新的组件做性能优化，因为你可以向子组件传递dispatch而不是回调函数。
例如，这里有一个例子，页面上显示一个变量count的值，同时有两个按钮，分别对这个变量进行加操作和减操作。
先来写reducer方法，这里便于阅读和更改，把erducer方法单独抽离出来放在reducer.js文件中，代码如下：
<!-- more -->

```JavaScript
export default function reducer (state, action) {
	switch (action.type) {
		case "decrement":
			return {
				count: state.count - 1
			}
			break;
		case "increment":
			return {
				count: state.count + 1
			}
	}
}
```
这个reducer方法就是一会儿需要用到的加操作和减操作。
现在新建一个Parent.js文件，开始完成上面的需求，代码如下：

```JavaScript
import { useReducer } from 'react';
import reducer from './reducer.js'

export default function Parent () {
	const initialState = {count: 0}
	const [state, dispatch] = useReducer(reducer, initialState)

	return (
		<>
			<div>Count：{state.count}</div>
			<button onClick={() => dispatch({type: "decrement"})}>-</button>
			<button onClick={() => dispatch({type: "increment"})}>+</button>
		</>
	)
}
```
现在点击按钮加和减就可以进行加减操作，并能看到数据发生变化。
针对这个需求可以看出使用useState明显会更方便，，现在稍加改变下需求，在Parent组件里有两个子组件，ChildOne组件和ChildTwo组件，在ChildTwo组件里能获取到父组件Parent的数据，在ChildOne组件也能获取到父组件Parent的数据，同时在ChildOne组件里可更改父组件里的数据，保证父组件和俩子组件里数据能实时更新，这时候我们就需要用到useReducer和useContext来实现了。
思路大致是这样：
1）、父组件通过createContext.Prover组件，value属性来传递数据给俩子组件。
2）、子组件通过useContext方法来获取父组件的数据。
3）、子组件通过reducer方法来更改父组件数据。
将createContext这一步的逻辑放在新文件PageContext.js中，代码如下：

```JavaScript
import { createContext } from 'react'
const DataContext = createContext()
export default DataContext
```
父组件Parent.js代码：

```JavaScript
import { useReducer } from 'react';
import reducer from './reducer.js'
import PageContext from './PageContext.js'
import ChildOne from './ChildOne.js'
import ChildTwo from './ChildTwo.js'

export default function Page6 () {
	const initialState = {count: 0}
	const [state, dispatch] = useReducer(reducer, initialState)

	return (
		<>
			<div>Count：{state.count}</div>
			<PageContext.Provider value={{state, dispatch}}>
				<ChildOne />
				<ChildTwo />
			</PageContext.Provider>
		</>
	)
}
```
在子组件ChildTwo中通过useContext来获取数据，ChildTwo.js代码如下：

```JavaScript
import {useContext} from 'react'
import PageContext from '../PageContext.js'

export default function ChildTwo () {
	const dataContext = useContext(PageContext)
	
	return (
		<>
			<p>{dataContext.state.count}</p>
			<div>ChildTwo页面</div>
		</>
	)
}
```
现在在子组件ChildOne中通过useContext来获取数据，并通过reducer方法来更改数据。ChildOne.js代码如下:

```JavaScript
import {useContext} from 'react'
import PageContext from '../PageContext.js'

export default function Child (props) {
	const dataContext = useContext(PageContext)
	
	return (
		<>
			<div>count：{dataContext.state.count}</div>
			<button onClick={() => dataContext.dispatch({type: "decrement"})}>change</button>
		</>
	)
}
```