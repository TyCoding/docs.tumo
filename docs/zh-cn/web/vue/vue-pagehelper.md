# Vue + ElementUI + SpringMVC实现分页

这一段时间写项目用到了Vue+ElementUI，这里记录一下使用ElementUI内置分页插件结合后端SSM框架的实现思路和实现过程。

其中遇到了很多坑，我会尽量把见到的坑都记录下来，希望对你有所帮助。

本案例对应的**开源项目地址**请看我的GitHub仓库：

* [优雅的入门SpringBoot+Mybatis，实现简单的CRUD ](https://github.com/TyCoding/spring-boot)

* [优雅的实现电商项目中搜索功能，整合SSM+Redis+Shiro+Solr框架，教你使用Vue+ElementUI写一个炫酷的后端页面 ](https://github.com/TyCoding/ssm-redis-solr)

<!--more-->

**首先** 让我们看一下最终效果：

![](https://tycoding.cn/2018/07/27/vue-6/1.png)

# 起步
本博文的主要讲一下Vue+ElementUI结合后端SpringMVC实现分页的实现思路，基本的elementUI用法请自行百度；

Vue的常用语法可以看我的 [博文](http://tycoding.cn/2018/07/25/vue-4/#more) 。

关于SSM的整合教程可以看我的这篇 [博文](http://tycoding.cn/2018/06/05/ssm-2/#more)； [GitHub](https://github.com/TyCoding/ssm)。

<br/>

**介绍**

本案例中设计到的技术栈：

* [ElementUI](http://element-cn.eleme.io/#/zh-CN)

* [Vue.js](https://cn.vuejs.org/v2/guide/)

* [vue-resource.js](https://www.npmjs.com/package/vue-resource)

* [SSM框架](http://tycoding.cn/2018/06/05/ssm-2/#more)

* PageHelper: Mybatis的分页插件


# 准备

1、SSM框架的整合教程可以参考我的这篇博文：[手摸手带你整合SSM框架](http://tycoding.cn/2018/06/05/ssm-2/#more); [GitHub](https://github.com/TyCoding/ssm)。

2、在后端项目中导入`PageHelper.jar`的依赖
```
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>4.0.0</version>
</dependency>
```

*****注意**
使用PageHelper分页插件除了要导入依赖，还需要在Mybatis配置文件中进行相关配置，并交给Spring进行管理。如下配置即可：

```
<plugins>
    <!-- com.github.pagehelper 为 PageHelper 类所在包名 -->
    <plugin interceptor="com.github.pagehelper.PageHelper">
        <!-- 设置数据库类型 Oracle,Mysql,MariaDB,SQLite,Hsqldb,PostgreSQL 六种数据库-->
        <property name="dialect" value="mysql"/>
    </plugin>
</plugins>
```
这里还要注意的是PageHelper5.X版本和PageHelper4.X版本PageHelper类所在的包名是不同的。
在Spring配置文件中扫描此配置文件即可：
![](https://tycoding.cn/2018/07/27/vue-6/3.png)

3、在HTML中导入`vue.js` and `element-ui` 。


好的，至此，我们把基本的环境已经讲过了，下面看下相关前端代码：

```html
<!-- 列表 -->
<el-table
        ref="user"
        :data="user"
        tooltip-effect="dark"
        style="width: 100%">
    <el-table-column
            prop="id"
            sortable
            label="编号"
            width="80">
    </el-table-column>
    <el-table-column
            prop="username"
            sortable
            label="联系人"
            width="120">
    </el-table-column>
    <el-table-column
            prop="phone"
            sortable
            label="联系电话"
            width="120">
    </el-table-column>
    <el-table-column
            prop="mailbox"
            label="电子邮箱"
            width="150">
    </el-table-column>
    <el-table-column
            prop="postalCode"
            sortable
            label="邮政编码"
            width="120">
    </el-table-column>
    <el-table-column
            prop="date"
            sortable
            label="注册时间"
            width="200">
    </el-table-column>
    <el-table-column
            prop="address"
            label="通讯地址"
            width="200"
            show-overflow-tooltip>
    </el-table-column>
</el-table>

<!-- 分页 -->
<div class="pagination">
    <el-pagination
            background
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="pageConf.pageCode"
            :page-sizes="pageConf.pageOption"
            :page-size="pageConf.pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="pageConf.totalPage">
    </el-pagination>
</div>
```

## 前端

注意我们上面前端HTML样式用使用Vue绑定的数据：

1、列表数据
```html
//注意这部分代码是在Vue实例中的data属性中定义的

data() {
	//用户信息
    //element-ui的table需要的参数必须是Array类型的
    user: [{
        username: '',
        phone: '',
        mailbox: '',
        postalCode: '',
        date: '',
        address: ''
    }],
}
```
上面ElementUI表格中`<el-table>`中用Vue绑定的`:data="user"`就是这个数据，**注意：**这里的user对象中的数据需要是**Array**类型的，不要问为什么，请去看ElementUI源码；


2、分页数据
```html
//注意这部分代码是在Vue实例中的data属性中定义的

data() {
	//定义分页Config
	pageConf: {
	    //设置一些初始值(会被覆盖)
	    pageCode: 1, //当前页
	    pageSize: 4, //每页显示的记录数
	    totalPage: 12, //总记录数
	    pageOption: [4, 10, 20], //分页选项
	    handleCurrentChange: function () {
	        console.log("页码改变了");
	    }
	},
}

methods: {
	//pageSize改变时触发的函数
    handleSizeChange(val) {},
    //当前页改变时触发的函数
    handleCurrentChange(val) {},
}
```
上面`<el-pagination>`中绑定的数据就来自这个对象:`pageConf`，那么下面你需要关注`<el-pagination>`中的几个配置参数（方法通过Vue的`@`绑定，数据通过Vue的`:`绑定）：


* `@size-change`: 表示每页记录的个数发生变化时触发的函数，如：原来是每页/3条，变为每页/6条；`handleSizeChange`中包含一个参数表示当前是每页显示几条记录。
	
* `@current-change`: 表示当前页发生变化时触发的函数，如：点击下一页；`handleCurrentChange`中包含一个参数表示当前是第几页。
	
* `:current-page`: 当前页，即我们命名的`pageCode`，表示当前页面上展示的第几页。
	
* `:page-sizes`: 分页选项，即页面提供一个列表让你选择每页显示多少条记录，注意这个参数的第一个值表示当前页是`每页/记录`，你写上即生效。
	
* `:page-size`: 表示每页显示的记录数，即我们命名的`pageSize`。
	
* `:total`: 表示总记录数，即我们这个表格中一共要显示多少条数据。

<br/>

### 注意：

* 以上代码可能与截图中样式不符，因为我把这篇博文中不涉及的都删除了。

* 表格中的数据来自`:data`这个绑定的**对象数组**中，即我们再Vue实例data中定义的`user: [{}]`，前提是你在每一个`<el-table-column>`中都定义了`prop`并标识了`user:[{}]`中定义的变量，不然element-ui不知道你想在表格的这一行显示什么，当然这已经比我们常用的表格渲染数据方便很多了。

* element-ui自带的分页插件需要提供数据才能正常显示分页信息，这些数据都应该是动态的，所以我们绑定在`pageConf`对象中；因为这些数据应该是后端读取出来的，即通过得到后端传来的分页数据，我们才知道这里的分页信息应该怎样定义。

* 在data中定义的`pageConf`是初始化参数，最后会被覆盖掉，但是要注意`pageOption`这个参数，一定要和初始的`pageSize`配合服用。

* 以上涉及两个函数`handleSizeChange`、`handleCurrentChange`，我们要在其触发时自动改变对应的`pageOption`参数。

<br/>

### 会遇到的坑

1、`<el-table>`中需要渲染的数据仅需要传入`:data="user"`即可，但是这个数据`user`必须是一个**对象数组**，一定是**数组**

2、想要`<el-table>`正确渲染你`user`中定义的数据，你必须为每个`<el-table-column>`定义`prop`属性，绑定对应你想展示的数据，不然ElementUI不知道你想展示什么。

3、`pageOption`分页选项一定要注意，要配合`pageSize`的默认值，不要乱定义，比如：`pageSize: 2, pageOption: [10,20,30]`，这样你就会发现页码根本不能正确显示，因为你设置`pageSize:2`表示你想每页展示2条数据，但是你又定义`pageOption: [10,20,30]`第一个参数即是默认被选中的，即你又想每页显示10条数据，那么ElementUI就蒙蔽了，不知道你到底想每页显示几条数据。

3、根据上面的参数，以及`handleSizeChange`、`handleCurrentChange`这两个函数的参数你就应该想到分页的实现其实是`pageCode`(当前页)和`pageSize`(每页显示的记录数)和后端进行数据交换的。在前端你需要关心的怎样把`pageSize`和`pageCode`传给后端进行分页查询；在后端你需要关心的是怎样调用`pageHelper`插件将分页的记录数据（包括`totalPage`、`user`数据等）return 给前端。


<br/>

## 后端

定义请求映射路径：`findByPage`
```
@RequestMapping("/findByPage")
public PageBean findByPage(@RequestParam("pageCode") int pageCode, @RequestParam("pageSize") int pageSize) {
    System.out.println("分页的数据：" + userService.findByPage(pageCode, pageSize));
    return userService.findByPage(pageCode, pageSize);
}
```

### 注意

如上是我们在Controller中定义的请求映射路径，其中需要接收两个参数：`pageCode`和`pageSize`分别表示当前页、每页显示的记录数；即前端请求这个方法时只需要将`pageCode`和`pageSize`传进来就行，后端使用`pageHelper`分页插件将查询到的数据进行分页，并将结果返回给前端。

对于请求映射中包含多个参数的，应该使用`@RequestParam()`进行标记，不然可能报错400等。


<br/>

# 逻辑思路

## 后端

首先我们需要定义分页实体类：`PageBean.java`
```java
public class PageBean() implements Serialization {
	//当前页
    private long total;
    //当前页记录
    private List rows;
}
```

因为我们使用了mybatis的分页插件：`PageHelper`，所以`PageHelper`最终为我们封装在`PageBean`的数据应该是这个样子的：

![](https://tycoding.cn/2018/07/27/vue-6/2.png)

**注意：**需要返回JSON格式数据。可以看到里面主要包含两个参数：`total`、`rows`

* `total`表示当前数据的分页得到的总页数，相当于我们前端定义的`pageCode`。
* `rows`表示当前查询到数据的集合体。

即后端的逻辑比较简单，因为最麻烦的分页逻辑，`PageHelper`已经帮我们完成了，我们需要做的：

1、在Controller中定义请求映射方法：`PageBean findByPage(@RequestParam("pageCode")int pageCode, @RequestParam("pageSize")int pageSize){}`
	
2、Controller调用Service，通过`PageHelper`分页插件获取到这两个参数pageCode,pageSize，自动进行分页计算。
	
3、Service调用Dao，指定对应的SQL`SELECT * FROM user`，可以看到这个SQL仅仅需要查询所有数据即可，返回的数据类型是`com.github.pagehelper.Page`
	
4、Controller需要返回给前端的数据类型是：`PageBean`(我们自定义的)，其中有两个参数：`com.github.pagehelper.Page.getTotal()`和`com.github.pagehelper.Page.getResult()`。
	
5、综上，我们基本已经获取到了数据，然后通过SpringMVC提供的注解：`@RsponseBody`(局部标识方法)或`@RestController`(全局标识类)，自动将返回的数据转换为JSON格式，然后再发送给前端。

<br/>

## 前端

前端逻辑相对复杂一些，我们主要需要关注两点：

> 1.进入页面触发的事件方法、以及点击分页相关的按钮怎样和后端交互？
> 2.如何将后端交互返回的数据赋值给表格中的绑定的数据、以及分页组件中绑定的数据，并实现HTML页面的渲染？

### 第一点

**进入页面触发的事件方法、以及点击分页相关的按钮怎样和后端交互？**

> 1.有哪些可能被触发的事件和方法？

* findByPage(pageCode,pageSize)
  这个是分页的核心方法，会被多次触发。又因为进入页面就应该理解渲染表格中的数据，所以分页方法应在渲染页面时就执行，所以需要在`created`声明周期函数中调用`findByPage(this.pageConf.pageCode,this.pageConf.pageSize)`(传入默认的值)。对应的HTML代码：
```html
findByPage(pageCode, pageSize) {},
```

* handleSizeChange(val)
  这个函数是当pageSize（每页显示的记录数）改变时被触发，通过HTML中的`@size-change`属性绑定。比如：原来4条/每页改变为6条/每页，就将触发这个函数；其中的参数`val`表示当前页每页显示几条记录`pageSize = val`。对应的HTML代码：
```html
handleSizeChange(val) {
    this.findByPage(this.pageConf.pageCode, val);
},
```
每当pageSize改变就需要重新调用`findByPage(this.pageConf.pageCode, val)`函数重新计算页面需要渲染的数据。

* handleCurrentChange(val)
  这个函数是当pageCode(当前页)改变时触发的函数，通过HTML中的`@current-change`属性绑定。比如：点击下一页、上一页，就会触发这个函数；其中的参数`val`表示当前是第几页`pageCode = val`。对应的HTML代码：
```html
handleCurrentChange(val) {
    this.findByPage(val, this.pageConf.pageSize);
},
```
每当pageCode改变时就需要重新调用`findByPage(val, this.pageConf.pageSize)`函数从新计算页面需要渲染的数据。


> 2.分页相关按钮是什么鬼？

在传统没有每页插件的时候，我们通常会手写分页逻辑，那么就需要为每一个页面绑定一个触发方法，而使用了element-ui提供的分页插件，大大简化了分页逻辑，其中点击的下一页、上一页、点击每页显示记录选项、去第几页等这些功能都是ElementUI自动帮我们绑定了事件。


> 3.怎样和后端交互？

和后端实现交互的方法主要是`findByPage()`这个核心方法，其相关JS代码：
```html
findByPage(pageCode, pageSize) {
    this.$http.post('/user/findByPage.do', {pageCode: pageCode, pageSize: pageSize}).then(result => {
        this.pageConf.totalPage = result.body.total;
        this.user = result.body.rows;
    });
},
```
如上，findByPage()是我们定义的分页的核心方法，所有其他分页中触发的方法都会调用这个方法重新和后端交互，获取到最新的数据并返回给页面。其中你需要注意：

* findByPage()中包含两个参数：pageCode、pageSize。
	
* 调用vue-resource提供的post请求方法，其中传入两个参数pageCode、pageSize；在`then()`回调函数中可获取请求返回的数据。
	
* 注意Controller返回的数据就在`result`这个参数中，但是实际的数据是在`result.body`中的，所以你直接`result.total`是获取不到数据的。
	
* 前面已经看到了，后端主要返回两个封装了数据的参数：`total`(总页数)、`rows`(核心数据)
	
* findByPage方法请求后端得到了`total`和`rows`，就应该分别赋值给`this.pageConf.totalPage`和`this.user`；根据Vue双向绑定的功能，页面新的数据会直接渲染出来。

### 第二点

**如何将后端交互返回的数据赋值给表格中的绑定的数据、以及分页组件中绑定的数据，并实现HTML页面的渲染？**

其实第一点中我们已经讲到了，因为Vue有一个双向绑定的功能，即我们请求后端将数据赋值给`data:{}`中的对象后，HTML页面会立即渲染新的`data`数据。

> 如何将后端返回的数据赋值给页面需要展示的数据？

首先是`<el-table>`中要渲染的数据，其来自`:data="user"`绑定的user对象，我们需要将后端返回的数据赋值给这个`user`根据双向绑定思想即会更新表格中的数据。

其次就是`<el-pagination>`中定义的分页参数，由于element-ui分页插件已经帮我们完成了很多逻辑计算，我们需要交互改变的参数只有三个：`pageCode`当前页、`pageSize`每页显示的记录数、`totalPage`总记录条数，而后端返回的数据我们也看过，综上：我们只需要将后端返回的总页数`total`赋值给`user`对象中的属性`totalPage`即可。

主要JavaScript代码
```JavaScript
findByPage(pageCode, pageSize) {
    this.$http.post('/user/findByPage.do', {pageCode: pageCode, pageSize: pageSize}).then(result => {
        this.pageConf.totalPage = result.body.total;
        this.user = result.body.rows;
    });
},
```

<br/>

# 代码编写

经过上面的分析，其实很多代码已经展示出来了，下面我们看看完整的代码：

## 后端

**实体类**
```java
public class PageBean implements Serializable {
    //当前页
    private long total;
    //当前页记录
    private List rows;

    ...
}

public class User implements Serializable {
    private Long id; //用户编号
    private String username; //用户名
    private String password; //密码
    private String phone; //联系电话
    private String mailbox; //邮箱
    private String address; //地址
    private String postalCode; //邮政编码
    private String date; //注册日期

    ...
}
```

**Controller**
```java
@ResponseBody
@RequestMapping("/findByPage")
public PageBean findByPage(@RequestParam("pageCode") int pageCode, @RequestParam("pageSize") int pageSize) {
    return userService.findByPage(pageCode, pageSize);
}
```

**Service**
```java
import com.github.pagehelper.Page;
import com.github.pagehelper.PageHelper;
import com.instrument.dao.UserDao;
import com.instrument.entity.PageBean;
import com.instrument.entity.User;

...

public PageBean findByPage(int pageCode, int pageSize) {
    //使用Mybatis分页插件
    PageHelper.startPage(pageCode,pageSize);

    //调用分页查询方法，其实就是查询所有数据，mybatis自动帮我们进行分页计算
    Page<User> page = userDao.findByPage();
    return new PageBean(page.getTotal(),page.getResult());
}
```
这里dao层调用的`findByPage()`对应的SQL仅仅是`SELECT * FROM 表`。而分页是调用的`startPage()`和`Page`函数两者共同完成的分页逻辑计算，其返回的数据主要是在`total`和`rows`中封装着。

**mapper.xml**
```xml
<!-- 分页查询 -->
<select id="findByPage" resultType="com.instrument.entity.User">
    SELECT * FROM user
</select>
```

## 前端

```html
<div id="#app">
	<el-table
	    ref="user"
	    :data="user"
	    tooltip-effect="dark"
	    style="width: 100%">
		<el-table-column
		        prop="id"
		        sortable
		        label="编号"
		        width="80">
		</el-table-column>
		<el-table-column
		        prop="username"
		        sortable
		        label="联系人"
		        width="120">
		</el-table-column>
		<el-table-column
		        prop="phone"
		        sortable
		        label="联系电话"
		        width="120">
		</el-table-column>
		<el-table-column
		        prop="mailbox"
		        label="电子邮箱"
		        width="150">
		</el-table-column>
		<el-table-column
		        prop="postalCode"
		        sortable
		        label="邮政编码"
		        width="120">
		</el-table-column>
		<el-table-column
		        prop="date"
		        sortable
		        label="注册时间"
		        width="200">
		</el-table-column>
		<el-table-column
		        prop="address"
		        label="通讯地址"
		        width="200"
		        show-overflow-tooltip>
		</el-table-column>
	</el-table>
	<!-- 分页 -->
	<div class="pagination">
	    <el-pagination
	            background
	            @size-change="handleSizeChange"
	            @current-change="handleCurrentChange"
	            :current-page="pageConf.pageCode"
	            :page-sizes="pageConf.pageOption"
	            :page-size="pageConf.pageSize"
	            layout="total, sizes, prev, pager, next, jumper"
	            :total="pageConf.totalPage">
	    </el-pagination>
	</div>
</div>

<script type="text/javascript" src="../vue.js"></script>
<script type="text/javascript">
new Vue({
	el: '#app'
	data(){
		//用户信息
        //element-ui的table需要的参数必须是Array类型的
        user: [{
            username: '',
            phone: '',
            mailbox: '',
            postalCode: '',
            date: '',
            address: ''
        }],
        //定义分页Config
        pageConf: {
            //设置一些初始值(会被覆盖)
            pageCode: 1, //当前页
            pageSize: 4, //每页显示的记录数
            totalPage: 12, //总记录数
            pageOption: [4, 10, 20], //分页选项
            handleCurrentChange: function () {
                console.log("页码改变了");
            }
        },
	},
	methods:{
		findByPage(pageCode, pageSize) {
            this.$http.post('/user/findByPage.do', {pageCode: pageCode, pageSize: pageSize}).then(result => {
                this.pageConf.totalPage = result.body.total;
                this.user = result.body.rows;
            });
        },
        //pageSize改变时触发的函数
        handleSizeChange(val) {
            this.findByPage(this.pageConf.pageCode, val);
        },
        //当前页改变时触发的函数
        handleCurrentChange(val) {
            this.findByPage(val, this.pageConf.pageSize);
        },

        // 获取所有数据
        findAll() {
            this.$http.post('/user/findAll.do').then(result => {
                this.user = result.body;
            });
        }
	},
	created(){
		this.findAll();
        this.findByPage(this.pageConf.pageCode, this.pageConf.pageSize);
	}
});
</script>
```
以上代码我们基本已经解释过了，唯一没有提到的就是`findAll()`这个方法，要知道，进入到页面后，首先就是展示所有数据（即使有没有分页）；那么就需要在生命周期函数`created`中执行`findAll()`获取所有数据直接渲染到页面上`this.user=result.body`即可。其次又因为我们使用了分页查询功能，进入页面后展示的数据应该是分页查询后的数据（因为我们设置有默认的分页参数值）。