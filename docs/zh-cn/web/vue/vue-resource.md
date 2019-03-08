# 过滤器

之前我们学习了Vue的 [vue基本指令](http://tycoding.cn/2018/07/21/vue-1/#more) 进阶学习，我们需要了解Vue的过滤器：Vue.js允许你自定义过滤器，可被用作一些常见元素的格式化。过滤器可以用在两个地方：mustache插值`{{ val }}`和`v-bind`表达式。

用法： 
```
{{ 过滤器名称 | function }}
```

<!--more-->

**定义：**

Vue提供了两种方式创建过滤器：
* 1、全局过滤器
```html
Vue.filter('过滤器名称', function(){})
```

* 2、私有过滤器
```html
new Vue()({
	el: '',
	data: {},
	methods: {},
	filters: {
		过滤器名称: function(){}
	}
})
```

## 全局过滤器
```html
<h3>{{ msg | myFilters('时光') }}</h3>

Vue.filter('myFilters', function(data, pattern){
	if(pattern == '时光'){
		var old = data.substring(0,4);
		return old + ',时光路口。';
	}
});
```

**解释**
如上：Vue提供的全局过滤器，直接使用`Vue`调用，而不是定义在`Vue实例`中
* `Vue.filter()`中第一个参数是过滤器名称，第二个参数是`function(){}`
* `function(){}`中还有两个参数，第一个参数是原始的值，第二个参数是你想给过滤器方法传递的值。

## 私有过滤器
```html
<h3>{{ msg | myFilters('时光') }}</h3>

var vm2 = new Vue({
	el: '#app2',
	data: {
		msg: '银河街角！'
	},
	methods: {},
	filters: {

		myFilters: function(data, pattern) {
			if(pattern == '时光'){
				var old = data.substring(0,4);
				return old + ',时光路口。';
			}
		}
})
```
私有过滤器和全局过滤器用法基本相同，仅仅是作用于不同而已。

# 自定义指令

## 按键修饰符
在我们搜索商品时，在一些网站中我们直接回车后立即进行搜索，而不是点击搜索按钮才会搜索，那么这个功能怎么实现呢？

那么我们就需要了解Vue中提供的**按键修饰符**
用法： `@keyup.按键别名 = "要调用的方法名"`

**按键别名**
> .enter
> .tab
> .esc
> .delete
> ...

**实例**
```html
<input type="text" @keyup.enter="open">

methods: {
	open(){
		alert("弹出");
	}
}
```

**自定义按键修饰符**
如果Vue提供的按键修饰符不能满足你的需求，你也可以使用Vue提供的自定义按键修饰符来实现，因为每个键盘的按键都对应了一个键盘码值，比如F2对应的键盘码值是：113：

用法： 
```
<input type="text" @keyup.f2="open">

Vue.config.keyCodes.f2 = 113;	

methods: {
	open(){
		alert("弹出");
	}
}
```

## 获取文本焦点
获取文本焦点使用了focus属性，那么我们需要定义一个`v-focus`指令

```
Vue.directive('focus', {
	bind: function(el) {},
	inserted: function(el) {},
	updated: function(el) {}
});
```
如上，使用`Vue.directive()`实现定义全局指令，需要注意以下几点：
* 1、在directive()方法中包含两个参数：
		- 参数1：指令的名称，注意，在定义的时候指令名称不需要加v-前缀，但是在使用的时候需要加v-前缀。
	- 参数2：是一个对象，这个对象包含一些指令相关的函数，这些函数可以在特定的阶段，执行相关的操作。

**示例：**
```javascript
Vue.directive('focus', {

});

// 使用的时候使用： v-focus
```

* 2、在directive()函数的第二个参数中（对象）中又包含了三个实例方法：
	- bind: 当指令绑定到元素上的时候，会立即执行这个bind函数，只执行一次；但是需要知道元素绑定了这个指令，若涉及对DOM操作的，并不会立即执行，因为元素不会立即插入到DOM中。所以涉及对元素进行DOM相关操作的，不要定义到这个方法中。
	- inserted: 当元素插入到DOM的时候，会立即执行，并只触发一次。
	- updated: 当VNode更新的时候，会指定updated，可能触发多次。

**实例：**
```javascript
Vue.directive('focus', {
	bind: function(el) {
		el.focus()
	}
});

// 调用
<input type="text" v-focus />
```
如上，其中`bind`函数的第一个参数永远是`el`，它表示绑定的那个元素，是一个原生的JS对象；这里我们调用了JS的focus方法

### 钩子函数
指令定义函数提供了几个钩子函数（可选）：
> bind
> inserted
> update
> componentUpdated: 所在组件的VNode及其孩子的VNode全部更新的时候调用
> unbind: 只调用一次，指令与元素解除绑定时调用

**钩子函数参数**
在上面使用`directive()`函数的时候我们已经介绍了一些常用的钩子函数，那么既然是函数，就可能需要进行传参，那么为了实现钩子函数传参，Vue提供了几个参数属性来实现对钩子函数参数的一些操作：

* `el:` 指令所绑定的元素，可以用来直接操作DOM。
* `binding:` 一个对象，包含以下属性：
	* name: 指令名，不包含`v-`前缀
	* value: 指令的绑定值，如`v-focus="1 + 1"`，那么value=2。
	* expression: 绑定值的字符串形式，如`v-focus="1+1"`，那么experssion的值是`1+1`。
	* ...

* ...

**实例：**

实现在文本框中输入的数据颜色要为蓝色
```javascript
<input type="text" v-color="'blue'">

// 自定义设置颜色的指令
Vue.directive('color', {
	bind: function(el, binding){
		el.style.color = binding.value
	}
});
```
其中因为设计要获取值的操作，所以使用`binding`这个对象钩子函数参数来使用接收，那么：
1. 这个`el`就表示当前这个`<input>`文本框对象
2. 这个`bniding`就表示`v-color="'blue'"`指令传递的参数`blue`（因为使用`''`单引号即不是字符串）
2. `biding-value`就是获取到`v-color`指令绑定的参数值是：`blue`，通过`el.style.color`表示设置这个文本框样式中的颜色属性


### 定义私有指令
使用**私有指令**和**全局指令**的用法基本相同，我们参考上面讲过的**私有过滤器**和**全局过滤器**就能猜想到**私有指令**的用法：

**实例**
```javascript
<p v-fontsize="'50px'">私有指令</p>

var vm = new Vue({
	el: '',
	data: {},
	methods: {},
	filters: {},
	directives: {
		'fontsize': {
			bind: function(el, binding){
				el.style.fontSize = binding.value
			}
		}
	}
});
```

**自定义指令的简写形式**
对于仅仅使用`bind`和`update`钩子函数的操作，可以进行下列的简写形式：

```javascript
<p v-fontsize="'50px'">私有指令</p>

var vm = new Vue({
	el: '',
	data: {},
	methods: {},
	filters: {},
	directives: {
		'fontsize': function(el, binding){
			el.style.fontSize = binding.value
		}
	}
});
```

# 综合案例
实现将列表数据渲染到表格中，并实现添加功能案例（包含上面讲到的所有技术的**实例**）：
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title></title>
    <style type="text/css">
    table {
        min-height: 25px;
        line-height: 25px;
        text-align: center;
        border-collapse: collapse;
    }

    table,
    table tr th,
    table tr td {
        border: 1px solid #0094ff;
        padding: 11px;
    }
    </style>
</head>

<body>
    <div id="app">
        id:
        <input type="text" v-model="id" v-focus v-color="'blue'"> username:
        <input type="text" v-model="username" @keyup.enter="add">
        <input type="button" value="添加" @click="add">
        <br/>
        <br/>
        <table>
            <thead>
                <tr>
                    <th>id</th>
                    <th>name</th>
                    <th>date</th>
                    <th>fun</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="user in list" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.username}}</td>
                    <td>{{user.time | dataFormat('') }}</td>
                    <td><a href="#" @click="del(user.id)">删除</a></td>
                </tr>
            </tbody>
        </table>
        <p>未使用过滤器：{{ new Date() }}</p>
        <p>使用全局过滤器：{{ new Date() | dataFormat('') }}</p>
    </div>
    <div id="app2">
        使用私有过滤器：{{ dt | dataFormat('')}}
        <p v-fontsize="'50px'">私有指令</p>
    </div>
    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript">
    // 自定义文本框获取焦点指令
    // Vue.directive()定义全局指令，包含两个参数：
    // 参数1：指令的名称，注意，在定义的时候指令的名称不需要加v-前缀，但是在使用的时候需要加v-focus
    // 参数2：是一个对象，这个对象中包含一些指令相关的函数，这些函数可以在特定的阶段，执行相关的操作
    Vue.directive('focus', {
        // 注意，在下面的函数中，第一个参数永远是el，表示被绑定了指令的那个元素，这个el参数是一个原生JS对象
        bind: function(el) { //每当指令绑定到元素上的时候，会立即执行这个bind函数，只执行一次
            // 注意：在元素绑定了指令时，还没有插入到DOM中去，这时候调用focus方法是没有作用的，因为一个元素，只有插入到DOM之后，才能获取焦点
            // el.focus()
        },
        inserted: function(el) { // inserted表示元素插入到DOM中的时候，会执行,触发一次
            el.focus()
        },
        updated: function(el) { // 当VNode更新的是否，会执行updated，可触发多次
        }
    });

    // 自定义设置颜色的指令
    Vue.directive('color', {
        bind: function(el, binding) {
            el.style.color = binding.value
        }
    });

    // 自定义按键
    Vue.config.keyCodes.f2 = 113;

    // 演示私有过滤器
    var vm2 = new Vue({
        el: '#app2',
        data: {
            dt: new Date(),
        },
        methods: {
            open() {
                alert("弹出");
            }
        },
        filters: {
            dataFormat: function(data, pattern) {
                // 获取当前日期
                var dt = new Date(data);

                // 获取年月日
                var y = dt.getFullYear();
                var m = dt.getMonth() + 1;
                var d = dt.getDate();

                if (pattern.toLowerCase() == 'yyyy-mm-dd') {
                    return `${y}-${m}-${d}`;
                } else {
                    var hh = dt.getHours();
                    var mm = dt.getMinutes();
                    var ss = dt.getSeconds();

                    // es6中提供的 yyyy-mm-dd hh:mm:ss 的简写形式
                    return `${y}-${m}-${d} ${hh}:${mm}:${ss}` + '-->私有';
                }
            }
        },
        // 自定义私有指令（简写形式）
        directives: {
            'fontsize': function(el, binding) {
                el.style.fontSize = binding.value
            }
        }
    });

    // 全局过滤器
    Vue.filter('dataFormat', function(data, pattern) {
        // 获取当前日期
        var dt = new Date(data);

        // 获取年月日
        var y = dt.getFullYear();
        var m = dt.getMonth() + 1;
        var d = dt.getDate();

        if (pattern.toLowerCase() == 'yyyy-mm-dd') {
            return `${y}-${m}-${d}`;
        } else {
            var hh = dt.getHours();
            var mm = dt.getMinutes();
            var ss = dt.getSeconds();

            return `${y}-${m}-${d} ${hh}:${mm}:${ss}`;
        }
    });

    // 创建Vue实例
    var vm = new Vue({
        el: '#app',
        data: {
            id: '',
            username: '',
            list: [
                { id: 1, username: '涂陌', time: new Date() },
                { id: 2, username: 'TyCoding', time: new Date() }
            ]
        },
        methods: {
            add() {
                var user = { id: this.id, username: this.username, time: new Date() };
                this.list.push(user);
            },
            del(id) {
                // some()是操作数组的方法，作用是循环数组，并当return true是就终止循环
                // 其中的user理解为循环list元素的别名，i表示索引
                this.list.some((user, i) => {
                    if (user.id == id) {
                        this.list.splice(i, 1);
                        return true;
                    }
                })
            }
        }
    });
    </script>
