---
title: Typora+PicGo实现文章图片上传到图床
date: 2024-09-27 18:06:47
tags:
categories:
  - 博客
---



## Typora+PicGo自动上传到GitHub图床

### 1. 创建GitHub仓库作为图库

首先在GitHub新建一个仓库，可以在创建仓库时勾选上添加README.md

将以这个仓库作为图床。

本博客选用CloudFlare CDN堆GitHub图床进行加速。

### 2. 下载PicGo并进行图床设置

PicGo是用来将图片上传到图床中的工具，它有支持很多图床，而且还有很多插件，通过插件，你可以选择用B站、Gitee等作为图床，或许就无需进行CDN访问加速也能在国内获得不错的访问加速

PicGo插件：[github.com/PicGo/Awesome-PicGo](https://github.com/PicGo/Awesome-PicGo) （在PicGo客户端也能访问到）

**PicGo的下载网址：**[Releases · Molunerfinn/PicGo (github.com)](https://github.com/Molunerfinn/picgo/releases)

选择一个稳定版本进行安装即可

<img src="https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409271801423.png" alt="image-20240927145055085" style="zoom:50%;" />

启动PicGo，在其中进行图床设置。其中Token是你GitHub的个人令牌，如果你没有，在个人GitHub主页点击头像--》Settings(设置)

下滑找到 Developer seetings(开发者设置)，点击Generate new token(生成新令牌)，勾选下图所示的repo仓库权限，下滑到最下面Generate即可，然后将Token复制到上图的设定Token位置

![image-20240927145537275](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409271801704.png)

你可以为Token设置一个过期失效，然后在Token失效后自己在PicGo中手动更新Token即可

一会还需要设置域名方便访问，这在第3部分中[图床域名访问](#section1)

<a id="section1"></a>

### 3. 图床CDN加速

#### 3.1 使用jsDelivr加速

jsDelivr是免费且开源的内容分发网络，你可以通过它加速和托管前端资源

jsdelivr的用法非常简单,你只需要在自定义域名中按照如下规则添加即可：

https://cdn.jsdelivr.net/gh/GitHub用户名/仓库名

>https://cdn.jsdelivr.net 固定的
>
>gh表示来源为 GitHub
>
>后面跟上你的用户名加仓库名即可

#### 3.2 使用CloudFlare

[CloudFlare官网](https://www.cloudflare.com/)

没有注册的可以注册一下，除了图床访问加速，我们也可以通过它对博客网站进行CDN加速

在主页找到Pages，点击创建。

![image-20240927143031302](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409271800722.png)



在创建应用程序中选择连接到Git，并和前面在GitHub中新建的仓库关联，之后全部默认即可

![image-20240927143124828](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409271800799.png)



部署完成后你就可以在Pages页中看到看到这个仓库的链接，你可以通过这个域名+你的资源路径访问到GitHub仓库中的资源

![image-20240927144327697](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409271802744.png)

将这个域名复制到PicGo图床设置的自定义域名中，就可以通过网络获取到仓库中的图片了

### 4. Typora自动上传

最后一步同时也是注入灵魂的一步，在Typora中配置本地图片自动上传并更新URL，这样就可以在本地编写博客时，直接将图片上传到图床中，博客写完后，直接提交到博客仓库即可，无需手动修改图片链接。

在Typora中点击文件--》偏好设置

![Snipaste_2024-09-27_17-12-03](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202409271758308.png)

配置完成，现在Typora可以将本地和网络上的图片自动上传到图床了。



——————————————————

>参考：
>
>[Typora+B站，无敌的高速图床，且高度自定义图片参数，只需要两步_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV13K4y197j7/?spm_id_from=333.880.my_history.page.click&vd_source=3b2548ca76d1728cf98fff60e44d3e0a)
>
>[手把手教你 Typora + PicGo 配置图床，提高写作效率！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/469339158#:~:text=手把手教你 Typo)
>
>[使用 jsDelivr CDN 对 Github 图床进行加速，带给你如丝滑般的图片体验！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/138582151#:~:text=学习 jsDeliv#:~:text=学习 jsDeliv)
