# Vue创建组件

## 什么是组件
什么是组件？组件的出现，就是为了拆分Vue实例的代码量的，能够让我们以不同的组件，来划分不同的功能模块，将来我们需要什么样地方功能，就可以去调用对应的组件即可。

组件化和模块化的区别：

* 模块化：是从代码逻辑的角度进行划分的，方便代码分层开发，保证每个功能模块的只能单一；
* 组件化：是从UI界面的角度进行划分的；前端的组件化，方便了UI组件的重用；



<!--more-->



## 定义Vue组件

### 方式一
1、使用`Vue.extend`来创建全局的Vue组件
2、通过`template`属性来指定组件要展示的HTML结构
3、通过`Vue.component()`定义全局组件
4、在HTML指定位置通过Tag标签的形式应用你的组件，组件的名称即是tag标签的名称


```javascript
<div id="app">
	<!-- 组件的名称即为对应tag标签的名称 -->
	<mycom1></mycom1>
</div>

<script>
	// 创建组件
	var com1 = Vue.extend({
		template: '<h3>这是使用Vue.extend 创建的组件</h3>'
	});

	// 使用Vue.component('组件的名称', 创建出来的组件模板对象)，定义全局组件
	Vue.component('mycom1', com1);
</script>
```

**注意**

* 用`Vue.component()`定义组件名称的时候有两种命名方式：1、驼峰命名：`Xxx`；2、`xxx`。注意，使用驼峰命名时，tag标签不能使用驼峰名称，应将相应的大写字母替换为`-`加小写字母，例如：`Vue.component('myCom', com1)`，那么tag标签应为：`<my-com></my-com>`


### 方式二
```html
<!-- 组件的名称即为对应tag标签的名称 -->
<mycom1></mycom1>

<script>

	// 使用Vue.component('组件的名称', 创建出来的组件模板对象)，定义全局组件
	Vue.component('mycom1', Vue.extend({
		template: '<h3>这是使用Vue.extend 创建的组件</h3>'
	}));
</script>
```
方式二就像是方式一的简化版，但是注意：
在`template`中定义的内容只能存在一个根节点元素，即上面例子中`template`中不能出现和`<h3>`平级的标签。
比如下面这种写法就是**错误的**
```
Vue.component('mycom1', Vue.extend({
	template: '<h3>这是使用Vue.extend 创建的组件</h3><span></span>'
}));
```
解决办法就是，外层嵌套一个根标签就行了：
```
Vue.component('mycom1', Vue.extend({
	template: '<div><h3>这是使用Vue.extend 创建的组件</h3><span></span></div>'
}));
```


### 方式三
1、JavaScript中仍使用`Vue.component`来定义全局组件，和**方式二**相似，但是里面不再是`template: 'HTML结构'`了，而是引用一个外部标签的`id`值，即`template:'id'`。

2、在被Vue实例控制的`app`**外面**，定义`<template id="id">你的HTML结构</template>>` 

这种方式的好处就是是就组件代码都是定义在HTML结构中的，有智能代码提示；而JavaScript中定义组件仅是写一个引用。

实例：
```
<div id="app">
	<!-- 页面引用 -->
</div>


<template id="tmp">
	<!-- 比如仅存在一个根节点元素 -->
	<div>
		<h1></h1>>
		<h2></h2>
	</div>
</template>

//JavaScript部分
Vue.component('mycom', {
	template: '#tmp'
});
```

### 定义私有组件
除了上面讲到的定义全局组件，我们也可以定义私有组件，使用`components: {}`函数。

如：
```
<div id="app">
    <tem></tem>
</div>
<template id="tmp">
    <h3>这是私有组件</h3>
</template>
<script type="text/javascript" src="../lib/vue.js"></script>
<script type="text/javascript">
new Vue({
    el: '#app',
    data: {},
    methods: {},
    components: {
        tem: {
            template: '#tmp'
        }
    }

});
</script>
```

