# 下载导入

如果你的电脑上安装了Git客户端，可以直接试用`git`命令下载项目：

```shell
git clone https://github.com/TyCoding/tumo.git
```

![image-20200531125946896](https://gitee.com/tytumo/pictures/raw/master/img/20200531125947.png)

同样，你也可以直接在 [https://github.com/TyCoding/tumo.git](https://github.com/TyCoding/tumo.git) 网站直接下载zip文件，解压后即可得到项目源码。接下来，桌面会出现 `tumo` 命名的文件夹：

![image-20200531131454829](https://gitee.com/tytumo/pictures/raw/master/img/20200531131454.png)

打开IntelliJ IDEA，点击Open or Import，选中刚才下载的tumo文件夹，点击Open：

![image-20200531131923678](https://gitee.com/tytumo/pictures/raw/master/img/20200531131923.png)

因为此项目是Maven项目，所以你需要预先在IDEA上配置好本地Maven，因为之前我们已经介绍了IDEA如何配置Maven，这里不再介绍。

如果IDEA已经配置好Maven，那么在打开`tumo`（Maven）项目后，IDEA将自动识别到此Maven项目并按照之前的Maven配置下载本项目所需的Maven依赖文件。Maven依赖下载完成后可以看到项目的`src/main/java`文件夹和`src/main/resources`文件夹以及`src/test/java`文件夹图标都自动改变颜色了：

![image-20200531135551675](https://gitee.com/tytumo/pictures/raw/master/img/20200531135551.png)

但有时IDEA并不能直接识别到这个Maven项目是SpringBoot项目，体现在IDEA的顶部工具栏中Configurations显示为灰色：

![image-20200531135822630](https://gitee.com/tytumo/pictures/raw/master/img/20200531135822.png)

此时我们需要手动更新Maven依赖，在IDEA右侧侧边栏上找到Maven，在弹出的侧边窗口上选择刷新按钮：

![image-20200531140117852](https://gitee.com/tytumo/pictures/raw/master/img/20200531140117.png)

等待IDEA加载后，最终才算是完全加载了此项目的Maven依赖：

![image-20200531140223824](https://gitee.com/tytumo/pictures/raw/master/img/20200531140223.png)

并且IDEA顶栏中Configurations显示`TumoApplication`：

![image-20200531140542079](https://gitee.com/tytumo/pictures/raw/master/img/20200531140542.png)

# 安装Lombok

项目中使用了Lombok注解，所以需要在IDEA中下载Lombok插件。注意区别项目中使用的Lombok注解是来自Maven中的Lombok依赖，所以项目一旦加载完毕Maven依赖就是可以直接运行的，而IDEA中的Lombok插件仅是为了让IDEA识别这个注解，不然IDEA编译项目时会报错❌（虽然并不会影响实际项目运行）。

在`Preferences>Plugins>Marketplace`搜索`Lombok`插件，并下载安装：

![image-20200531133806200](https://gitee.com/tytumo/pictures/raw/master/img/20200531133806.png)

# 导入数据库

使用Navicat（当然也可以使用其他数据库管理软件）新建一个MySQL数据，命名为：`tumo`：

![image-20200531134220034](https://gitee.com/tytumo/pictures/raw/master/img/20200531134220.png)

在项目源码中找到`tumo/db/db.sql`文件并打开，复制其中的SQL代码，在Navicat中打开`tumo`数据库，并点击`New Query`，将之前复制的SQL代码粘贴到这里，并执行运行命令：

![image-20200531134723510](https://gitee.com/tytumo/pictures/raw/master/img/20200531134723.png)

最终将创建如下表结构：

![image-20200531134824152](https://gitee.com/tytumo/pictures/raw/master/img/20200531134824.png)

接下来需要修改项目中关于MySQL的配置，修改为自己电脑上的MySQL连接地址和密码等。在项目源码中找到`tumo/src/main/resources/application-ev.yml`文件：

![image-20200531140431480](https://gitee.com/tytumo/pictures/raw/master/img/20200531140431.png)

修改完毕后，点击IDEA上方`TumoApplication`右侧紧挨的绿色按钮，启动项目：

![image-20200531140746583](https://gitee.com/tytumo/pictures/raw/master/img/20200531140747.png)

项目启动成功后，使用浏览器（建议Chrome）访问：

- 博客前台：[http://localhost:8080/](localhost:8080/)
- 博客后台：[http://localhost:8080/login](localhost:8080/login)

![截屏2020-06-29 下午7.02.00](http://cdn.tycoding.cn/20200629190203.png)

到此为止，[Tumo](https://github.com/TyCoding/tumo) 单体架构项目已经启动完成。