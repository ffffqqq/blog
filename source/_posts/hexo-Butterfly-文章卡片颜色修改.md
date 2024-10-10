---
title: hexo-Butterfly 文章卡片颜色修改
date: 2024-10-10 22:17:32
tags:
  - 博客教学
categories:
  - 博客
description: 在hexo butterfly主题下添加自定义格式，并设置文章盒子的颜色
cover: https://pic-bed1-9nl.pages.dev/imgs/index_imgs/post_cover2.jpg
---





# 引入自定义格式



1. 如果主题是通过GitHub源下载的，那么你的css文件位于

​	`项目根目录\theme\butterfly\source\css\`

​	直接在这下面新建css文件即可

2. 如果是通过npm包管理器下载，即 npm install ... ，那么css文件位于

​	`项目根路径\node_modules\hexo-theme-butterfly\source\css\`

​	你可以新增一个用于自定义的文件夹，命名为：_custom，这个文件夹用来存放自定义 css 格式。然后还需要在 \css 目录下的index.styl中添加如下：

```stylus
@import '_custom/*.css'	//导入_custom文件夹下所有css文件

// 如果需要引入外链:
@import 'https://cdn.jsdelivr.net/gh/username/repo/css/xxx.css'
```

之后就可以进行格式修改了

# 修改格式

以标题的修改文章盒子颜色为例

在刚才的_custom文件夹中新增一个css，命名随意，然后向其中添加:

```css
/* 首页文章卡片 */
#recent-posts > .recent-post-item{
    background:rgba(255, 255, 255, 0.9);
}
/* 首页侧栏卡片 */
.card-widget{
  background:rgba(255, 255, 255, 0.9)!important;
}
/* 文章页面正文背景 */
div#post{
  background: rgba(255, 255, 255, 0.9);
}
/* 分页页面 */
div#page{
  background: rgba(255, 255, 255, 0.9);
}
/* 归档页面 */
div#archive{
  background: rgba(255, 255, 255, 0.9);
}
/* 标签页面 */
div#tag{
  background: rgba(255, 255, 255, 0.9);
}
/* 分类页面 */
div#category{
  background: rgba(255, 255, 255, 0.9);
}
```

rgba(255, 255, 255, 0.9) 前三个表示对应的rgb值，最后一个是调节透明度。

此时也只是配置了默认状态下文章的颜色，但由于显示优先级顺序，自定义的样式会刷新主题原来的配置，当你切换黑夜模式时，会发现你的文章盒子仍然是透明的，没有随着主题变成黑色，像这样：

![未配置黑夜模式](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202410110139853.jpg)

因此还需要针对黑夜模式进行处理，还是在刚刚创建的那个css文件下添加以下代码（具体细节可以自己调整）

```css
/* 黑夜模式下*/
[data-theme='dark'] #recent-posts > .recent-post-item{
    background:rgba(0, 0, 0, 0.6)!important;
}
[data-theme='dark'] .card-widget{
  background:rgba(0, 0, 0, 0.9)!important;
}
[data-theme='dark'] div#post{
  background:rgba(0, 0, 0, 0.9)!important;
}
[data-theme='dark'] div#page{
  background:rgba(0, 0, 0, 0.9)!important;
}
[data-theme='dark'] div#archive{
  background:rgba(0, 0, 0, 0.9)!important;
}
[data-theme='dark'] div#tag{
  background:rgba(0, 0, 0, 0.9)!important;
}
[data-theme='dark'] div#category{
  background:rgba(0, 0, 0, 0.9)!important;
}
```

现在，切换黑夜模式时，**所有的盒子也会随之变成黑色了**！

![配置黑夜模式后](https://pic-bed1-9nl.pages.dev/imgs/post_Imags/202410110144967.jpg)

如果还有其他格式修改需求可以看文章参考。

——————————————————————————

参考：

[hexo-butterfly-样式修改butterfly主题升级，记录基于hexo-butterfly的修改过程，涉及自 - 掘金 (juejin.cn)](https://juejin.cn/post/7064584521210920974#heading-1)

[Butterfly主题美化进阶 - ThinkGone - 博客园 (cnblogs.com)](https://www.cnblogs.com/thinkgone/p/16348996.html)
