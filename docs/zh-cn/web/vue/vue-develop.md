**Vue实现前后端分离项目的初体验**

经过之前学习的Vue的知识：

* [vue基本指令](http://tycoding.cn/2018/07/21/vue-1/#more)
* [vue组件](http://tycoding.cn/2018/07/23/vue-3/#more)
* [vue-resource](http://tycoding.cn/2018/07/22/vue-2/#more)
* [vue路由](http://tycoding.cn/2018/07/23/vue-3/#more)

其实我们已经可以开始实战运用到实际的Web项目中了，由于本人是Java后端开发的，所以后端是基于SpringMVC的。

以下我们将演示如何使用Vue组件实现一个用户登录系统。

<!--more-->

# 介绍

## 后端

>后端基于spring、springMVC、mybatis框架

对后端SSM框架搭建不熟悉的可以参考我的博文： [SSM项目整合](http://tycoding.cn/2018/06/05/ssm-2/#more)

## 前端

前端使用了wepack打包工具，利用了`vue.cli`脚手架快速搭建的项目。由于本人对一些技术也不是很熟悉，所以给大家提供些官方文档，
想要深入学习还是要仔细分析官方文档。传送门如下（也是本项目要用到的）：

* [node.js](https://nodejs.org/zh-cn/)
* [vue.js](https://cn.vuejs.org/v2/guide/)
* [vue-cli](https://github.com/vuejs/vue-cli)
* [vue-resource](https://www.npmjs.com/package/vue-resource)
* [vue-router](https://router.vuejs.org/)
* [webpack](http://webpack.github.io/)
* [element-ui](http://element.eleme.io/#/zh-CN)

以上技术都是比较常用的，`webpack`类似一个打包工具，它会将你项目中的Vue组件打包为一个庞大的`js`文件（当然我们是看不懂的），而我们的前端项目是部署在`node.js`提供的web容器中的。

即前后端分离的实际效果是这样的：
![index.html](vue-5/1.png)

![index](vue-5/2.png)

有没有感觉很神奇，哈哈，反正我第一次见到的时候就是觉得很神奇，因为`html`中完全看不到任何js、css代码，但是却能渲染出来这么好看的页面。下面我们就讲一下，怎么实现这个过程吧！！

<br/>

# 环境
上面我们提到的技术，在本案例中都会遇到，后端的SSM框架请参考我的博客文档，介绍一下前端环境的搭建：

**1、安装`node.js`**

具体安装流程还是要去看`node.js`官网。如果安装完成，请在终端输入`npm -v`查看是否安装成功。一般会出现类似如下版本号：

>v8.11.3

**2、安装`VueCLI`脚手架**

`VueCLI`能够帮助我们快速搭建一个webpack的项目。

在已经安装好`node.js`的前提上，在终端输入：`npm install vue-cli -g`开始安装VueCLI。如果安装完成，请在终端输入：`vue -V`，会出现如下版本信息：

>2.9.6

具体安装流程可以参看：[博文](https://blog.csdn.net/qq_36711388/article/details/79405402)

由于**VueCLI脚手架**已经帮我们安装好了`webpack`、以及相关的node依赖包，所以我们不需要再手动安装了。


## 范例

如果安装完成，我们可以看到出现如下目录结构：

![](https://tycoding.cn/2018/07/27/vue-5/3.png)

### 启动项目

在终端项目路径下输入`npm run dev`命令；正常情况下，会出现如下信息：

![](https://tycoding.cn/2018/07/27/vue-5/4.png)

在浏览器中输入指定的URL，会出现如下页面：

![](https://tycoding.cn/2018/07/27/vue-5/5.png)

我们继续观察，打开项目中的`index.html`

![](https://tycoding.cn/2018/07/27/vue-5/6.png)

我们看到，这个HTML中没有任何代码，甚至没有引入js、css，但是页面中的视图是怎样渲染出来的呢？

仔细看项目结构，我们能看到在`src/components/`下有一个`HelloWorld.vue`程序，我们页面中的程序就是通过这些`.vue`组件来渲染出来的。

### 打包项目

如果我们想将项目部署到服务器上，你放一堆`.vue`程序，浏览器是无法解析出来的，所以我们需要了解一下`webpack`的打包命令：

> npm run build

![](https://tycoding.cn/2018/07/27/vue-5/7.png)

正常情况下，会显示上图中的信息，表示打包成功了，会在项目根目录中生成一个叫`dist`的文件夹，里面是生成的静态项目：

![](https://tycoding.cn/2018/07/27/vue-5/8.png)

我们双击`dist/index.html`，会看到和之前一样的页面，但是其中引入了一个`XX.js`文件

# 开始
经过上面的步骤我们应该了解到了所谓前端分离的简易概念，其实在之前的博文： [Vue组件](http://tycoding.cn/2018/07/23/vue-3/#more) 我们已经了解到了Vue的模块化开发流程。配合`.vue`组件，其实思路还是相同的。

## 搭建前端
开始之前，我们首先要安装`vue-resource`（`element-ui`），执行：

> npm install vue-resource
> npm install element-ui


### 第一步：修改main.js
`main.js`文件是`webpack`的核心入口，我们需要在其中引入`Vue-resource`以及`router`

```
import VueResource from 'vue-resource'

import router from './router/index.js'
```

**在Vue实例中注册router**
```
new Vue({
	router,
});
```

完整代码：

![](https://tycoding.cn/2018/07/27/vue-5/9.png)


### 第二步：修改router/index.js
这是有关Vue路由的配置，前面我们也已经讲过了vue的路由，这里不再多说，代码如下：

![](https://tycoding.cn/2018/07/27/vue-5/10.png)

如上就配置了，如果你访问`localhost:8081/`，那么就会自动路由跳转到`login.vue`组件中，提示我们登录;其中的`/home`表示，如果登录成功，就跳转到`home.vue`组件中，相当于登录成功后跳转到后台页面。

### 第三步：创建login.vue
在`src/components/`下创建`login.vue`组件。

login组件中表单样式就不再讲了，我们主要看一下怎样利用`v-model`绑定表单数据，并请求后端

**表单原型**

```html
<!-- 登录表单 -->
<el-form :model="login" status-icon :rules="rule" ref="login">
    <el-form-item prop="username">
        <el-input prefix-icon="el-icon-ump-yonghu" v-model="login.username"
                  auto-complete="off"/>
    </el-form-item>
    <el-form-item prop="password">
        <el-input prefix-icon="el-icon-ump-mima" type="password" v-model="login.password"
                  auto-complete="off"/>
    </el-form-item>
    <el-form-item>
        <el-checkbox class="check" v-model="checked">记住我</el-checkbox>
    </el-form-item>
    <el-form-item>
        <el-button class="btn" type="primary" @click="submitForm('login')">登录</el-button>
    </el-form-item>
</el-form>
<div>
    <p><a href="#" class="tips">还没有账号？点我去注册</a></p>
</div>
```
在上面的表单中，我们只需要关注三个点：

* v-model="login.username"
* v-model="login.password"
* @click="submitForm('login')"

为什么不关注其他的？
注意，这个案例由于我使用了`element-ui`，从标签中就能看出来，其中涉及到了一些element-ui提供的js校验，我们只需要关注Vue的逻辑即可。


**提交表单**

关于上面提到的element-ui的校验部分
![](https://tycoding.cn/2018/07/27/vue-5/11.png)

表单提交方法部分
```javascript
methods: {
    submitForm(login) {
        this.$refs[login].validate((valid) => {
            if (valid) {
                //提交表单
                this.$http.post('http://127.0.0.1:8080/login.do', {
                    username: this.login.username,
                    password: this.login.password
                }).then(result => {
                    console.log(result);
                    if (result.bodyText === 'index') {
                        this.$router.push({ path: 'home' }); //跳转到home组件中
                    } else {
                        console.log("登录失败");
                        return false;
                    }
                });
            } else {
                console.log('error submit!!');
                return false;
            }
        });
    },
}
```
上面就是我们要讲到的核心部分：请求后端的接口

**解释：**

* 首先需要注意的`this.$refs[login].validate((valid)){}`是element-ui提供的表单验证的逻辑，但是是结合vue.js的。因为我们若在不验证表单直接提交的时候，会在表单提交按钮中直接传`@click="submitForm(login)"`，因为此时的`login`是一个`data`中已经声明的对象，其中包含两个参数：`username`,`password`。而element-ui提供的方式则会根据`.validate()`获取到`login`所包含的参数从而实现校验。

* 经过上面的校验，如果校验成功，那么将调用`this.$http.post()`进行提交post表单，这是`vue-resource`提供的方式，[博文](http://tycoding.cn/2018/07/22/vue-2/#more) 中我们也讲过。其中包含了两个参数，username,password。

* 请求成功，调用`.then()`回获取到成功的请求结果。判断请求的结果：我这里是从后端返回的参数（`return "index"`）中判断是否登录成功，如果能录成功，就应该跳转到`home`组件中。

* 调用`vue-router`中提供的`$router.push`方法，我们可以理解为向`Router`对象中添加了一条路由地址，其URL是：`path: 'home'`，那么就表明了会跳转带名字叫`/home`的路径下，整好对应的是我们配置好的`home.vue`组件。


**注意：**

* 最重要的就是跨域请求问题，本例中`node.js`提供的web服务器地址是：`127.0.0.1:8081`，但是我们后端Tomcat服务器的地址是：`127.0.0.1:8080`，而默认是不能在一个域中访问另一个域中的资源的，所以也就出现了跨域请求的概念。

* 其次重要的就是`$http.post()`的第一个参数：URL地址，不要写`locahost:`，不要写....  具体原因不是很清楚，不然请求还会报错为跨域请求。

* 解决跨域请求的方式也有很多，这里我提供一个比较简单的方式，只需要在后端的`web.xml`中提供一个允许跨域访问的过滤器就行了，后面会介绍。

* 还有就是之前我们就说过Vue中默认提交的post请求时不包含表单格式的，所以需要配置，我已经在`main.js`中写了`Vue.http.options.emulateJSON = true;`设置全局表单提交格式，所以在`post()`方法中就没有设置。


### 第四部：创建home.vue

在`src/components/`下创建`home.vue`组件

![](https://tycoding.cn/2018/07/27/vue-5/15.png)

<br/>
<br/>

## 搭建后端

经过上面介绍了前端搭建步骤后，搭建后端我们就相对熟悉了，我们的目标就是在controller中提供一个接口`login.do`让前端访问。

### 解决跨域请求

之前已经说了跨域请求很重要，不然我们写的所有请求都无法顺利访问后端接口。解决的方式如下：

**配置**

我们只需要在项目中的`web.xml`中配置如下代码即可。因为这个过滤器是tomcat提供的，所以我们并不需要导入任何jar包。
```
<!--配置允许跨域访问-->
<filter>
    <filter-name>CorsFilter</filter-name>
    <filter-class>org.apache.catalina.filters.CorsFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>CorsFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Controller

讲了那么多，终于到了Controller层，这里就比较简单了，就是根据获取到的参数判断数据库中有没有对应的用户，有就登录成功，否者亏登录失败。

![](https://tycoding.cn/2018/07/27/vue-5/12.png)

由于我这里使用了shiro，需要将密码加密处理，所以没有直接传入到service层，当然思路是相同的。

**注意：**

* 我这个`login.do`接口返回的是JSON字符串，前面使用了`@RestController`注解，不要误认为是返回的页面，那么就会404的。
* 接受的参数要用`@RequestParam`注解标记，不然会接受不到前端传递的数据


**请求成功**

![](https://tycoding.cn/2018/07/27/vue-5/13.png)

跳转到home组件中：

![](https://tycoding.cn/2018/07/27/vue-5/14.png)