</body>
</html>
```
![](https://tycoding.cn/2018/07/22/vue-2/1.png)


# Vue实例的生命周期

* 什么是声明周期：从Vue实例创建、运行、到销毁期间，伴随着发生的事件的过程成为生命周期。
* 生命周期钩子：就是声明周期事件的别名。
* 主要的声明周期函数分类

> 创建期间的声明周期函数：
> 	* beforeCreate: 实例刚在内存中被创建，此时，还没有初始化好data和methods属性。
> 	* created: 实例已经在内存中创建好，此时data和methods已经创建好，但还没有编译模板。
> 	* beforeMount: 此时已经完成了模板的编译，但是还没有挂载到页面上。
> 	* mounted: 此时，已经将编译好的模板，挂载到了页面指定的容器中。
> 
> 运行期间的声明周期函数:
> 	* beforeUpdate: 状态更新之前执行此函数，此时的data数据是最新的，但是此时还没有开始渲染DOM节点
> 	* updated: 实例更新完毕之后调用此函数，此时data中的状态值和界面上显示的数据都是最新的，界面已经被重新渲染好了。
> 
> 销毁期间的生命周期函数
> 	* beforeDestory: 实例销毁之前调用，在这一步，实例仍然可以使用。
> 	* destroyed: Vue实例销毁后调用，调用后，Vue实例指示的所有东西都会解除绑定，所有的事件监听器都会被移除，所所有的子实例也会被销毁。

## beforeCreate
此函数执行的时候，`data`和`methods`中的数据还没有初始化。

![](https://tycoding.cn/2018/07/22/vue-2/2.png)
![](https://tycoding.cn/2018/07/22/vue-2/3.png)

## created
此函数中，data和methods都已经初始化好了，如果需要调用`methods`中的方法或操作`data`中的值最早就在`created`函数中操作。

![](https://tycoding.cn/2018/07/22/vue-2/4.png)
![](https://tycoding.cn/2018/07/22/vue-2/5.png)

## beforeMount
此函数执行的时候，模板已经在内存中编译好了，但是尚未挂载到页面中去。

![](https://tycoding.cn/2018/07/22/vue-2/6.png)
![](https://tycoding.cn/2018/07/22/vue-2/7.png)

## mounted
只要执行完了`mounted`，表示整个Vue实例已经初始化完毕了，此时组件已经进入了运行阶段。

![](https://tycoding.cn/2018/07/22/vue-2/8.png)

## 图示
![](https://tycoding.cn/2018/07/22/vue-2/lifecycle.png)


# vue-resource实现请求提交
作为一个后端开发者，我们需要的数据都应该是从数据库中取出来的，目前JSP页面越来越不常用，而更常用HTML页面，那么就体现出来类似Vue这种框架的好处了。

下面我们就了解一下怎样使用Vue实现发送AJAX的请求：

## 配置
首先使用Vue实现发送AJAX请求，我们需要导入一个包：
```
vue-resource.js
```

**Methods**
```
this.$http.get('url', [options]).then(successCallback, errorCallback);

