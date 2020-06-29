# 前言

由于作者使用的MacOS系统，文档中截图和Windows系统可能有所区别，但大同小异。文档中使用的IDE为IntelliJ IDEA 2020.1.1 (Ultimate Edition)，如果读者使用的是Eclipse请自行百度Eclipse如何导入Maven项目。

# 安装JDK

因为项目中使用到了JDK 8的一些特性，所以本地JDK版本不能低于8。

JDK8官方下载地址：[https://www.oracle.com/technetwork/java/javase/downloads](https://www.oracle.com/technetwork/java/javase/downloads)。

# 安装MySQL

项目数据库采用MySQL社区版，版本为5.7.x。下载地址：[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)。未避免数据库导入出错，请保持和笔者相同的版本。

# 安装Maven

下载地址：[https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)。如果Maven官方镜像下载速度太慢，可以修改为使用阿里云镜像。步骤如下：

找到 `apache-maven-3.6.3/conf/settings.xml`文件，将如下代码：

```xml
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
  </mirrors>
```

修改为：

```xml
  <mirrors>
    <mirror>      
			<id>nexus-aliyun</id>    
			<name>nexus-aliyun</name>  
			<url>http://maven.aliyun.com/nexus/content/groups/public</url>    
			<mirrorOf>central</mirrorOf>      
	  </mirror>
  </mirrors>
```

# 安装IDEA

下载地址：[https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)。**Ultimate**版本免费试用30天（可以通过Github开源项目免费申请License）

# IDEA配置本地Maven

在IDEA顶部工具栏中选择`Preferences>Build,Execution,Deployment>Build Tools>Maven` 配置本地Maven的相关信息：

![image-20200531132647743](https://gitee.com/tytumo/pictures/raw/master/img/20200531132647.png)

# 解决本地访问GitHub太慢

[Mac 解决GitHub下载速度太慢问题](https://www.jianshu.com/p/238f8242e1a6)