---
title: Linux图形化界面安装
date: 2024-09-24 05:57:30
categories:
-Linux
---
## 如何在CentOS Linux中使用图形化界面

当自己安装Linux系统时，只能在命令行交互，不够直观

本处通过图形化界面 X Window System实现图形化界面显示

进入后命令行出现，首先登陆root用户获取权限才能进行配置。

```shell
su -
# 输入密码
```

* 先查看是否能上网。
  查看ip地址，看网卡编号。输入代码

```shell
ip addr show
```

* 查看网卡编号，网卡为ens33。

​	知道网卡编号后，查看网卡状态，输入命令

```shell
cat /etc/sysconfig/network-scripts/ifcfg-ens33
```

​	发现网卡关闭（ONBOOT=no），开启网卡，输入命令

```shell
ifup ens33
```

​	打开该网卡

​	开启网卡后，检查是否能上网，输入命令

```shell
ping www.baidu.com
```

> 按下crtl + c 结束

* 开始安装图形界面，先安装X Window System
  输入命令

```shell
yum groupinstall "X Window System"
```

​	安装过程有问你选择（y/n）一律

```shell
y
```

​	显示complete！，安装成功。

* 然后开始安装图形界面。
  输入命令

```shell
yum groupinstall -y "GNOME"
```

​	安装图形界面，有问选择（y/n）也是一律

```shell
y
```

​	直到complete！


​	然后输入进入图形界面指令

```shell
init 5
```

​	或者

```shell
startx
```

进入图形界面

至此，Centos7命令和图形界面安装及下载完成。

————————————————

原文链接：[CentOS 7 命令行界面与图形界面安装下载及图文注释（附下载链接）](https://blog.csdn.net/weixin_47903763/article/details/109011614)

