---
title: 使用create-react-app快速搭建React开发环境及路由等的使用
date: 2022-03-01 14:17:48
categories: React
tags:
---
create-react-app是来自于FaceBook，通过该命令我们无需配置就能快速构建React开发环境。是基于Webpack + ES6/

```JavaScript
npm install -g create-react-app (cnpm install -g create-react-app)
create-react-app react-demo
cd react-demo
npm start
```

现在就可以运行起来了。然而一个项目中还会涉及到页面间跳转等，这就需要使用到路由管理了。
安装react-router-dom：`npm install react-router-dom --save` （ps：这里安装完后版本是6.2.1）
<!-- more -->
现在来更改官网例子，首先在src文件夹下创建components文件夹，然后创建Home.js文件、Page1.js文件、Page2.js文件、Page3.js文件，此时目录就如下所示。
![alt 目录](https://moguxingqiu.oss-cn-hangzhou.aliyuncs.com/upload/config/blog/c92ad280175f8cc74857caf1fed61931.jpeg)

![alt 预览结果](https://moguxingqiu.oss-cn-hangzhou.aliyuncs.com/upload/config/blog/573dd6fd5a51e5771168215db9004f93.jpeg)
Home.js文件代码如下：

```JavaScript
import React from 'react';
import { Link, Outlet } from 'react-router-dom';
 
export default function Home () {
	return(
		<div>
			<div>This is Home!</div>
			<Link to="/Page1?name=tom" style={{color:'red'}}>
				<div>点击跳转到Page1</div>
			</Link>
			
			<Link to="/Page2" style={{color:'blue'}}>
				<div>点击跳转到Page2</div>
			</Link>

			<Link to="/Page3" style={{color:'gold'}}>
				<div>点击跳转到Page3</div>
			</Link>

			 <Outlet />

		</div>
	);
}
```

 
Page1.js代码如下：

```JavaScript
export default function Page1 () {
	return(
		<div>
			<div>This is Page1!</div>
		</div>
	);
}
```

Page2.js与page3.js代码内容类似。
页面创建完了，现在来配置路由，更改App.js文件中内容，引入路由管理所需的组件，以及刚刚新建的几个页面。代码如下：
### ①、嵌套路由的配置。

```JavaScript
import React from 'react';
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';
import Home from './components/Home';
import Page1 from './components/Page1';
import Page2 from './components/Page2';
import Page3 from './components/Page3';
import './App.css';

export default class App extends React.Component {
  render () {
    return (
      <Router>
        <Routes>
          <Route path="/" exact element={<Home/>} >
            <Route path="/Page1" element={<Page1/>} />
            <Route path="/Page2" element={<Page2/>} />
            <Route path="Page3" element={<Page3/>} />
          </Route>
        </Routes>
      </Router>
    );
  }
}
```

下面运行npm start在浏览器中就可以看到如下效果：
![预览结果](https://moguxingqiu.oss-cn-hangzhou.aliyuncs.com/upload/config/blog/573dd6fd5a51e5771168215db9004f93.jpeg)
点击跳转到Page1后效果如下：
![预览结果](https://moguxingqiu.oss-cn-hangzhou.aliyuncs.com/upload/config/blog/2d6bdc082f0e97fb7f38a0ff4cef7dca.jpeg)
### ②、非嵌套路由：
更改App.js文件中代码：

```JavaScript
import React from 'react';
import { BrowserRouter as Router, Route, Routes } from 'react-router-dom';
import Home from './components/Home';
import Page1 from './components/Page1';
import Page2 from './components/Page2';
import Page3 from './components/Page3';
import './App.css';

export default class App extends React.Component {
  render () {
    return (
      <Router>
        <Routes>
          <Route path="/" exact element={<Home/>} />
          <Route path="/Page1" element={<Page1/>} />
          <Route path="/Page2" element={<Page2/>} />
          <Route path="Page3" element={<Page3/>} />
        </Routes>
      </Router>
    );
  }
}
```

现在点击跳转到Page1后效果如下：
![alt 预览结果](https://moguxingqiu.oss-cn-hangzhou.aliyuncs.com/upload/config/blog/a220e1a6a0845d00318e23ac4195ef0f.jpeg)
总结：
### 1、在react-router-dom v6中Route组件必须使用Routes嵌套，替换了v5中的Switch组件。
### 2、将原来的component改为element，必须以组件形式引入，而不是原来一个变量就行了。如{<Home/>}
### 3、嵌套路由必须在父级中添加Outlet组件，作为子组件的占位符，相当于vue-router中的router-view。
### 4、获取路由参数：
①、通过useParams获取动态路由的值。
②、通过useSearchParams获取查询字符串的值。

```JavaScript
import { useParams, useSearchParams  } from "react-router-dom";
```

```JavaScript
// 获取动态路由的值
  const params = useSearchParams();

  // 获取查询字符串的值
  const [searchParams, setSearchParams] = useSearchParams();
  console.log('searchParams', searchParams.get("name"))
```

上述searchParams变量，通过typeof查看是一个对象，但是不可直接点击查看其属性，可通过get方法访问指定属性的值。。
可通过setSearchParams修改查询字符串的值。如：

```JavaScript
setSearchParams({
	name: "心欲无痕"
})
```

### 5、可通过Link组件跳转，也可通过useNavigate方法跳转：

```JavaScript
import React from "react";
import { useNavigate } from "react-router-dom";

export default function Home () {
	const navigate = useNavigate ()
	const doJump = () => {
		navigate("/Page3")
	}
	return (
		<div>
			<div>This is Home！</div>
			<button onClick={doJump}>跳转到Page3页面</button>
		</div>
	)
}
```

### 6、跳转传参：
刚刚的doJump方法改成这样：

```JavaScript
const doJump = () => {
	navigate("/Page3", {state: {name: "张三", age: 28}})
}
```

然后在page3.js页面通过如下方式获取：

```JavaScript
import React from 'react';
import { useLocation } from 'react-router-dom';
 
export default function Page3 () {
	const location = useLocation()
	console.log('location', location.state)
	return(
		<div>
			<div>This is Page3!</div>
		</div>
	);
}
```

最后打印结果如下：
![预览结果](https://moguxingqiu.oss-cn-hangzhou.aliyuncs.com/upload/config/blog/fd6104d915c2ab89a22014c5c0731659.jpeg)

### 7、通过配置实现路由管理（useRoutes）。
在src目录下新建routes文件夹，在该文件夹下新建routes.js文件，代码如下：
```JavaScript
import React from 'react';
import { RouteObject } from "react-router-dom";
import Home from '../components/Home';
import Page1 from '../components/Page1';
import Page2 from '../components/Page2';
import Page3 from '../components/Page3';

const routes: RouteObject[] = [
	{
		path: "/",
		element: <Home />,
		children: [
			{
				path: "page1",
				element: <Page1 />
			},
			{
				path: "page2",
				element: <Page2 />
			},
			{
				path: "page3",
				element: <Page3 />
			}
		]
	}
]

export default routes
```
更改App.js文件代码：
```JavaScript
import React from 'react';
import { BrowserRouter as Router, useRoutes } from 'react-router-dom';
import routes from './routes/routes.js'
import './App.css';

export default function App () {
  const GetRoutes = () => {
    const route = useRoutes(routes)
    return route
  }
  return (
    <Router>
      <GetRoutes />
    </Router>
  )
}
```

ps：useRoutes的整个组件都必须放入<Router>当中

### 8、useEffect使用：
由于函数组件没有生命周期，可以使用useEffect来替代，他可以看做是componentDidMount、componentDidUpdate、componentWillUnmount这三个函数的组合。
```JavaScript
import React, { useState, useEffect } from 'react';
 
export default function Page2 () {
	const [count, setCount] = useState(0);

	useEffect(() => {
		getDatas ()
	})

	const getDatas = () => {
		let arr = [134,2345,355,46]
		console.log('arr', arr)
	}

	return(
		<div>
			<p>You clicked {count} times</p>
			<button onClick={() => setCount(count + 1)}>按钮</button>
			<div>This is Page2!</div>
		</div>
	);
}
```
这里，在页面初次加载完会执行getDatas方法，控制台会打印出数据，每次点击按钮更新count后，控制台会依次执行getDatas方法打印出数据，如果想通知React跳过对effect的调用，可以传一个空数组作为useEffect的第二个参数，这样就只会运行一次effect(仅在组件挂在和卸载时执行)。
```JavaScript
useEffect(() => {
	getDatas ()
}, [])
```
同样的，如果某些特定值在两次渲染之间没有发生变化，可以通知React跳过对effect的调用，比如只有count属性变化时才去调用effect，其他情况下不调用effect。
```JavaScript
useEffect(() => {
	getDatas ()
}, [count])
```