## 组件元素

### 组件的data
在组件中，同样可以有自己的data数据，但是用法和Vue实例中的data用法有所不同：

1. 组件中的data必须是一个方法，即
```
data: function(){ }
```

2. 组件中的data方法必须返回一个Object对象
```
data: function(){
	return object
}
```

实例：
```
<div id="app">
    <mycom1></mycom1>
</div>
<script type="text/javascript" src="../lib/vue.js"></script>
<script type="text/javascript">
Vue.component('mycom1', Vue.extend({
	data: function(){
		return {
			msg: '这是组件的data'
		}
	},
	template: '<h3>这是使用Vue.extend 创建的组件-- {{msg}}</h3>'
}));

new Vue({
    el: '#app',
    data: {},
    methods: {},
    components: {
    }
});
</script>
```

### 组件的切换
在遇到登录注册表单时，通常我们需要对两个按钮进行切换实现显示不同的表单，那么Vue中的组件切换正符合了这个功能要求。

#### 方式一

可以使用Vue提供的`v-if`和`v-else`来实现**两个**组件间的切换，但是，仅支持切换两个组件

实例：
```html
<div id="app">
    <a href="#" @click.prevent="flag=true">登录</a>
    <a href="#" @click.prevent="flag=false">注册</a>

    <login v-if="flag"></login>
    <register v-else="flag"></register>
</div>
<template id="login">
    <h3>这是登录表单</h3>
</template>
<template id="register">
	<h3>这是注册表单</h3>
</template>
<script type="text/javascript" src="../lib/vue.js"></script>
<script type="text/javascript">
// 创建登录组件
Vue.component('login', {
	template: '#login'
});

// 创建注册组件
Vue.component('register', {
	template: '#register'
})

new Vue({
    el: '#app',
    data: {
    	flag: true
    },
    methods: {}

});
</script>
```
定义`flag`参数，当flag=true就显示组件，当flar=false就隐藏组件


#### 方式二

Vue提供了`component`来展示对应名称的组件。其中`component`是一个占位符，`:is`属性，可以用来指定要展示的组件名称

实例：
```html
<div id="app">
    <a href="#" @click.prevent="comName='login'">登录</a>
    <a href="#" @click.prevent="comName='register'">注册</a>

    <component :is="comName"></component>
</div>
<template id="login">
    <h3>这是登录表单</h3>
</template>
<template id="register">
	<h3>这是注册表单</h3>
</template>
<script type="text/javascript" src="../lib/vue.js"></script>
<script type="text/javascript">
// 创建登录组件
Vue.component('login', {
	template: '#login'
});

// 创建注册组件
Vue.component('register', {
	template: '#register'
})

new Vue({
    el: '#app',
    data: {
    	comName: 'login'
    },
    methods: {}
});
</script>
```
即使用Vue提供的`component`，它能够实现自动对组件进行展示和隐藏，通过`:is='组件名称'`。


##  父子组件间传值

### 父组件给子组件传值

父组件给子组件传值，即实现在子组件中调用父组件中的`methods`或是获取父组件中的`data`.

#### 传参数

```html
<body>
	<!-- 父组件实例 -->
    <div id="app">
		
        <!-- 子组件实例 -->
        <com1></com1>
    </div>

    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript">

    new Vue({
        el: '#app',
        data: {
        	msg: '这是父组件data值'
        },
        methods: {
        	show(){
        		console.log("这是父组件的show方法");
        	}
        },

        // 子组件
        components: {
        	com1: {
        		template: '<h2>这是子组件</h2>'
        	}
        }
    });
    </script>
</body>
```
如上，当我们直接在子组件中引用父组件`data`中定义的`msg`数据，将上面的修改为：
```
com1: {
	template: '<h2>这是子组件--{{msg}}</h2>'
}
```
那么就会报错。所以就证实了默认在子组件中不能访问父组件中的`data`或`methods`。