this.$http.post('url', [body], [options]).then(successCallback, errorCallback);
```
解释：
* `this`表示的是当前Vue实例对象，而`vue-resource.js`提供了`$http`属性用来调用其内置的请求方法，并且`vue-resource.js`是基于`vue.js`的。
* `options`是指可选的请求参数，就是你发送请求想要传递的参数。
* `then`可以实现发送完请求后，通过其获取请求**成功**响应的数据
* `then`中包含两个参数`successCallback`和`errorCallback`，这两个都是对象，我们可以通过其进行对相应数据的操作。


## 实例
```html
<button @click="getInfo">点击我</button>

var vm = new Vue({
	el: '#app',
	data: {},
	methods: {
		getInfo(){
			this.$http.get('url').then(result => {
				console.log(result.body);
			})
		}
	}
});
```
**解释：**
当我们请求成功后，可以通过`then`来获取请求成功响应的数据，而可以通过`.data`或`.body`来获取响应data，而我们通常使用`result.body`来获取具体响应的参数。注意其中的`result => {}`是ES6中的写法。

## post请求

**注意：** post请求常用于类似提交表单的功能，而对于提交表单，存在一个表单提交格式，默认是：`application/x-wwww-form-urlencoded` ；而通过Vue发起的post请求，默认没有表单格式，所以，有的服务器就处理不了。

那么我们可以通过post方法的第三个参数：`{ emulateJSON: true }`来设置提交内容类型为普通表单数据格式。

**实例**

```
this.$http.post('url', {}, { emulateJSON: true }).then(result => {
    console.log(result.body)
})
```

其他请求方法与上面的雷同，具体方法请参考官方文档。


# 请求接口根域名配置
由于我们个人的项目可能是部署到本地的Tomcat服务器上的，可能不会涉及请求接口的域名配置，那么我们先看一个案例：

发送post请求到服务器接口
```html
...

