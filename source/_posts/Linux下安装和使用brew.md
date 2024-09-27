---
title: Linux下安装和使用brew
date: 2024-09-25 05:05:46
tags:
categories: 
  - Linux
---
## 下载HomeBrew和使用

Brew是一个非常流行的开源包管理器，可以理解为一个命令行版本的应用商店。它是相对安全的，如果你知道自己正在下载什么。起码目前 Homebrew 上不存在恶意包，关于它的介绍，可以看：[Homebrew 使用详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/30704752)

本文主要用于使用CentOS Linux系统的同学在一部全新的Linux虚拟机上安装Brew，同时也方便本人回顾

### 1 安装必要的基础软件

本文安装的是 Homebrew 3.0.9 , 官方文档中描述存在如下依赖:

```
* GCC 4.7.0 or newer
* Linux 2.6.32 or newer
* Glibc 2.13 or newer
* 64-bit x86_64 CPU
```

可以跳过 GCC 的检查，之后使用 Homebrew 进行安装。

- 检查内核版本

```shell
uname -a 
#输出
Linux myhost 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux 
```

- 检查 Glibc 版本

```shell
ldd --version 
#输出
ldd (GNU libc) 2.17 Copyright (C) 2012 Free Software Foundation, Inc. This is free software; see the source for copying conditions.  There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. Written by Roland McGrath and Ulrich Drepper.
```

- 安装基础软件

***Development tools*** 是一个可以给开发编译运维的配置基本初始环境的工具。它包含一系列组合包，可以在初始化脚本配置的时候更方便、快捷、高效。

```shell
yum groupinstall -y ‘Development Tools’
yum install -y curl file git 
```