为了解决子组件获取父组件数据，Vue提供了以下方式获取：
* 1、父组件在引用子组件的时候，通过属性绑定的方式`v-bind:`，把需要传递给子组件的数据以属性绑定的形式传递到子组件内部，供子组件使用。

* 2、在子组件中，通过`props: []`获取到父组件传递过来的数据；这样就完成了父组件向子组件传值

**实例**
```html
<body>
	<!-- 父组件实例 -->
    <div id="app">
		
        <!-- 子组件实例 -->
        <com1 :fatoson="msg"></com1>
    </div>

    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript">

    new Vue({
        el: '#app',
        data: {
        	msg: '这是父组件data值'
        },
        methods: {},

        // 子组件
        components: {
        	com1: {
        		template: '<h2>这是子组件--{{fatoson}}</h2>',
        		props: ['fatoson']
        	}
        }
    });
    </script>
</body>
```
如上，我们在子组件实例中使用了`v-bind`绑定了一个参数`fatoson`，其值是：`msg`即在父组件data中定义的值；那么就相当于父组件的一个data数据被Vue绑定到了子组件实例中，且父组件data值的别名是`fatoson`，那么在子组件中通过`props: []`属性就能获取到这个别名，然后就实现了父组件向子组件传值。需要注意以下：

* 1、我们可以将为子组件绑定的参数名称是父组件值的别名。即此时`msg`相当于`fatoson`。
* 2、注意命名中若是驼峰命名，在HTML中必须用`-`替换。
* 3、注意`props`是Vue为父组将向子组件传值提供的一个参数，且他是唯一的数组类型的。
* 4、注意子组件通过`props`获取到的父组件的值是**只读**的，即不能修改。


#### 传方法

上面讲了父组件给子组件传递普通的参数，下面我们将了解到父组件怎样给子组件传递方法。

```html
<body>
	<!-- 父组件实例 -->
    <div id="app">
		
        <!-- 子组件实例 -->
        <com1 @open="show"></com1>
    </div>
    <template id="tmp">
    	<div>
    		<h2>这是子组件</h2>
    		<input type="button" @click="myclick" value="子组件按钮，点击触发父组件方法"/>
    	</div>
    </template>

    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript">

    new Vue({
        el: '#app',
        data: {
        	msg: '这是父组件data值'
        },
        methods: {
        	show(){
        		console.log("这是父组件的show方法");
        	}
        },

        // 子组件
        components: {
        	com1: {
        		template: '#tmp',
        		props: ['fatoson'],
        		methods: {
        			myclick(){
       		 			this.$emit('open');
        			}
        		}
        	}
        }
    });
    </script>
</body>
```
**解释：**

* 1、这里子组件的`template`数据引用外部的`<template></template>`中的HTML代码。
* 2、与传值思路相同，传递方法也需要在子组件实例中使用`v-on`即`@`来绑定方法，方法别名`@open`，方法的值是`show`是在父组件中定义的方法名。
* 3、与传值思路相同，传值使用了`props`来接受传递的参数，那么传方法提供了`$emit()`元素

#### 综合
上面讲的父组件向子组件传递方法，那么既然是方法就肯定能传递方法参数。我们只需要在父组件方法中指定值名称即可

```
@子组件接收的方法别名="父组件中的方法(父组件中的参数值或data)"
```


```html
<body>
	<!-- 父组件实例 -->
    <div id="app">
		
        <!-- 子组件实例 -->
        <com1 @open="show(fatosonval)"></com1>
    </div>
    <template id="tmp">
    	<div>
    		<h2>这是子组件</h2>
    		<input type="button" @click="myclick" value="子组件按钮，点击触发父组件方法"/>
    	</div>
    </template>

    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript">

    new Vue({
        el: '#app',
        data: {
        	fatosonval: {
        		id: '1',
        		name: '涂陌'
        	}
        },
        methods: {
        	show(data){
        		console.log(data);
        	}
        },

        // 子组件
        components: {
        	com1: {
        		template: '#tmp',
        		methods: {
        			myclick(){
       		 			this.$emit('open');
        			}
        		}
        	}
        }
    });
    </script>
</body>
```

