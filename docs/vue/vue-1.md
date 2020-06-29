---
title: 初识Vue
tags: vue
top: 25
comments: false
date: 2018-07-21 20:58:12
---

**什么是Vue.js**

* Vue.js是目前很火的前端框架；与Angular.js、React.js并称为前端三大主流框架。
* Vue.js是一套构建用户界面的框架，只关注视图层（MVC中的V层）；它易于上手，便于和第三方库或既有项目整合，
* 在Vue中，一个核心的概念就是减少对DOM元素的操作，让程序员更多的去关注业务逻辑。

<!-- more -->

# 后端的MVC和前端的MVVM之间的区别
* MVC是后端的分层开发概念
* MVVM是前端视图层的概念，主要关注于：视图层分离；也就是说：MVV将前端分为三个部分Model、View、VM（ViewModel）
	1. Model： 页面需要展示的数据
	2. View: 视图、HTML
	3. VM: 数据（Model）和视图（View）之间的调度者

**图解**
![](http://cdn.tycoding.cn/20200629094442.png)


# 入门案例
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>

<!-- Vue实例的控制区域 -->
<div id="app">
	<!-- 插值表达式 -->
	<p>{{ msg }}</p>
</div>

<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
	// 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
	var vm = new Vue({
		el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
		data: {		// data中存放的是el中需要的数据
			msg: 'Hello Vue!'
		}
	});
</script>

</body>
</html>
```

**解释：**
* 1、首先需要引入Vue.js
* 2、声明Vue实例的控制区域，我们可以放在指定div中，或者body体；控制区域下使用的Vue语法就会被Vue识别到
* 3、创建Vue实例，这个实例其实就是MVVM中的vm调度者
* 4、`el:` 表示当时Vue实例的控制区域；`data:` 存放当前Vue实例中所需的Model（数据）。其中的msg就是一个Vue的元素。
* 5、在指定Vue实例区域下，展示我们已经声明的元素`msg`，使用：`{{msg}}`即可展示出来（其中msg是在Vue中声明的元素，如果未声明会报错）


# 常用指令

## v-cloak
在入门案例中我们初步了解到，在View层我们可以通过插值表达式的方式展示Vue绑定的数据；但是存在一个问题，当网络很慢的情况下，即在`vue.js`还没有加载完毕时，视图层不会将`{{val}}`识别为Vue中的插值表达式，而是作为字符串显示到页面上。

为了解决这个问题，Vue.js提供了`v-cloak`指令，可以解决插值表达式闪烁的问题。
```html
<p v-cloak>{{ msg }}</p>
```

**扩展**
解决插值表达式闪烁问题，除了`v-cloak`指令，Vue还提供了`v-text`指令。
```html
<p v-text="msg"></p>
```

**v-cloak与v-text的区别**
`v-text`默认会覆盖掉元素中原有的内容
`v-cloak`不会覆盖掉原有的内容

![](http://cdn.tycoding.cn/20200629094450.png)

## v-bind
在上面，我们了解了`v-text`输出Vue元素，但是如果我们需要显示的数据是一段`HTML`代码，`v-text`和`{{val}}`都将无能为力，
为此，Vue提供了专门渲染HTML数据的指令：`v-html`

![](http://cdn.tycoding.cn/20200629094454.png)

上面我们学习的指令都是Vue内置的指令，那么在Vue内置的指令中显示Vue绑定的变量，这当然没毛病；但是如果直接在HTML属性中使用Vue绑定的指令（不是用`{{val}}`）这样可以吗？
```html
<input type="button" value="msg"/>
```
回答当然是不行的，因为在HTML属性中直接使用Vue绑定的变量，HTML并不能将其识别为其引用的是Vue中的元素，而是作为一个字符串输出。

为了解决上述问题，Vue提供了`v-bind:`指令来绑定一些HTML属性：
```html
<input type="button" v-bind:value="msg">
```
如上，被`v-bind:`绑定的属性，其元素不再是字符串，而是被识别为Vue的绑定的变量（同样这个变量必须声明了）。另外`v-bind:`还有一个简易写法：`:`。
```html
<!-- Vue实例的控制区域 -->
<div id="app">
    <input type="button" value="msg" />
    <input type="button" v-bind:value="msg">
    <input type="button" :value="msg">
</div>

<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
    // 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
    var vm = new Vue({
        el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
        data: { // data中存放的是el中需要的数据
            msg: '戳我'
        }
    });
</script>
```
![](http://cdn.tycoding.cn/20200629094459.png)

## v-on
Vue提供了事件绑定机制的指令：`v-on:`；用其我们可以用来绑定一些常见的触发事件：click、mouseover ...
```html
<!-- Vue实例的控制区域 -->
<div id="app">
    <input type="button" :value="msg" v-on:click="show">
    <input type="button" :value="msg" v-on:mouseover="show">
</div>

<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
    // 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
    var vm = new Vue({
        el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
        data: { // data中存放的是el中需要的数据
            msg: '戳我'
        },
        methods: {
        	show: function(){
        		alert("hello");
        	}
        }
    });
</script>
```
其中`methods`是Vue内置的**对象**，用于存放一些自定义的方法函数

**拓展**
使用js内置的函数`setInterval`(定时器)，实现跑马灯效果：

```html
<!-- Vue实例的控制区域 -->
<div id="app">
    <input type="button" value="开始" @click="action">
    <input type="button" value="停止" @click="stop">
    <p>{{msg}}</p>
</div>

<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
    // 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
    var vm = new Vue({
        el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
        data: { // data中存放的是el中需要的数据
            msg: '嘻嘻，哈哈',
            intervalId: null
        },
        methods: {
        	action(){
        		if(this.intervalId != null) return;

        		// 定时器
        		this.intervalId = setInterval(() => {
        			// 截取首字符
        			var start = this.msg.substring(0, 1);
        			// 截取第一个字符后的所有字符
        			var end = this.msg.substring(1);
        			// 将后面的字符拼接到前面，实现循环的效果
        			this.msg = end + start;
        		},400)
        	},
        	stop(){
	        	// 停止定时器
        		clearInterval(this.intervalId)
        		// 每次清除定时器后需要将intervalId重新设置为null
        		this.intervalId = null;
        	}
        }
    });
</script>
```

**解释：**
1、`v-on:`也有简写形式：`@`，用法如上。
2、在视图层取VM中的数据我们可以使用`{{val}}`或一些内置指令；而在VM实例内部获取定义的其他变量或方法等，使用：`this.数据属性名`（其中的this表示当前VM实例对象）。
3、`methodName:function(){}`在ES6中有一个简便的写法：`methodName(){}`。
4、`setInterval()`和`clearInterval()`是js中内置的函数，用法如上。
5、正常我们调用函数会写：`name(function(){})`，而ES6也提供了一个方式：`methodName(() => {})`，这种用法的好处就解决了`this`指向问题，因为如果元素定义在了函数内部，那么其中的`this`就表示当前函数的对象，如果我们需要使用外部的对象，除了在外部全局定义一个对象，一个简单的方式就是使用ES6提供的`=>`。

![](http://cdn.tycoding.cn/20200629094507.png)

## 事件修饰符
> .stop 阻止冒泡
> .prevent 阻止默认事件
> .capture 添加时间侦听器时使用时间捕获模式
> .self 只当事件在该元素本身（比如不是子元素）触发时触发回调
> .once 事件只触发一次

用法：
```html
<!-- Vue实例的控制区域 -->
<div id="app">
    <div @click="divClick">
        <input type="button" value="戳我" @click.stop="btnClick">
        <input type="button" value="戳我" @click.prevent="btnClick">
    </div>
</div>
<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
	// 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
	var vm = new Vue({
	    el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
	    data: { // data中存放的是el中需要的数据
	        msg: '嘻嘻，哈哈',
	    },
	    methods: {
	        divClick() {
	        	console.log("这是div的点击事件");
	        },
	        btnClick() {
	        	console.log("这是btn的点击事件");
	        }
	    }
	});
</script>
```

## v-model

* 唯一的双向绑定指令：`v-model`
* 单向绑定指令：`v-bing`

**实例**
```html
<!-- Vue实例的控制区域 -->
<div id="app">
    <input type="text" v-model="msg">
    <p>{{msg}}</p>
</div>
<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
	// 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
	var vm = new Vue({
	    el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
	    data: { // data中存放的是el中需要的数据
	        msg: 'hello!',
	    },
	    methods: {

	    }
	});
</script>
```
![](http://cdn.tycoding.cn/20200629094513.png)

## vue中的样式

### 外联样式

* 数组
`<h2 :class="['italic','color']">涂陌</h2>`
其中的italic、color是自定义的类名，需在外部定义CSS样式

* 数组中嵌套对象
`<h2 :class="['italic',{'color': flag}]">涂陌</h2>`
其中的flag是Vue绑定的变量，在`data`进行声明

* 直接使用对象
`<h2 :class="{italic:true, color:flag}">涂陌</h2>`

**实例**
```html
<style>
	.italic {
	    font-style: italic;
	}

	.color {
	    color: skyblue;
	}
</style>
<!-- Vue实例的控制区域 -->
<div id="app">
	<h2 :class="['italic','color']">涂陌</h2>
    <h2 :class="['italic', {'color':flag}]">涂陌</h2>
    <h2 :class="{italic:false, color:flag}">涂陌</h2>
</div>
<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
	// 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
	var vm = new Vue({
	    el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
	    data: { // data中存放的是el中需要的数据
	        flag: true
	    },
	    methods: {

	    }
	});
</script>
```
![](http://cdn.tycoding.cn/20200629094518.png)

### 内联样式

* 将样式对象定义到`data`中，并在`:style`中引用
```html
<h2 :style="styleObj">涂陌</h2>

data: {
	styleObj: { 'color': 'red', 'font-weight': '200px'}
}
```

* 在`:style`中通过数组，引用多个`data`上的样式对象
```html
<h2 :style="[styleObj, styleObj2]">涂陌</h2>

data: {
	styleObj: { 'color': 'red', 'font-weight': '200px'},
	styleObj2: { 'font-style': 'italic' }
}
```

**实例**
```html
<!-- Vue实例的控制区域 -->
<div id="app">
    <h2 :style="styleObj">涂陌</h2>
    <h2 :style="[styleObj, styleObj2]">涂陌</h2>
</div>
<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
	// 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
	var vm = new Vue({
	    el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
	    data: { // data中存放的是el中需要的数据
	        styleObj: { 'color': 'red', 'font-weight': '200px' },
	        styleObj2: { 'font-style': 'italic' }
	    },
	    methods: {

	    }
	});
</script>
```
![](http://cdn.tycoding.cn/20200629094523.png)


## v-for
Vue提供了遍历集合、数组的指令：`v-for`；用法: `v-for="别名 in 集合名"`

### 迭代数组
```html
<p v-for="item, i in list">索引：{{i}} --- 值：{{item}}</p>

data: {
	list: [1,2,3,4]
}
```
其中的`i`是迭代得到的别名，可写可不写，但是必须是在迭代元素别名的后面定义

### 迭代对象数组
```html
<p v-for="item in list2">id: {{item.id}} --- name: {{item.name}}</p>

data: {
	list2: [
        	{ id:1, name: '嘻嘻' },
        	{ id:2, name: '哈哈' }
        ],
}
```
迭代对象数组，通过	`{{迭代元素别名.属性名}}`的方式，这个属性名就是对象数组中定义的元素属性名

### 迭代对象
```html
<p v-for="(val, key) in user">键: {{key}} --- 值: {{val}}</p>

data: {
	user: {
        	id: 1,
        	name: '涂陌'
        }
}
```
迭代对象，迭代得到的是对象的`value`值和`key`值，注意得到的第一个是value值，第二个是key值，与我们定义的对象属性顺序是刚好相反的。

### 实例
```html
<!-- Vue实例的控制区域 -->
<div id="app">
    <p v-for="item, i in list">索引：{{i}} --- 值：{{item}}</p>
    <p v-for="item in list2">id: {{item.id}} --- name: {{item.name}}</p>
    <p v-for="(val, key) in user">键: {{key}} --- 值: {{val}}</p>
</div>
<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
	// 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
	var vm = new Vue({
	    el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
	    data: { // data中存放的是el中需要的数据
	        list: [1,2,3,4], 
	        list2: [
	        	{ id:1, name: '嘻嘻' },
	        	{ id:2, name: '哈哈' }
	        ],
	        user: {
	        	id: 1,
	        	name: '涂陌'
	        }
	    },
	    methods: {

	    }
	});
</script>
```
![](http://cdn.tycoding.cn/20200629094532.png)

### 注意
在vue2.0+版本里，当使用`v-for`渲染数据，必须制定对应的`key`值（这里的key是一个属性，不是前面迭代的key值）。

**用法:**
```html
<p v-for="item in user" :key="item.id">
```

其中`:key`就说明了key属性必须是通过`v-bind`绑定的元素，而`:key=""`中指定的值必须是`string/number`类型的值，比如此处使用的是`item.id`中ID是number值，并且是唯一的。


**目的：**
避免迭代元素时，为循环元素绑定的是列表中的第几个元素（指定位置），而不是指定的某个元素（指定身份）。

## v-show和v-if
Vue提供了两个指令来实现元素显示状态的切换：`v-if` `v-show`

**区别**
* `v-if`的特点：每次都会重新删除和创建元素，具有较高的切换性能消耗（因为每次执行都要进行删除和创建元素）。

* `v-show`的特点：每次不会重建进行DOM的删除和创建操作，只是切换了元素的`display:none`样式，具有较高的初识渲染消耗（即每次都只是将元素隐藏了，并没有真正的删除掉）。


**实例**
```html
<!-- Vue实例的控制区域 -->
<div id="app">
    <input type="button" @click="flag=!flag" value="toggle">

    <h3 v-if="flag">这是v-if控制的元素</h3>
    <h3 v-show="flag">这是v-show控制的元素</h3>
</div>
<!-- js部分 -->
<script src="../lib/vue.js"></script>
<script>
    // 创建Vue实例，这个new出来的对象就是MVVM中的vm调度者
    var vm = new Vue({
        el: '#app', // element的简写，表示我们当前new的这个Vue实例的区域
        data: { // data中存放的是el中需要的数据
            flag: false
        },
        methods: {

        }
    });
</script>
```
![](http://cdn.tycoding.cn/20200629094538.png)

![](http://cdn.tycoding.cn/20200629094542.png)





<br/>

# 交流

QQGroup：671017003   

WeChatGroup:  关注公众号查看

# 联系

- [Blog@TyCoding's blog](http://www.tycoding.cn)
- [GitHub@TyCoding](https://github.com/TyCoding)
- [ZhiHu@TyCoding](https://www.zhihu.com/people/tomo-83-82/activities)

