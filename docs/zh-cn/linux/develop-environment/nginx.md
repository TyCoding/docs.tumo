![](nginx/logo.jpg)

Nginx是一款高性能HTTP服务器、反向代理服务器及电子邮件（IMAP、POP3）代理服务器，官方测试nginx能够支持5万并发连接。

<!--more-->

**Nginx应用场景：**

1. http服务器。Nginx是一个http服务器，可以独立提供http服务。可以做网页静态服务器。
2. 虚拟主机。可以实现在一台服务器虚拟出多个网站。
3. 反向代理，负载均衡。当网站的并发量过大需要配置服务器集群时可以使用Nginx做反向代理。并且多台服务器可以平均分摊负载。

<br/>

# 写在前面

**本文章默认使用的软件：**

* Centos7
* nginx-1.12.2
* Jdk8
* Tomcat8

**开发工具：**

* VMware
* SecureCRT

如果第一次在虚拟机上安装Linux系统，我建议你看一下这篇文章：[FastDFS系统搭建](https://tycoding.cn/2018/08/29/fastdfs/)，其中详细记录了虚拟机如何安装CentOS和联网。本文默认你已经安装了Centos7。

<br/>

# 安装Nginx

<br/>

## 环境准备

1. 安装gcc的环境

```
yum install gcc-c++
```

2. PCRE（perl库，包括perl兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式）

```
yum install -y pcre pcre-devel
```

3. Zlib（zlib提供了很多压缩和解压的方式，nginx使用http包的内容进行gzip）

```
yum install -y zlib zlib-devel
```

4. OpenSSL（一个强大的安全套接字层密码库，为nginx支持的https(即在SSL协议上传输http)协议服务）

```
yum install -y openssl openssl-devel
```

<br/>



## 安装

在本地电脑下载nginx的压缩包（我这里是`nginx-1.12.2.tar.gz`），然后打开SecureCRT连接服务器，使用命令`gz`将本地的文件上传到Linux服务器上。

> 注：如果在SecureCRT上输入`gz`显示command not found，是因为Linux虚拟机上没有安装`lrsz`服务。

执行：

```
yum -y indtall lrzsz
```

然后可以输入`rz`命令选择要上传的文件。默认上传到当前路径上。

1. **解压：**

```
tar zxvf nginx-1.12.2.tar.gz
```

2. **创建Makefile文件，执行命令：**

```
[root@localhost ~]# cd nginx-1.12.2
[root@localhost nginx-1.12.2]# ./configure
```

![](https://tycoding.cn/2018/11/27/nginx/1.png)

完成后可以看到Makefile文件：

![](https://tycoding.cn/2018/11/27/nginx/2.png)

**拓展**

Makefile是一种配置文件，Makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译…。Makefile就像一个Shell脚本一样。

3. **编译**

执行命令：

```
[root@localhost nginx-1.12.2]# make
```

4. **安装**

执行命令：

```
[root@localhost nginx-1.12.2]# make install
```

到此，Nginx安装已经完成。

<br/>

# Nginx启动与访问

上面我本编译安装的Nginx其实默认被安装在CentOS7系统的`/usr/local/nginx`目录。

![](https://tycoding.cn/2018/11/27/nginx/3.png)

**启动Nginx**

```
[root@localhost nginx]# cd sbin
[root@localhost sbin]# ./nginx
```

此时已经启动成功Nginx，可以直接在浏览器上输入虚拟机IP（可通过`ip addr`命令查看），即可访问到Nginx欢迎界面，如果显示未连接，可以：

1. 先使用本机的终端工具`ping`虚拟机IP地址看是否成功

```
ping ip
```

如果ping成功，请看下一步，如果失败，请检查虚拟机是否联网

2. 关闭CentOS7的防火墙，开放80端口

```
[root@localhost sbin]# systemctl stop firewalld.service #停止firewall
[root@localhost sbin]# systemctl disable firewalld.service #禁止firewall开机启动
[root@localhost sbin]# firewall-cmd --state #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
```

一般情况下就能访问到了：

![](https://tycoding.cn/2018/11/27/nginx/4.png)

**查看Nginx进程**

```
[root@localhost sbin]# ps aux|grep nginx
```

**关闭Nginx**

```
[root@localhost sbin]# ./nginx -s stop
```

或者

```
[root@localhost sbin]# ./nginx -s quit
```

**重启Nginx**

```
[root@localhost sbin]# ./nginx -s reload
```

**检查Nginx配置文件是否正确**

```
[root@localhost sbin]# ./nginx -t
```

<br/>



# 部署静态网站

经过上面的操作，我们已经正常启动了nginx，那么如何将我们的静态项目部署到服务器的Nginx上呢？

为了模拟操作，我这里只部署一个`index.html`网页为例：

1. **上传静态网站**

```
# 回到根目录下
[root@localhost sbin]# cd ../

# 创建文件夹`my`，视为我们的项目文件夹
[root@localhost nginx]# mkdir my

# 在文件夹`my`下创建一个`index.html`网页
[root@localhost nginx]# cd my
[root@localhost my]# vi index.html
```

写入
​```
<html>
<head>
<title>Hello</title>
</head>
<body>
<h2>Hello Nginx!</h2>
</body>
</html>
```

2. **修改Nginx的配置文件**

修改`/usr/local/nginx/conf/nginx.conf`文件：

```
[root@localhost my]# cd ../conf
[root@localhost conf]# vi nginx.conf
```

在`http {}`这个节点下新创建一个`server {}`节点：

```
 server {
     listen 81;
     server_name localhost;
     location / {
         root my;
         index index.html;
     }
 }
```

如此，我们已经将81端口绑定了`/nginx`文件夹下的名称为`my`的项目。重启Nginx，访问：`192.168.148.132:81`即可以访问到我们刚才新创建的网页：`index.html`。

<br/>

## 绑定域名

**域名**是由一串用“.”分隔的字符逐层的Internet上某一台计算机或计算机组的名称，用于在数据传输时表示计算机的电子方位。域名是一个IP地址的“面具”。域名的目的是便于记忆和沟通的一组服务器的地址。域名按照**域名系统DNS**的规则流程组成，在DNS中注册的任何名称都是域名。域名用于各种网络环境和应用程序特定的命名和寻址目的。通常，域名表示互联网协议（IP）资源。

一个域名对应一个IP地址，一个IP地址可以被多个域名绑定。

为了模拟，我们可以在本地hosts文件中配置域名和IP映射关系，这样就不用走DNS服务器了。

因为hosts文件内容不能直接修改，需要把hosts文件拷贝出来然后修改后再替换进去就行了（我这里使用的MACOS系统）。

1. **修改hosts**

```
cp /private/etc/hosts ~/Desktop/
vi ~/Desktop/hosts
```

添加：

```
192.168.148.132 www.tumo.xixi
```

然后替换原来的

```
cp ~/Desktop/hosts /private/etc/
```

2. 修改Nginx配置文件

```
[root@localhost conf]# vi /usr/local/nginx/conf/nginx.conf
```

为了模拟效果，我们可以先把之前新增的`server {}`节点81端口改为80端口：

```
 server {
     listen 80;
     server_name localhost;
     location / {
         root my;
         index index.html;
     }
 }
```

重启Nginx：

```
[root@localhost conf]# cd ../sbin
[root@localhost sbin]# ./nginx -s stop
[root@localhost sbin]# ./nginx
```

访问`192.168.148.132`发现还是Nginx的Welcome页面，因为nginx.conf默认配置的80端口就是指向Nginx欢迎页，且默认的`server_name`就是`localhost`。那么想实现不同的域名访问不同的资源且还必须是80端口，就需要绑定域名：

修改`nginx.conf`

```
 server {
     listen 80;
     server_name tumo.xixi;
     location / {
         root my;
         index index.html;
     }
 }
```

如上，很简单，只需要把`server_name`改为我们要绑定的域名地址就好了，然后重启Nginx，在浏览器上访问`tumo.xixi`就展示我们之前创建的`index.html`，而输入`192.168.148.132`访问的还是Nginx的欢迎页，这就实现了域名的绑定。多个域名绑定同一个IP地址，但是不同的域名指向了不同的资源地址。

<br/>

# Nginx反向代理

> 什么是反向代理？

反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器，并从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

> 正向代理

![](https://tycoding.cn/2018/11/27/nginx/5.png)

**正向代理**，主要针对客户端。当用户通过PC向Internet发送请求时，可以通过一个代理服务器来统一处理请求并转发给Internet，比如一个教室的所有学生机都需要通过老师的教师机才能实现联网，那么这个教师机就相当于一个代理服务器，负责将PC的网络请求转发给Internet，然后Internet将相应的数据再通过代理服务器转发给不同的PC。

> 反向代理

![](https://tycoding.cn/2018/11/27/nginx/6.png)

**反向代理**的过程则刚好相反，主要针对服务器。当用户通过网络请求不同的资源，而这些资源被分布在不同的服务器上，那么不同的请求就应该指向对应不同的服务器，那么就需要一个中介：反向代理服务器。通过反向代理服务器将不同的资源请求信息发送给不同的服务器，然后服务器将不同的信心都返回给反向代理服务器，最后通过反向代理服务器将这些结果信息展示在Internet上。

<br/>

## 配置反向代理

这里我们以一个非常实用的案例来演示如何配置Nginx的反向代理实现不同的域名访问不同的页面。

1. 在服务器上安装JDK8和Tomcat8

通过`rz`命令将本地的JDK和Tomcat安装包上传到服务器。因为Tomcat解压即可用，我们这里记录一下如何安装JDK：

```
# 解压
tar zxvf jdk-8u191-linux-x64.tar.gz

# 配置JDK环境
vi /etc/profile
```

在`profile`文件的结尾处添加如下环境配置：

```
export JAVA_HOME=/root/jdk1.8.0_191    
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$PATH:$JAVA_HOME/bin
```

`:wq!`保存并退出，输入命令：

```
[root@localhost ~]# source /etc/profile
```

更新配置。然后输入`java -version`显示则说明JDK配置成功。

解压Tomcat，在其`bin`目录下执行命令`./startup.sh`启动Tomcat服务器，然后我们再浏览器上访问：`http://192.168.148.132:8080/`显示Tomcat的欢迎页面则配置成功。

2. 拷贝项目到Tomcat服务器下

为了模拟，我们在`tomcat/webapps/ROOT/`下创建一个`index.html`网页：

```
[root@localhost ROOT]# vi index.html
```

并写入：

```
<html>
<head>
<title>Hello</title>
</head>
<body>
<h2>Hello Tomcat!</h2>
</body>
</html>
```

保存并退出，再次访问`http://192.168.148.132:8080/`发现展示的即是我们刚创建的页面

![](https://tycoding.cn/2018/11/27/nginx/7.png)

3. 配置反向代理

之前讲了**反向代理**的概念，那么很实用的一个例子就是，当我们想在购买的阿里云服务器上部署两个项目，一个项目是用Nginx部署的静态网站（占用80端口），另一个项目部署在Tomcat服务器上（占用8080端口）。

这时，我们固然是可以通过：`ip:8080`来访问我们的Web项目的，但是请求路径上显示了端口好久显得很别扭，为此，我们通过Nginx的反向代理就能解决：

* 3.1在本地配置一个二级域名映射

修改本机的hosts文件，添加：

```
192.168.148.132 site.tumo.xixi
```

这时我们访问`site.tumo.xixi`默认进入nginx的欢迎页，这是正确的。

![](https://tycoding.cn/2018/11/27/nginx/8.png)

注意：此时的`site.tumo.xixi`是`tumo.xixi`的一个二级子域名。

* 3.2 修改服务器Nginx的配置文件，添加如下配置

```
upstream site {
        server 192.168.148.132:8080;
}
server {
        listen 80;
        server_name site.tumo.xixi;
        location / {
       		proxy_pass http://site;
        	index index.html;
        }
}
```

注意，其中`server {}`节点下的`proxy_pass`表示反向代理的地址，其中`http://site`这个`site`其实是指向上面`upstream site {}`节点的`site`名称，因此要保持两者名称一致。但实际上我们不采用这种方式也能实现反向代理：

```
# upstream site {
#         server 192.168.148.132:8080;
# }
server {
        listen 80;
        server_name site.tumo.xixi;
        location / {
       		# proxy_pass http://site;
        	proxy_pass http://192.168.148.132:8080;
        	index index.html;
        }
}
```

![](https://tycoding.cn/2018/11/27/nginx/9.png)

两者的区别就是第一种方式通过*指向*的方式可以配置更多，必须实现**负载均衡**就需要在`upstream site {}`节点下配置。

<br/>

# Nginx配置负载均衡

> 什么是负载均衡？

**负载均衡（Load Balance）**，其意思就是分摊到多个操作单元上进行执行，从而共同完成工作任务。

**负载均衡** 是建立在现有网络结构上，提供一种廉价有效透明的方法扩展网络设备和服务器的带宽，增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性。



1. **模拟负载均衡**，我们可以提供多个Tomcat服务器，采用不同的端口区分。

为了模拟**负载均衡**效果，可以copy 2份虚拟机上的Tomcat服务器，命名为tomcat-2，tomcat-3:

```
[root@localhost ~]# cp ~/apache-tomcat-8.5.33 ~/tomcat-2
[root@localhost ~]# cp ~/apache-tomcat-8.5.33 ~/tomcat-3
```

修改端口号分别为8180，8280。我们主要修改`/tomcat/conf/server.xml`配置文件中的`<Server port="8005"`和`<Connector port="8080"`这两个节点的端口

```
# 修改tomcat-2服务器的`/conf/server.xml`参数
<Server port="8006" shutdown="SHUTDOWN">
	    <Connector port="8180" protocol="HTTP/1.1"	

# 修改tomcat-3服务器的`/conf/server.xml`参数
<Server port="8007" shutdown="SHUTDOWN">
	    <Connector port="8280" protocol="HTTP/1.1"	
```

如上，我们在虚拟机上配置了三个Tomcat服务器，分别使用8080，8180，8280端口。

2. **配置负载均衡**

修改Nginx下的配置文件

```
[root@localhost ~]# vi /usr/local/nginx/conf/nginx.conf
```

修改之前配置的`upstream site {}`节点：

```
 upstream site {
     server 192.168.148.132:8080;
     server 192.168.148.132:8180;
     server 192.168.148.132:8280;
}
server {
    listen 80;
    server_name site.tumo.xixi;
    location / {
    	proxy_pass http://site;
    	index index.html;
    }
}
```

这样我们就给Nginx配置了3台服务器，都指向了`site.tumo.xixi`这个域名地址，那么访问这个地址时同时会访问这三台服务器，也就是三台服务器平均分摊访问压力。

为了更好的实现效果，我们可以依次修改tomcat-2和tomcat-3服务器的`/webapps/ROOT/index.html`网页显示数据，更容易区分每次访问的是哪台服务器。

然后运行这三个Tomcat服务器，在浏览器上访问：`site.tumo.xixi`，多次刷新页面，每次访问的都是不同的页面，且依次是配置负载均衡的三台服务器次序。

如果你想让某个服务器承担更大的压力，可以为其设置权重：

```
 upstream site {
     server 192.168.148.132:8080;
     server 192.168.148.132:8180 weight=2;
     server 192.168.148.132:8280;
}
```