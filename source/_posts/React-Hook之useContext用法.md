---
title: React Hook之useContext用法
date: 2022-03-02 17:55:26
categories: React
tags:
---
```JavaScript
const value = useContext(MyContext);
```
接收一个context对象（createContext的返回值）并返回该context的当前值。当前的context的值由上层组件中距离当前组件最近的<MyContext.Provider></MyContext.Provider>的value属性决定。
当组件上层最近的<MyContext.Provider>更新时，该Hook会触发重渲染，并使用最新传递给MyContext的context value值。
使用context可进行父子组件传值或跨组件传值。
基础用法，比如现在这里有一个列表组件（父组件），里面有一个列表项组件（子组件），子组件要获取到父组件的值，可以使用props传值，这里就略过，讲一下子组件通过context来取值。
<!-- more -->
父组件List.js代码：

```JavaScript
import ListItem from './ListItem.js'
import { useState } from 'react'

export default function List () {
	let listData = [
		{
			name: "张三",
			age: 34,
			id: 1
		}, {
			name: "李四",
			age: 23,
			id: 2
		}, {
			name: "王五",
			age: 45,
			id: 3
		}
	]
	let [list, setList] = useState(listData)
	
	return (
		<ul>
			<ListItem />
		</ul>
	)
}
```
子组件ListItem.js：

```JavaScript
export default function ListItem () {
    /*
    此处获取父组件传的值，然后渲染每一列表项的数据，该怎么写呢？？？？
     */
}
```
首先创建ListContext.js文件来统一管理上下文的实例，通过export default将实例导出，在List.js文件中引入，包裹需要获取父组件值的子组件，然后在子组件中导入该实例进行使用。
ListContext.js代码：

```JavaScript
import { createContext } from 'react';

const ListContext = createContext()

export default ListContext
```
现在在List.js文件中引入该实例，修改后代码如下：

```JavaScript
import ListItem from './ListItem.js'
import { useState } from 'react'
import ListContext from './ListContext.js'


export default function List () {
	let listData = [
		{
			name: "张三",
			age: 34,
			id: 1
		}, {
			name: "李四",
			age: 23,
			id: 2
		}, {
			name: "王五",
			age: 45,
			id: 3
		}
	]
	let [list, setList] = useState(listData)
	
	return (
		<ul>
			<ListContext.Provider value={list}>
				<ListItem /> 
			</ListContext.Provider>
		</ul>
	)
}
```
子组件ListItem.js中代码：

```JavaScript
import { useContext } from 'react';
import ListContext from './ListContext.js'

export default function ListItem () {
    let list = useContext(ListContext)
	// console.log('list-list', list)。此处可获取父组件中传过来的值。
	return (
		<>
			{
				list.map(item => {
					return (
						<li key={item.id}>
							姓名：{item.name}，年龄：{item.age}
							<p>hello world!</p>
						</li>
					)
				})
			}
		</>
	)
}
```
这么一看，父子组件传值还是使用props传值方便多了，使用这种方式实属多此一举，确实，但是如果子组件层级较多呢？也就是涉及到跨组件传值，层层嵌套呢？这时候使用props传值就有点麻烦了，使用context会方便得多。
比如ListItem.js组件里又嵌套了一层Item组件，在Item组件里想获取祖先组件List的值。
现在需要修改ListItem.js文件代码为：

```JavaScript
import Item from './Item.js'

export default function ListItem () {
	return (
		<Item />
	)
}
```
孙子组件Item.js代码：

```JavaScript
import ListContext from './ListContext.js'
import { useContext } from 'react'

export default function Item () {
	let data = useContext(ListContext)
	console.log('list-list', data)
    /* 在这里可以获取到祖先组件传递的值 */
	return (
		<div>测试一下哦~</div>
	)
}
```