methods: {
	add(){
		this.$http.post('http://tycoding.cn/api/add', {}, {emulateJSON: true}).then(result => {
			
		});
	}
}
```
如上，当我们发送请求的时候，URL路径需要写上域名地址`http://tycoding.cn`，然后才是请求路径`/api/add`，那么我们每次发送ajax请求都会需要写这个域名地址，就会显得比较麻烦，所以`Vue-resource`给我们提供了一种设置默认请求**根域名**的配置：
```
Vue.http.options.root = 'http://tycoding.cn/';
```
如上，就是一个全局的请求根域名配置。

**注意**
仅了解了上面的配置可能请求还会404，那么我们需要知道：
> 如果我们通过了全局配置请求接口的根域名，那么每次发送HTTP请求时，请求的URL路径应该以相对域名开头，即前面不能带`/`：
> 	* 如果`this.$http.post('/xxx')`请求URL带了`/`，那么Vue就不会启用上面的全局请求根域名配置，就会404.
> 	* 如果前面不带`/`即：`this.$http.post('xxx')`，那么就会启用上面的全局请求根域名配置进行URL的拼接


## 全局配置表单提交格式选项
上面讲到了如果使用`post`请求提交**表单**，那么你应该指定`{emulateJSON: true}`参数，那么每次进行post请求都指定又会显得很麻烦，那么`vue-resource`也给我们提供了一个全局配置的方式：
```
Vue.http.options.emulateJSON = true;
```
这样我们就不需要再post请求中再配置第三个参数了：
```
methods: {
	add(){
		this.$http.post('http://tycoding.cn/api/add', {}).then(result => {
			
		});
	}
}
```

