---
title: React封装全局Toast方法
date: 2022-03-07 18:08:02
categories: React
tags:
---
项目中经常会使用到Toast，尤其是接口报错或者用户执行某一操作成功或失败后给予提示。
大概思路：
1）、创建Toast组件。
2）、引入Toast组件，封装showToast方法，将其插入到dom文档中。
3）、封装全局方法，便于在其他组件中随时使用，而不用每次都引入。
<!-- more -->
toast.css代码：

```
.toast-container {
	width: 100%;
	height: 100%;
	background-color: transparent;
	position: fixed;
	left: 0;
	top: 0;
	display: flex;
	justify-content: center;
	align-items: center;
}
.toast {
	padding: 10px;
	max-width: 220px;
	min-width: 120px;
	border-radius: 10px;
	background-color: rgba(0, 0, 0, 0.7);
	font-size: 18px;
	color: #fff;
	text-align: center;
	animation: translate 0.5s ease-in-out;
}

@keyframes translate {
	0% {
		opacity: 0
	}
	100% {
		opacity: 1
	}
}

```

Toast.js代码：

```JavaScript
import "./toast.css"
import { useEffect, useState } from 'react'

export default function Toast (props) {
	let [isShow, setIsShow] = useState(true)
	useEffect(() => {
		if (isShow) {
			let timer = setTimeout(() => {
				setIsShow(false)
				document.body.removeChild(document.getElementById('my-toast'))
			}, props.duration || 2200)
			return () => {
				clearTimeout(timer)
			}
		}
	}, [isShow])
	
	return (
		<>
			{
				isShow ?
				(
					<div className="toast-container">
						<div className="toast">{props.children}</div>
					</div>
				) : ""
			}
		</>
	)
}
```
showToast.js代码：

```JavaScript
import ReactDOM from 'react-dom';
import Toast from './Toast.js'

const toast = {
	showToast: (txt, duration = 2200) => {
		let div = document.createElement('div')
		div.id = 'my-toast'
		document.body.appendChild(div)
		const show = <Toast duration={duration}>{txt}</Toast>
		ReactDOM.render(show, div)
	}
}

export default toast
```
最后封装全局方法，这里有两种，一种是挂载到window对象上，一种是直接挂载到React上。都需要先在项目的index.js中引入上面的index.js文件。
index.js部分代码：

```JavaScript
import React from 'react';
import Toast from './components/Toast/index.js'
window.Toast = Toast.showToast
// 或者
React.$toast = Toast.showToast
```
如果是挂载到window对象上，那么在其他地方使用：

```JavaScript
window.Toast('什么鬼！什么鬼！')
```
如果是挂载到React上，则这样使用：

```JavaScript
React.$toast('什么鬼！什么鬼！')
```

