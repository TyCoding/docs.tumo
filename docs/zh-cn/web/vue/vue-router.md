# Vue路由

**什么是Vue路由？**

* **后端路由：**即请求的URL地址都对应后端的接口，请求URL响应对应的服务器的资源。

* **前段路由：**对于单页面程序来说，主要通过URL中的hash(#号)来实现不同页面之间的切换，同时，hash有一个特点：HTTP请求中不会包含hash相关的内容；所以单页面程序中的页面跳转主要通过hash实现。

<!--more-->

## 入门

### 导入类库

首先需要导入vue-router组件类库：
```
<script type="text/javascript" src="../lib/vue-router.js"></script>>
```

当导入类库后，window全局对象中就存在了一个路由的构造函数： `VueRouter`；就像导入Vue类库后存在一个`Vue`的构造函数一样，我们能够通过`new VueRouter`的方式实例化路由对象。

如果你使用了`new VueRouter({});`构造函数来实例化了一个路由对象，你会发现浏览器路径中会出现`#/`路径，这个路径就是前面讲到的URL中的hash，他并不会向后端发送任何请求，而仅仅是作页面跳转，如果你`#/`后拼接了一个不存在的路径，自然也不会进行跳转，页面也不会发送任何请求。

**实例：**

![](https://tycoding.cn/2018/07/25/vue-4/1.png)
![](https://tycoding.cn/2018/07/25/vue-4/2.png)


### 基本使用

上面我们讲到了使用`new VueRouter({})`的方式实例化一个路由对象，其中包含几个参数：
```javascript
<!-- HTML -->
<div id="app">
	
	<!-- 给路由对象创建一个容器，包裹在父组件`app`内
		相当于一个占位符，路由规则匹配到的组件内容就会展示到`<router-view></router-view>`中
	-->
	<router-view></router-view>
</div>

<!-- =================分割线===================== -->

<!-- JavaScript -->
//创建login组件
var login = {
	template: 'login组件'
}

//创建register组件
var register = {
	template: 'register组件'
}

// Router实例
var 路由对象名称 = new VueRouter({
	routes: [
		{ path: '/监听URL路径', component: login(组件名称) },
		{ path: '/监听URL路径', component: register(组件名称) }
	]
});

// Vue实例
var vm = new Vue({
	el: 'app',
	data: {},
	methods: {},
	router: 路由对象名称   
});
```

**解释：**

* 1、`routes` 表示这个路由对象中的 **路由匹配规则**，可以存在多个规则（**注意：**这里是`routes`而不是`routers`）
	
* 2、属性一：`path` 表示监听哪个路由连接地址，即你想跳转都哪个路径上，应在这里注册实现跳转到对应的组件上
	
* 3、属性二：`component` 表示如果路由匹配了前面的`path`，则展示component属性对应的那个组件
	
* 4、`component`的属性值必须是一个组件的模板对象，不能是组件的引用名称
	
* 5、经过`1-4`的步骤基本完成了路由对象的创建，下面要将这个路由对象注入到Vue实例中，使用`router: 组件对象名称`实现
	
* 6、创建对应需要监听的组件，如上我们创建了`login`和`register`组件，与之前讲的不同是这里是一个`var 组件名称`其值是一个组件对象，和之前的`Vue.component`方式相似，但是这里仅仅是一个组件对象，并没有注册到Vue实例中，因为没有组件名称，所以不能在HTML中使用`<login></login>` ，注意这里的`login`是组件对象的名称。


**步骤：**

* 1、创建router实例`new VueRouter`，完成相关属性的定义；

* 2、将这个路由对象注册到Vue实例中，使用`router: 路由对象名称`的方式；

* 3、创建第一步中定义的组件名称对应的组件，直接在`<script>`中定义var组件对象名称即可，在`template`中定义具体的HTML视图，或是通过`template: '#id'`引用外部视图也行。

* 4、在Vue实例控制域`app`中，创建`<router-view></router-view>`，相当于router容器，你想在页面上展示几个组件就应该在页面中创建几个容器。


如上，我们可以写具体的跳转链接了：
```
<div id="app>
	<a href="#/login">登录</a>
	<a href="#/register">注册</a>
</div>
```
如上，当我们点击登录或注册，Vue-router就会监听都对应的URL地址，然后在`path`规则中刚好匹配到规则`login`，那么就会跳转到对应的组件`component: login`的login组件中。
可能你会疑惑了，为什么这里的`href`需要写为`#/login`而不是`/login`或`login`，你尝试一下就知道了，因为vue-router监听URL地址是基于`hash`的，不加`#/`就会找不到路径。

如果你觉得每次都加`#/`麻烦的话，Vue-router页提供了一个Tag:`<router-link to="URL地址"></router-link>`，其在浏览器中会被解析为`<a>`标签。


**实例**
```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
    <title></title>
</head>
<body>
	<div id="app">
		<h1>父组件</h1>

		<a href="#/login">登录</a>
		<router-link to="register">注册</router-link>

		<router-view></router-view>
	</div>
    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript" src="../lib/vue-router.js"></script>
    <script type="text/javascript">

    	var login = {
    		template: '<h2>登录组件</h2>'
    	}

    	var register = {
    		template: '<h2>注册组件</h2>'
    	}

		var routerObj = new VueRouter({
			routes: [
				{ path: '/login', component: login },
				{ path: '/register', component: register }
			]
		});

		var vm = new Vue({
			el: '#app',
			data: {},
			methods: {},
			router: routerObj
		});
    </script>
</body>
</html>
```
但是上面的实例中，我们发现默认进入的根路径中仅有一个父组件名称，一般我们的登录页面应该直接显示登录框，所以vue-router提供了重定向的动能`{path: '', redirect: ''}`，即在`router: []`中监听根路径，如果监听都访问的是根路径就重定向到登录URL就好了。
```
var routerObj = new VueRouter({
	routes: [
		{ path: '/', redirect: '/login' },
		{ path: '/login', component: login },
		{ path: '/register', component: register }
	]
});
```

## 路由参数传递

在进行页面跳转，即路由的时候，我们可能需要在发送URL时传递一些参数，常见的就如`http://tycoding.cn/api?id=1&name='涂陌'`这种格式。

那么在URL中传递的参数，vue-router提供了一种获取方式：`this.$route`。

那么什么时候能获取到传递的参数呢？ 回顾前面讲到的Vue声明周期函数，那么在自定义组件中自然也存在生命周期函数，所以最早操作组件`data`和`methods`中数据的阶段就是`created`这个声明周期函数的阶段。

实例：

![](https://tycoding.cn/2018/07/25/vue-4/3.png)

![](https://tycoding.cn/2018/07/25/vue-4/4.png)

上面打印的值中，我们能看到，我们再`VueRouter`中创建的`path`匹配规则，实际在HTML中会被渲染为相关的**正则表达式**，来实现路径的匹配。
其次，我们还能发现，在URL中拼接的参数`id`在`this.$route`对象的`query`属性中，我们通过`this.$route.query.id`即可获得传递的id值：`2`。



## 路由嵌套

路由嵌套，顾名思义即在父级路由内部存在子路由。例如：

> 根路径：`http://tycoding.cn/`
> 父级路由地址：`http://tycoding.cn/api`
> 子级路由地址：`http://tycoding.cn/api/login`


**实例：**
```html
<!DOCTYPE html>
<html>
<head>
	<meta charset="UTF-8">
    <title></title>
</head>
<body>
	<div id="app">
		<router-link to="/account">Account</router-link>

		<router-view></router-view>
	</div>

	<template id="tmp">
		<div>
			<h1>这是Account组件</h1>

			<router-link to="/account/login">登录</router-link>
			<router-link to="/account/register">注册</router-link>

			<router-view></router-view>
		</div>
	</template>
    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript" src="../lib/vue-router.js"></script>
    <script type="text/javascript">

    	var account = {
    		template: '#tmp'
    	}

    	var login = {
    		template: '<h2>登录组件</h2>',
    	}

    	var register = {
    		template: '<h2>注册组件</h2>'
    	}

		var routerObj = new VueRouter({
			routes: [
				{ 
					path: '/account', 
					component: account,
					children: [
						{ path: 'login', component: login },
						{ path: 'register', component: register }
					]
			    },
			]
		});

		var vm = new Vue({
			el: '#app',
			data: {},
			methods: {},
			router: routerObj
		});
    </script>
</body>
</html>
```

如上，我们使用了`routes: []`中的另外一个属性：`children`，顾名思义就是表示这个父规则`/account`下存在一些子规则，且在URL中应该体现出来：

![](https://tycoding.cn/2018/07/25/vue-4/5.png)

**注意：**

在`children`中定义的子组件的`path`规则不能加`/`，即如上的，直接写`path: login`即可，这样请求`account/login`地址时，vue-router会找`/account`规则下的`login`规则，且不加`/`vue-router才会自动将`login`视为`account`下的子路径，并自动拼接`account/`，否则不会自动拼接，那么也无法完成路由的嵌套。


## 命名视图

我们常见的后台开发页面，经常遇到上、左、中的布局方式；那么以前我们可能使用iframe实现页面间的跳转，但是现在我们学习的路由要比其更加的方便好用。

**命名视图**的思想就是为每一个页面展示的视图都起一个名字，目的是为了为每个`<router-view></router-view>`容器刚好匹配一个指定的视图。使用方式：
```
<router-view></router-view>
<router-view name="left"></router-view>
<router-view name="main"></router-view>

routes: [
	{ 
		path: '/', components: {
			'default': header
			'left': left
			'main': main
		}
	}
]
```

**解释：**

其中的`path`是根路径`/`，而使用`components`代替之前的`component`，目的就是可匹配其下的多个规则；`default`表示默认的视图组件是`header`这个组件，即会匹配到第一个`<router-view>`视图容器中；下面的两个组件会根据name名称需要对应的组件。




实现上、左、中的布局：

**思路：**

1、我们需要创建三个组件，名称分别为：`header`、`left`、`main`；并且在`app`中创建三个`<router-view></router-view>`路由容器。

2、采用**命名视图**的方式为每个视图都起一个名字：`<router-view name="left"></router-view>` ...

3、注册路由实例。


**实例：**

![](https://tycoding.cn/2018/07/25/vue-4/6.png)

![](https://tycoding.cn/2018/07/25/vue-4/7.png)