## 实例

请求后台并即时渲染表格数据的案例：
```html
<!-- html段 -->
<input type="text" v-model="username">
<input type="button" value="添加" @click="add">

<table>
	<tr>
		<th>编号</th>
		<th>用户名</th>
		<th>操作</th>
	</tr>
	<tbody>
		<tr v-for="user in list" :key="user.id">
			<td>{{user.id}}</td>
			<td>{{user.username}}</td>
			<td>
				<a href="#" @click.pervent="del(user.id)">删除</a>
			</td>
		</tr>
	</tbody>
</table>

<!-- javascript段 -->
<script type="text/javascript">
// 设置全局根域名
Vue.http.options.root = 'http://tycoding.cn/';

//设置全局表单提交格式
Vue.http.options.emulateJSON = true;

// 实例化Vue
new Vue({
	el: '',
	data: {
		username: '',
		list: []
	},
	created: {
		// 因为进入列表页面就需要在列表中显示出数据，那么就需要实现加载页面时自动加载findAll方法
		// 而之前我们已经知道了，Vue的声明周期中，最早可以操作methods和data中的数据的阶段是：created生命周期函数阶段。
		// 那么在这里调用findAll方法即可
		this.findAll();
	},
	methods: {
		// 查询所有列表数据
		findAll(){
			this.$http.get('api/findAll').then(result => {
				this.list = result.body;
			}),
		},

		// 添加功能
		add(){
			this.$http.post('api/add', {username: this.username}).then(result => {
				if(result.body.status == 0){
					// 如果状态码为0就表示请求成功，这个状态码的值根据实际定
					// 请求成功，即添加了一条新的数据，那么需要重新刷新列表（不然新数据不能及时的更新到页面上）
					this.findAll()
				}else{
					alert('添加失败');
				}
			})
		},

		//删除功能
		del(id){
			this.$http.get('api/del' + id).then(result => {
				if(result.body.status == 0){
					//请求成功
					//刷新列表
					this.findAll();
				}else{
					alert('删除失败');
				}
			})
		}
	}
});
</script>
```
如上我们已经完成了常见的几个功能，后面我们将会介绍基于`SpringMVC`框架，实现与Vue整合并重写增删改查功能。

**注意：**

* 1、首先我们需要配置根域名，且具体的AJAX请求URL不能添加'/'；如果是基于本地的Tomcat服务器的项目，可能不需要配置根域名，具体视情况而定

* 2、实现查询所有列表数据功能，**思路**是：1、发送AJAX请求数据；2、将响应的数据赋值给data中的list集合`this.list = result.body`。注意响应数据是存放到body中的，具体请F12查看浏览器请求头信息和响应头信息。

* 3、上面获取了数据库中的列表数据，我们需要渲染到页面上，那么点击进入列表页面，列表页面中应该立即显示数据库中的所有数据，即`findAll`方法应该在初始化页面的同时自动去调用，并将数据赋值给list列表。而我们之前讲过操作methods和data中参数的最早时机是`created`声明周期函数阶段，那么我们直接在`created`函数中调用`findAll`方法即可实现自动加载。

* 4、添加功能的**思路**：1、在`data`中先声明需要添加的参数；2、在表单中用`v-model`绑定需要添加的参数；3、点击添加功能按钮，绑定`@click`事件，在`methods`中写对应的方法；4、发送AJAX请求，并在URL中拼接需要添加的数据（通过this.username）获取绑定的参数；5、如果是post请求，还需要设置表单提交格式`{emulateJSON: true}`，而我们使用了全局配置就不需要再在post参数中指定了；6、如果添加成功，就调用`findAll`方法重新刷新列表

* 6、删除功能需要在绑定`@click`事件的时候将`id`传入。并且我们需要使用`@click.pervent`来阻止`<a>`标签的默认跳转。

* 7、上面仅是提供演示，具体操作由实际情况而定.
