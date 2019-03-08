# CentOS-7.x 系统安装

由于正在学习的分布式项目中用到了`FastDFS`文件系统，因为各种原因就只好手动搭建环境。搭建过程确实很复杂，我也是折腾了好长时间才解决的，看了网上的教程，但很少有直接就能搭建成功得博文教程，所以这里自己写了一个完成的教程，并附带了所需的配置文件。

需要注意的是Linux系统、版本的问题，我这里使用的是CentOS7的版本，如果大家搭建请尽量保证版本一致。

<!--more-->

## 起步

```
1、下载Linux系统（以CentOS为例）
  [CentOS7-Minimal](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1804.iso)
  
2、安装SecureCRT
  因为实际的服务器并不存在桌面，所以我们安装Minimal版本的CentOS即可，但是在CentOS的黑窗口中操作并不方便，所以下载SecureCRT操纵服务器。
```

<br/>

### 安装CentOS

本例中用VMware安装的CentOS系统，在安装时需要注意一个问题：

![](https://tycoding.cn/2018/08/29/fastdfs/1.png)

如上图所示，在安装CentOS时不要创建新用户，设置个ROOT密码即可，这样登录进去系统后默认就是ROOT权限，避免了权限不够的问题。（登录用户名密码默认都是`root`）

**1、登录系统并配置连接网络**

输入命令：`cd /etc/sysconfig/network-scripts/ && ls`，编辑列表中的第一个文件（因为文件名称可能不相同）

![](https://tycoding.cn/2018/08/29/fastdfs/2.png)

将最后行的`no`改成`yes`即可

![](https://tycoding.cn/2018/08/29/fastdfs/3.png)

修改点击键盘的`i`键进入编辑模式，修改完成后按下ESC键，再输入`:wq!`保存退出即可；然后执行命令`service network restart`重启服务。

此时已经完成了CentOS的联网，输入命令`ip addr`查看服务器的IP地址（如果联网成功，会出现192.168.xx.xx类似这样的IP地址）：

![](https://tycoding.cn/2018/08/29/fastdfs/4.png)

这个IP地址即使此服务器的外网IP，即我们要用SecureCRT连接的服务器的IP

**2、使用SecureCRT连接服务器**

SecureCRT的使用方法不在说了，就是简单的创建一个连接，输入服务器的IP地址、账户、密码即可完成连接。

**3、关闭CentOS7的防火墙**

作为我们测试用的本地服务器，加一个防火墙实在是没有必要的，所以索性关闭防火墙。
输入命令：
```
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
firewall-cmd --state #查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
```