>当运行yum groupinstall -y ‘Development Tools’时可能会提示无该安装包，这可能是是由于使用的yum源缺乏该安装包导致的
>
>解决方法是：
>
>```shell
>//安装epel源
>sudo yum install epel-release
>
>//更新
>yum update
>```
>
>🔔：yum源好像已经无法使用，可以更新yum源 [无法使用yum指令怎么办](#section2)

### 2 升级 git 和 curl

HomeBrew的安装需要使用到git和支持https2的curl版本，由于CentOS默认版本过低，需要更新git和curl的版本

- 升级 git 版本，不低于 2.7.0

 🔔 🔔 🔔 🔔

:exclamation: **原帖这里的更新方法已经失效**

这里使用的是：

```shell
yum install -y https://repo.ius.io/ius-release-el7.rpm
yum install -y epel-release 

# re-install git:
yum erase -y git*
yum install -y git-core
```

>参考：[linux 升级git_linux升级git版本-CSDN博客](https://blog.csdn.net/qq_26545503/article/details/125617944)

查看版本

```shell
git --version 
#输出
git version 2.36.6
```

- 升级 curl 版本，不低于 7.41.0

默认版本的curl7.29不支持http2

 🔔 🔔 🔔 🔔

报错为：`无法打开 http://www.city-fan.org/ftp/contrib/yum-repo/`

可以试试科学上网

**这里的方法是：**

>一、添加city-fan.org的源
>
>```shell
>vim /etc/yum.repos.d/city-fan.org.repo  # 编辑文件添加如下
>```
>
>二、将以下内容复制过去
>
>```shell
>[city-fan.org]
>name=city-fan.org repository for Red Hat Enterprise Linux (and clones) $releasever ($basearch)
>#baseurl=http://mirror.city-fan.org/ftp/contrib/yum-repo/rhel$releasever/$basearch
>mirrorlist=http://mirror.city-fan.org/ftp/contrib/yum-repo/mirrorlist-rhel$releasever
>enabled=1
>gpgcheck=0
>gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-city-fan.org
>
>[city-fan.org-debuginfo]
>name=city-fan.org debuginfo repository for Red Hat Enterprise Linux (and clones) $releasever ($basearch)
>#baseurl=http://www.city-fan.org/ftp/contrib-debug/rhel$releasever/$basearch
>mirrorlist=http://www.city-fan.org/ftp/contrib-debug/mirrorlist-rhel$releasever
>enabled=1
>gpgcheck=0
>gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-city-fan.org
>
>[city-fan.org-source]
>name=city-fan.org source repository for Red Hat Enterprise Linux (and clones) $releasever
>#baseurl=http://mirror.city-fan.org/ftp/contrib/yum-repo/rhel$releasever/source
>mirrorlist=http://mirror.city-fan.org/ftp/contrib/yum-repo/source-mirrorlist-rhel$releasever
>enabled=1
>gpgcheck=0
>gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-city-fan.org
>```
>
>三、更新curl
>
>```shell
>yum update curl
>```
>
>查看版本
>
>```shell
>curl --version 
>#输出
>curl 8.2.1 
>```
>
>————————————————
>
>原文链接：[centos7 升级curl-8.2.1 支持http2 （yum update） - 诗里刻画的影子 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zinging/p/17934351.html)
>
>[Linux的curl相关文档  -  Linux - everything curl (haxx.se)](https://ec.haxx.se/install/linux.html)

### 3. 安装 Homebrew

- 切换到非 root 用户

```shell
su feng 
```

>在此之前，请确认你的非root用户具有使用sudo命令的权限，并为该用户配置sudo(super user do)认证
>
>如果想要让用户具有超级用户权限请点击：[跳转到更改用户权限](#section1)

- 安装 Homebrew

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)" 
```

> 以上下载是github的链接，国内下载速度比较慢，可以用：
>
> ```shell
> /bin/bash -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
> ```
>
> 若采用此方法下载，有可能还需要安装zsh（如已经安装则跳过此步骤）
>
> centos使用下面命令安装zsh
>
> ```shell
> sudo yum update
> sudo yum install zsh
> ```
>
> 如果是debian或者ubuntu，执行如下命令
>
> ```shell
> sudo apt update
> sudo apt install zsh
> ```

(实测通过GitHub下载速度也不慢)

>下载过程中可能会出现：
>
>```shell
>error: RPC 失败。curl 92 HTTP/2 stream 5 was not closed cleanly: CANCEL (err 8)
>error: 预期仍然需要 2109 个字节的正文
>fetch-pack: unexpected disconnect while reading sideband packet
>fatal: 过早的文件结束符（EOF）
>fatal: fetch-pack：无效的 index-pack 输出
>Failed during: /bin/git fetch --force origin
>```
>
>这是由于网络状况差导致的
>
>请尝试更换网络或等待网络状态正常时继续进行

- 添加 brew 命令到 PATH

```shell
test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
test -r ~/.bash_profile && echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.bash_profile
echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile
```

直接将 `.linuxbrew/bin` 添加到 PATH 会导致安装新工具时，make 报错。

- 查看版本

```shell
brew  --version
#输出
Homebrew 4.3.24
Homebrew/homebrew-core (git revision 1ce004f2f0b; last commit 2021-03-28)
```

- [可选]添加 brew 到 Zsh

```shell
test -r ~/.zshrc && echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.zshrc
```

### 4. 使用

- 安装 GCC

```shell
brew install gcc

==> Downloading https://linuxbrew.bintray.com/bottles/isl-0.23.x86_64_linux.bottle.tar.gz
==> Downloading from https://akamai.bintray.com/46/46537a01a0cc82ea18528293b6bac03be7ab14b2fc3a7a28051f1b26e24f6c57?__gda__=exp=1616978202~hmac=b7727c9ae79b54bebc199a095c01e70251446ed4012bfb5900dd877e06229fd1&response-content-disposition=attachment%3Bfilename%3D%22isl-0
######################################################################## 100.0%
==> Downloading https://linuxbrew.bintray.com/bottles/gcc-10.2.0_4.x86_64_linux.bottle.tar.gz
==> Downloading from https://akamai.bintray.com/e6/e62efea399fd03d854ff871210b941e5c3277d821e9148badceeda883f53f016?__gda__=exp=1616978204~hmac=a7e19e02629bc1c462635349fe3c01039f581e2014153c704f72e54bc2d93c35&response-content-disposition=attachment%3Bfilename%3D%22gcc-1
#################                                                         24.9%
######################################################################## 100.0%
==> Installing dependencies for gcc: isl
==> Installing gcc dependency: isl
==> Pouring isl-0.23.x86_64_linux.bottle.tar.gz
🍺  /home/linuxbrew/.linuxbrew/Cellar/isl/0.23: 73 files, 6.6MB
==> Installing gcc
==> Pouring gcc-10.2.0_4.x86_64_linux.bottle.tar.gz
Warning: The post-install step did not complete successfully
You can try again using:
  brew postinstall gcc
==> Summary
🍺  /home/linuxbrew/.linuxbrew/Cellar/gcc/10.2.0_4: 1,481 files, 264.3MB
==> No outdated dependents to upgrade!
==> Checking for dependents of upgraded formulae...
==> Reinstalling 1 broken dependent from source:
gcc
```



- 安装 Kubectl

```shell
brew install kubectl

==> Downloading https://linuxbrew.bintray.com/bottles/kubernetes-cli-1.20.4_1.x86_64_linux.bottle.tar.gz
==> Downloading from https://akamai.bintray.com/c5/c5dd394d0bd150aff18ded31f29e52ece6dcbd064ebe46e7f746891822a12eff?__gda__=exp=1616978407~hmac=13c24ddb7a7e90118b1aa8184d2c4688f719a945c4f0d033352a7cae9c845887&response-content-disposition=attachment%3Bfilename%3D%22kuber
######################################################################## 100.0%
==> Pouring kubernetes-cli-1.20.4_1.x86_64_linux.bottle.tar.gz
==> Caveats
Bash completion has been installed to:
  /home/linuxbrew/.linuxbrew/etc/bash_completion.d
==> Summary
🍺  /home/linuxbrew/.linuxbrew/Cellar/kubernetes-cli/1.20.4_1: 246 files, 40.9MB
```



- 安装 Kind

```shell
brew install kind

==> Downloading https://linuxbrew.bintray.com/bottles/kind-0.10.0.x86_64_linux.bottle.tar.gz
==> Downloading from https://akamai.bintray.com/7a/7a5595da7f271219a53cbd95a5b729c581fb3e4f8ece6d449074cbe33f10d627?__gda__=exp=1616978455~hmac=754a3789f01e3c27ebdfa1b65e51392eaeacf61f878a9d607b78d342f52c5dfa&response-content-disposition=attachment%3Bfilename%3D%22kind-
######################################################################## 100.0%
==> Pouring kind-0.10.0.x86_64_linux.bottle.tar.gz
==> Caveats
Bash completion has been installed to:
  /home/linuxbrew/.linuxbrew/etc/bash_completion.d
==> Summary
🍺  /home/linuxbrew/.linuxbrew/Cellar/kind/0.10.0: 8 files, 9.4MB
```

---

————————————————

原文链接：[CentOS 7 下安装并配置 Homebrew – 陈少文的网站 (chenshaowen.com)](https://www.chenshaowen.com/blog/install-homebrew-in-centos-7.html)

## 使用brew安装nodejs

当我使用homebrew安装nodejs后，它要求我进行配置。提示如下：

```shell
==> Installing node@20
==> Pouring node@20--20.17.0.x86_64_linux.bottle.tar.gz
==> Caveats
node@20 is keg-only, which means it was not symlinked into /home/linuxbrew/.linuxbrew,
because this is an alternate version of another formula.

If you need to have node@20 first in your PATH, run:
  echo 'export PATH="/home/linuxbrew/.linuxbrew/opt/node@20/bin:$PATH"' >> /home/feng/.bash_profile

For compilers to find node@20 you may need to set:
  export LDFLAGS="-L/home/linuxbrew/.linuxbrew/opt/node@20/lib"
  export CPPFLAGS="-I/home/linuxbrew/.linuxbrew/opt/node@20/include"
==> Summary
🍺  /home/linuxbrew/.linuxbrew/Cellar/node@20/20.17.0: 2,068 files, 66.6MB
==> Running `brew cleanup node@20`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
==> Caveats
==> node@20
node@20 is keg-only, which means it was not symlinked into /home/linuxbrew/.linuxbrew,
because this is an alternate version of another formula.

If you need to have node@20 first in your PATH, run:
  echo 'export PATH="/home/linuxbrew/.linuxbrew/opt/node@20/bin:$PATH"' >> /home/feng/.bash_profile

For compilers to find node@20 you may need to set:
  export LDFLAGS="-L/home/linuxbrew/.linuxbrew/opt/node@20/lib"
  export CPPFLAGS="-I/home/linuxbrew/.linuxbrew/opt/node@20/include"

```

**其中的有用信息是:**

```shell
node@20 is keg-only, which means it was not symlinked into /home/linuxbrew/.linuxbrew,
because this is an alternate version of another formula.

If you need to have node@20 first in your PATH, run:
  echo 'export PATH="/home/linuxbrew/.linuxbrew/opt/node@20/bin:$PATH"' >> /home/feng/.bash_profile

For compilers to find node@20 you may need to set:
  export LDFLAGS="-L/home/linuxbrew/.linuxbrew/opt/node@20/lib"
  export CPPFLAGS="-I/home/linuxbrew/.linuxbrew/opt/node@20/include"
```

按照提示所说，运行

```shell
#这里由于用户名不同文件位置可能也有所不同，请参考Linux的提示
echo 'export PATH="/home/linuxbrew/.linuxbrew/opt/node@20/bin:$PATH"' >> /home/feng/.bash_profile
```

将以下内容添加到  ~/.bashrc中

```shell
vi ~/.bashrc
#将以下内容添加最后面
export LDFLAGS="-L/home/linuxbrew/.linuxbrew/opt/node@20/lib"
export CPPFLAGS="-I/home/linuxbrew/.linuxbrew/opt/node@20/include"
```

配置完成后，运行

```shell
node -v
#输出
bash: node: command not found...
```

**发现仍然不能使用node**

***解决方法：***

**检查 Node.js 是否安装**： 运行以下命令来确认 Node.js 是否在 Homebrew 中安装：

```bash
brew list | grep node
```

如果没有找到 Node.js，尝试重新安装：

```bash
brew install node@20
```

**添加 Node.js 到 `PATH`**： 如果 Node.js 已安装，但仍然无法找到，可能是 `PATH` 没有正确设置。你可以将 Node.js 的安装路径添加到你的 `PATH` 中。在 `~/.bashrc` 或 `~/.zshrc` 文件中添加以下行：

```bash
vi ~/.bashrc #或者vi ~/.zshrc
#将以下内容复制到文件中
export PATH="/home/linuxbrew/.linuxbrew/opt/node@20/bin:$PATH"
```

然后保存文件并运行：

```bash
source ~/.bashrc
```

或者如果你使用的是 Zsh：

```bash
source ~/.zshrc
```

**验证安装**： 再次检查 Node.js 是否可以正常使用：

```bash
node -v
#输出
v20.17.0
```

安装成功

* 如果你需要使用最新版的npm，请运行：

```bash
npm install -g npm@latest
```

* `npm`默认的官网源可能会比较慢，如果想要后续的下载速度快一些，可以通过下面的方式将源设置为淘宝源或阿里源。

```bash
npm config get registry # 查看源
#哪个国内镜像源都可以
npm config set registry https://registry.npmmirror.com # 修改为淘宝源
npm config set registry https://npm.aliyun.com #修改为阿里源
```

**文章中的源都具有时效性**

如果源不可用，可以自行查找最新的镜像源并更新

---

**以下内容不一定需要**

<a id="section1"></a>

## 更改普通用户权限

1.切换到root用户下

su -

2.添加sudo文件的写权限,命令是:
chmod u+w /etc/sudoers

3.编辑sudoers文件
vi /etc/sudoers
找到这行 root ALL=(ALL) ALL,在他下面添加xxx ALL=(ALL) ALL (这里的xxx是你的用户名)

ps:这里说下你可以sudoers添加下面四行中任意一条

    youuser            ALL=(ALL)                ALL  
    %youuser           ALL=(ALL)                ALL  
    youuser            ALL=(ALL)                NOPASSWD: ALL  
    %youuser           ALL=(ALL)                NOPASSWD: ALL  

第一行:允许用户youuser执行sudo命令(需要输入密码).
第二行:允许用户组youuser里面的用户执行sudo命令(需要输入密码).
第三行:允许用户youuser执行sudo命令,并且在执行的时候不输入密码.
第四行:允许用户组youuser里面的用户执行sudo命令,并且在执行的时候不输入密码.

4.撤销sudoers文件写权限,命令:
chmod u-w /etc/sudoers

这样普通用户就可以使用sudo了.
补充：

注意，在Ubuntu系统下，Unix操作系统并没有为root创建密码，需要使用sudo passwd root来为root用户配置密码，之后才可以登入。

## 无法使用yum指令

CentOS报错：Could not retrieve mirrorlist http://mirrorlist.centos.org/?release=7&arch=x86_64&repo=os&i

最终解决方法：**更新yum源**

具体参考这篇博客：[yum指令报错](https://blog.csdn.net/gnwu1111/article/details/140172717?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0-140172717-blog-103771510.235)

简单存个档，以防网页丢失：

>1、检查网络配置是否正常
>
>在linux下ping一下看看是不是网络链接正常。出现这种原因就是一般网络没链接好。那修改一下dns，找到/etc/sysconfig/network-scripts/ifcfg-ens33
>注意一下，ifcfg-ens33后面的数字是随机产生的
>
>将onboot改为yes，重新启动网络，service network restart，然后ping www.baidu.com如果通了的话，就证明链接成功。这样就可以正常yum update了

>2、检查有没有配置/etc/resolv.conf
>
>解决方法：
>
>vi /etc/resolv.conf
>
>:wq保存退出即可，之后再执行yum操作，成功！

<a id="section2"></a>

3、如果还不行 **更新yum源**
　　使用阿里源：	

​	1.备份当前的yum源

　　　　`mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup`

​	2.下载新的CentOS-Base.repo 到/etc/yum.repos.d/

```shell
　CentOS 5
 
　　　　wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-5.repo
 
　　　　或者
 
　　　　curl -o /etc/yum.repos.d/CentOS-Base.repo 
     
```

```shell
　　　　CentOS 6
 
　　　　wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
 
　　　　或者
 
　　　　curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo
```

```shell
　　　　CentOS 7
 
　　　　wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
 
　　　　或者
 
　　　　curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

  3. 清空并生成缓存

     ​	   yum clean all

​	  	yum makecache
　　🔔:　yum 会把下载的软件包和header存储在cache中(默认路径/var/cache/yum/)，而不自动删除。如果觉得占用磁盘空间，可以使用yum clean指令进行清除，更精确 的用法是yum clean headers清除header，yum clean packages清除下载的rpm包，yum clean all全部清除。

使用网易源：

　1.网易
　　　2.备份当前的yum源

`mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup`

 　3. 下载对应版本repo文件, 放入/etc/yum.repos.d/

🔔和前面阿里云的是一样的，只需要更改网址即可

>http://mirrors.163.com/.help/CentOS5-Base-163.repo	CentOS5
>
>http://mirrors.163.com/.help/CentOS6-Base-163.repo	CentOS6
>
>http://mirrors.163.com/.help/CentOS7-Base-163.repo	CentOS7

　4.运行以下命令生成缓存

​	  	yum clean all

​	  	yum makecache
————————————————

原文链接：https://blog.csdn.net/gnwu1111/article/details/140172717