![](https://tycoding.cn/2018/07/23/vue-3/1.png)

### 子组件给父组件传值

即实现在父组件中调用子组件中的方法

```html
<body>
	<!-- 父组件实例 -->
    <div id="app">
		
        <!-- 子组件实例 -->
        <com1 @open="show"></com1>
    </div>
    <template id="tmp">
    	<div>
    		<h2>这是子组件</h2>
    		<input type="button" @click="myclick" value="子组件按钮，点击触发父组件方法"/>
    	</div>
    </template>

    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript">

    new Vue({
        el: '#app',
        data: {
        	fatosonval: null
        },
        methods: {
        	show(data){
        		console.log(data);
        	}
        },

        // 子组件
        components: {
        	com1: {
        		template: '#tmp',
        		data(){
        			return {
        				sonval: { name: 'TyCoding', age: 19 }
        			}
        		},
        		methods: {
        			myclick(){
       		 			this.$emit('open', this.sonval);
        			}
        		}
        	}
        }
    });
    </script>
</body>
```
之前我们实现父组件向子组件传值的时候，需要在`@open()`中指定父组件中的`data`，而子组件给父组件传值的时候就不需要再指定了，而是直接在调用子组件中的`open`（这是父组件方法的别名），将子组件的值写进`$emit('方法别名', data参数)`中即可。

![](https://tycoding.cn/2018/07/23/vue-3/2.png)


## Vue获取DOM元素和组件

首先我们需要明白的就是Vue并不提倡我们操作DOM元素，Vue的宗旨就是让我们只关心业务逻辑。

那么通常我们需要获取一个如`<h2></h2>`中的值，采用原生JS通常需要先为tag标签定义一个id属性，然后通过JS代码`document.getElementById('id').innterText`来获取到`<h2>`中的文本数据，而Vue也实现了操作原生DOM的功能：

    * 在需要获取的HTML标签中指定`ref`属性，其值可自定义。
    * 在Vue实例中，使用`this.$refs.指定的值`来获取DOM对象，进行操作。

```
<h2 ref="h2">这是h2的文本数据</h2>>

console.log(this.$refs.h2.innerText);
```

**通过$refs还能轻易获取子组件中的data和methods数据**

首先需要为子组件引用实例定义`ref="com1"`属性，然后在Vue实例中通过`this.$refs.com1`即可获取子组件中的所有对象，即还能获取到子组件中的`data`和`methods`

实例：
```html
<body>
    <!-- 父组件实例 -->
    <div id="app">
        <!-- 子组件实例 -->
        <com1 ref="com1"></com1>
        <input type="button" @click="show" value="获取元素">

        <h2 ref="h2">这是父组件</h2>
    </div>
    <template id="tmp">
        <div>
            <h2>这是子组件</h2>
        </div>
    </template>
    <script type="text/javascript" src="../lib/vue.js"></script>
    <script type="text/javascript">
    new Vue({
        el: '#app',
        methods: {
            show(){
                // console.log(this.$refs.h2.innerText);
                // console.log(this.$refs.com1); //获取子组件中的所有实例对象
                // console.log(this.$refs.com1.sonval); //获取子组件中定义的data值
                console.log(this.$refs.com1.sonshow()); //调用子组件中定义的方法

            }
        },

        // 子组件
        components: {
            com1: {
                template: '#tmp',
                data() {
                    return {
                        sonval: { name: 'TyCoding', age: 19 }
                    }
                },
                methods: {
                    sonshow() {
                        console.log('子组件的方法');
                    }
                }
            }
        }
    });
    </script>
</body>
```