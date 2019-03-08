![](https://tycoding.cn/2018/11/28/docker/logo.png)

 **Docker**是一种容器技术，它的存在就是为了解决容器技术本身的复杂性。Docker容器拥有很高的性能，同时同一台宿主机中可以运行更多的容器，是用户尽可能的充分利用系统资源。

<!--more-->

# 概念

> 1.什么是虚拟化？

在计算机中，虚拟化（Virtualization）是一种资源管理技术，是将计算机的各种实体资源，如服务器、网络、内存及储存等，予以抽象、转换后呈现出来，打破实体结构间的不可切割的障碍，是用户可以比原来的组态更好的方式来应用这些资源。这些资源的新虚拟部分是不受现有资源的架设方式、地域或物理组态所限制。

* 全虚拟化架构

虚拟机的监视器（hypervisor）是类似于用户的应用程序运行在主机的OS之上，如VMware的workstation，这种虚拟化产品提供了虚拟的硬件。

> 2.容器技术

容器和管理程序虚拟化（hypervisor virtualization, HV）不同，管理程序虚拟化通过中间层将一台或多台独立的机器虚拟运行在物理硬件之上，而**容器**是直接运行在操作系统内核之上的用户空间。因此，容器虚拟化也被称为“操作系统级虚拟化”，容器技术可以让多个独立的用户空间运行在同一台宿主机上。

**限制**

1. 由于“客居”与操作系统，容器只能运行与底层主机相同或相似的操作系统，比如在Ubuntu服务中运行CentOS，但无法运行Windows。
2. 相对于彻底隔离的管理程序虚拟化，容器被认为是不安全的。

最新的容器技术引入了OpenVZ、Solaris Zones以及Linux容器（LXC）。使用这些新技术，容器不再仅仅是一个单纯的运行环境。在自己的权限类内，容器更像一个完整宿主机。和传统虚拟化及半虚拟化想比，容器不需要模拟层（emulation layer）和管理层（hypervisor layer），而使用操作系统的系统调用接口。

## Docker特点

1. 上手快

用户可以很容易的把自己的程序Docker化。Docker依赖于“写时复制”（copy-on-write）模型，开箱即用。

2. 快速高效的开发声明周期

<br/>

## Docker组件

### Docker客户端和服务器

Docker是一个客户端-服务器（C/S）架构程序。Docker客户端只需要向Docker服务器或守护进程发出请求，服务器或守护进程将完成所有工作并返回结果。Docker提供了一个命令行工具Docker以及一整套RESTful API。你可以在同一台宿主机上运行Docker守护进程和客户端，也可以从本地的Docker客户端连接到运行在另一台宿主机上的远程Docker守护进程。

![](https://tycoding.cn/2018/11/28/docker/1.png)

### Docker 镜像

**镜像** 是构建Docker的基石。用户及基于镜像来运行自己的容器。镜像也是Docker声明周期中的“构建”部分。奖项是基于联合文件系统的一种层式结构，由一系列指令一步一步构建出来。

### Registry 注册中心

Docker用Registry来保存用户构建的镜像。Registry分为共有和私有两种。Docker公司运营公共的Registry叫做Docker Hub。

### Docker容器

Docker可以帮助你构建和部署容器，你只需要把你的程序打包放进容器即可。容器是基于镜像启动的，容器找那个可以运行一个或多个进程。我们可以认为，镜像是Docker生命周期中构建和打包阶段，而容器则是启动或执行阶段。容器基于镜像启动。

![](https://tycoding.cn/2018/11/28/docker/2.png)

<br/>

# Docker的安装与启动

本例中使用了CentOS7作为服务器，有关VMware安装CentOS7的教程请看我的这篇文章：[FastDFS系统搭建](https://tycoding.cn/2018/08/29/fastdfs/) 。

通过以下命令在线在CentOS7中安装Docker：

```
yum install docker
```

![](https://tycoding.cn/2018/11/28/docker/3.png)

**查看Docker版本**

```
[root@localhost ~]# docker -v
Docker version 1.13.1, build 8633870/1.13.1
```

## 启动与停止Docker

`systemctl`命令是系统服务管理器指令，它是`service`和`chkconfig`两个命令组合。

1. 启动Docker

```
systemctl start docker
```

2. 停止Docker

```
systemctl stop docker
```

3. 重启Docker

```
systemctl restart docker
```

4. 查看Docker状态

```
systemctl status docker
```

5. 开机启动Docker

```
systemctl enable docker
```

6. 查看Docker概要信息

```
docker info
```

7. 查看Docker帮助文档

```
docker -help
```

<br/>

## Docker镜像操作

Docker镜像由文件系统堆叠而成（是一种文件的储存形式）。最低端是一个文件引导系统，即bootfs。实际上，当一个容器启动后，它将会被移动到内存中，而引导文件系统则会被卸载，以留出更多的内存供磁盘镜像使用。Docker容器启动是需要一些文件的，而这些文件就可以被称为Docker镜像。

![](https://tycoding.cn/2018/11/28/docker/4.png)

1. 列出镜像

```
[root@localhost ~]# docker images
```

![](https://tycoding.cn/2018/11/28/docker/5.png)

* REPOSITORY: 镜像所在的仓库名称
* TAG： 镜像标签
* IMAGE ID：镜像ID
* CREATED：镜像的创建日期（不是获取该镜像的日期）
* SIZE：镜像大小

这些镜像都储存在Docker宿主机的`/var/lib/docker`目录下。

2. 搜索镜像

```
[root@localhost ~]# docker search 镜像名称
```

![](https://tycoding.cn/2018/11/28/docker/6.png)

3. 从Docker Hub拉取镜像

去Docker Hub官网查找所需的Docker镜像：[https://hub.docker.com/explore/](https://hub.docker.com/explore/) ，然后通过以下命令在线pull：

```
[root@localhost ~]# docker pull 镜像名称
[root@localhost ~]# docker pull 镜像名称:版本
```

因为官方提供的Docker镜像加速服务很慢，我们可以配置`ustc`的镜像。输入以下命令配置`ustc`镜像：

```
[root@localhost ~]# vi /etc/docker/daemon.json
```

没有就创建，向其中写入：

```
{
        "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```

重启Docker服务：

```
[root@localhost ~]# systemctl restart docker
```



4. 删除镜像

删除指定镜像

```
[root@localhost ~]# docker rmi $IMAGE_ID  #这个$IMAGE_ID数值可以根据`docker images`命令查看
```

删除所有镜像

```
[root@localhost ~]# docker rmi `docker images -q`
```

<br/>

## Docker容器操作

1. 查看正在运行的容器

```
[root@localhost ~]# docker ps
```

2. 查看所有的容器（启动过的历史容器）

```
[root@localhost ~]# docker ps -a
```

3. 查看最后一次运行的容器

```
[root@localhost ~]# docker ps -|
```

4. 查看停止的容器

```
[root@localhost ~]# docker ps -f status=exited
```

5. 删除容器

删除指定容器：

```
[root@localhost ~]# docker rm $CONTAINER_ID/NAME
```

删除所有容器：

```
[root@localhost ~]# docker rm `docker ps -a -q`
```



<br/>

## 创建和启动容器

**创建容器常用的参数说明**

1. 创建容器命令：`docker run`
2. `-i`: 表示运行容器
3. `-t`: 表示容器启动后会进入其命令行，加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。
4. `—name`: 为创建的容器命名
5. `-v`: 表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个`-v`做多个目录或文件映射。注意：最好做目录映射，在宿主机上修改，然后共享到容器上。
6. `-d`: 在`run`后面加上`-d`参数，则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加`-i` `-t`两个参数，创建后就会自动进去容器）。
7. `-p`： 表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个`-p`做多个端口映射。

### 交互式容器

创建一个交互式容器并取名为`mycentos5`:

```
[root@localhost ~]# docker run -it --name=mycentos5 centos:7 /bin/bash
[root@53f33e279914 /]#
```

此时我们可以新建一个连接，通过`docker ps`命令看到刚才创建的容器正在启动中：

![](https://tycoding.cn/2018/11/28/docker/7.png)

通过`exit`命令可以退出当前容器：

```
[root@53f33e279914 /]# exit
exit
[root@localhost ~]# 
```

此时再通过`docker ps`命令查看刚才启动的容器也停止了。

### 守护式容器

输入以下命令创建一个名字为`mycentos6`的容器：

```
[root@localhost ~]# docker run -di --name=mycentos6 centos:7
```

创建后这个容器会在后台运行，而不是直接进入到这个容器中。可以通过`docker ps`命令查看。

登录守护式容器：

```
[root@localhost ~]# docker exec -it mycentos6 /bin/bash
```

可以通过`exit`命令退出，但是容器不会停止。

### 停止和启动容器

停止正在运行的容器

```
[root@localhost ~]# docker stop 容器名称
```

启动已运行过的容器：

```
[root@localhost ~]# docker start 容器名称
```

<br/>

## 其他操作

### 文件拷贝

将文件拷贝到容器内可以用如下命令：

```
[root@localhost ~]# docker cp 需要拷贝的文件或目录 容器名称:容器目录
```

将文件从容器中拷贝出来

```
[root@localhost ~]# docker cp 容器名称:容器目录 需要拷贝的文件或目录
```

### 目录挂载

在创建容器的时候，将宿主机的目录和容器内的目录进行映射，这样我们就可以通过修改宿主机某个目录的文件从而影响容器。格式为：

```
[root@localhost ~]# docker run -di -v 宿主机目录:容器目录
```

### 查看容器

```
[root@localhost ~]# docker inspect mycentos5
```

<br/>

# 部署应用

## Mysql部署

1. 拉取MySQL镜像

```
[root@localhost ~]# docker pull mysql:5.7
```

2. 创建MySQL容器

```
[root@localhost ~]# docker run -di --name docker_mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.7
```

如上我们创建一个名称为`docker_mysql`的MySQL5.7版本的守护式容器，且配置MySQL登录密码是`root`。

* `-p` 代表端口映射，格式为 `宿主机映射端口:容器运行端口`
* `-e` 代表添加环境变量， `MYSQL_ROOT_PASSWORD`是root用户的登录密码

3. 进入MySQL容器，登录MySQL

```
[root@localhost ~]# docker exec -it docker_mysql /bin/bash
```

登录MySQL

```
mysql -u root -p
```

4. 远程连接MySQL

![](https://tycoding.cn/2018/11/28/docker/8.png)

## Tomcat部署

1. 拉取Tomcat-8 && JDK-8 镜像

```
[root@localhost ~]# docker pull tomcat:8-jre8
```

2. 部署Web应用

为了更好的演示Docker部署Tomcat的使用方式，我们可以先将需要部署的web项目发送到服务器的某个路径下，我这里在`/root/`目录下创建了`/root/site/`目录作为项目的根目录，在其中创建`index.html`文件并写入：

```
<html>
<head>
<title>Hello</title>
</head>
<body>
<h2>Hello Docker-Tomcat!</h2>
</body>
</html>
```

2. 创建Tomcat容器

```
[root@localhost ~]# docker run -di --name=docker_tomcat -p 9000:8080 -v /root/site/:/usr/local/tomcat/webapps/ROOT --privileged=true tomcat:8-jre8
```

以上就创建一个Tomcat容器，其容器名称Wie`docker_tomcat`，`-di`表示是一个守护式容器；`-p 9000:8080`表示此容器端口映射为`9000->8080`，即对外的端口是9000，映射到容器里Tomcat服务器的端口8080，`--privileged`是以root权限运行。

![](https://tycoding.cn/2018/11/28/docker/9.png)

通过命令看到，当我们启动了容器，其中的Tomcat服务器也自动启动了。当然对于部署Nginx或MySQL的Docker容器，当启动容器时都会启动对应的服务。

**注意**

上面我们指定了宿主机的`/root/site/`目录映射到`docker_tomcat`容器的`/usr/local/tomcat/webapps/ROOT`目录，为什么是这个目录呢？

Docker虚拟化，它会在内部虚拟一个操作系统，是在其宿主机内核上的一层空间，所有有一定的目录结构，我们可以通过`docker exec -it docker_tomcat /bin/bash`命令进入到`docker_tomcat`容器内部，通过`ls`命令查看目录会发现其Tomcat服务器确实安装在`docker_tomcat`容器的`/usr/local/tomcat`目录下。

最后，我们在浏览器上访问：`http://192.168.148.132:9000/`，即发现页面展示了我们刚才在`/root/site/`目录下创建的`index.html`网页：

![](https://tycoding.cn/2018/11/28/docker/10.png)

所以，如果你想要修改容器中Tomcat的端口号，直接进入容器的`/usr/local/tomcat/conf/server.xml`自改即可。

<br/>

## Nginx部署

1. 拉取Nginx镜像

```
docker pull nginx
```

2. 创建Nginx容器

```
docker run -di --name=docker_nginx -p 80:80 nginx
```

3. 测试

![](https://tycoding.cn/2018/11/28/docker/11.png)

在浏览器上访问：`http://192.168.148.132/`可以进入到Nginx的欢迎页。



## 其他

以上我们介绍了MySQL、Tomcat、Nginx容器的创建，对于其他的服务，如Redis等操作基本相同，不再阐述。

<br/>

# 备份与迁移

1. 容器保存为镜像

可以通过以下命令将我们已创建（配置好的）容器打包为镜像，这样我们以后就能用该镜像再次创建新的容器了：

```
docker commit docker_tomcat my_tomcat
```

`docker_tomcat`是容器名称；`my_tomcat`是新的镜像名称。



2. 镜像备份

通过以下命令可以将镜像打包为tar文件：

```
docker save -o my_tomcat.tar my_tomcat
```

`-o`输出到的文件



3. 镜像恢复与

当我们删除了`docker_tomcat`镜像后，可以通过以下命令将刚才打包备份的`.tar`镜像文件恢复成一个Docker镜像：

```
docker load -i my_tomcat.tar
```

`-i`输入的文件。
