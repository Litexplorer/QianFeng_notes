---
title:01-单体应用：04章-BootStrap教程
time:2019年10月14日
---

[toc]

## 〇、视频介绍

视频来源于千锋网，在线观看地址为：https://www.bilibili.com/video/av29299488/?p=1

笔记地址为博主的博客：https://www.funtl.com/。



## 一、BootStrap简介及安装



### 1.2 BootStrap安装

首先下载BootStrap，地址为：https://getbootstrap.com/。由于浏览器的兼容性问题，尽量选取3.X的版本学习（这是因为BootStrap3.X版本支持的最低IE浏览器版本为IE8）。



下载完成以后，我们得到一个压缩包。解压这个压缩包，并放到项目的目录下即可。

> 注意：BootStrap框架中用到了很多jquery的语法，因此需要依赖jquery。



### 1.3 BootStrap入门案例

#### 1.3.1 项目目录结构

本次我们使用IntelliJ IDEA开发前端界面。

在正式开始项目以前，我们需要规范一下项目的包结构：

1. 静态资源文件统一放在*assets*文件夹下；
1. 用到的框架文件需要放在*plugins*文件夹下，这个文件夹再继续分具体哪一个框架；
1. 自定义资源文件放到*images*、*css*、*js*文件夹中；
1. *index.html*文件放到*assets*文件夹下；

最后的目录结构如下图所示：

![1571152522721](assets/1571152522721.png)

> 为什么要将index放到根目录下？
>
> - 这是由于使用IDEA直接启动HTML文件的时候，是以根目录（在本例中是BP_HelloWorld目录）为起点的，此时，如果在HTML文件中使用相对路径引入资源文件的话，基路径是BP_HelloWorld目录。为了避免歧义，最好把HTML文件放到根目录下。

#### 1.3.2 编写首页文件

接下来编写index页面：

##### 1.3.2.1 在头部引入css样式

在<head>中加入以下代码：

```html
<link rel="stylesheet" type="text/css" href="assets/plugins/bootstrap/css/bootstrap.min.css"/>
    <link rel="stylesheet" type="text/css" href="assets/plugins/bootstrap/css/bootstrap.css"/>

```



##### 1.3.2.2 引入js文件

在<body>中加入以下代码：

```html
<script src="assets/plugins/jquery-3.3.1.min.js"></script>
<script src="assets/plugins/bootstrap/js/bootstrap.min.js" ></script>
<script src="assets/plugins/bootstrap/css/bootstrap-theme.css" ></script>
```

上面第一行是引入BootStrap所依赖的jquery框架，==为了保险起见，最好放在第一行==。



##### 1.3.2.3 修改<meta>

把<meta>标签修改为以下形式：

```html
<meta charset="UTF-8"  name="viewport" content="width=device-width, initial-scale=1.0">
```

上面的`content`含义是自适应屏幕大小。



##### 1.3.2.4 加入IE8和IE9支持

在<head>中加入以下代码：

```html
<!-- HTML5 Shiv 和 Respond.js 用于让 IE8 支持 HTML5元素和媒体查询 -->
    <!-- 注意： 如果通过 file://  引入 Respond.js 文件，则该文件无法起效果 -->
    <!--[if lt IE 9]>
    <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.0/html5shiv.js"></script>
    <script src="https://oss.maxcdn.com/libs/respond.js/1.3.0/respond.min.js"></script>
    <![endif]-->
```

如果不需要兼容IE8和IE9，那么上面的代码可以不添加。

最后，加上一个标题用于页面的显示即可：

```html
<h1>Hello, world!</h1>
```

> 完整的HTML页面代码如下所示：
>
> ```html
> <!DOCTYPE html>
> <html lang="en">
> <head>
>     <meta charset="UTF-8"  name="viewport" content="width=device-width, initial-scale=1.0">
>     <title>首页</title>
> 
>     <link rel="stylesheet" type="text/css" href="assets/plugins/bootstrap/css/bootstrap.min.css"/>
>     <link rel="stylesheet" type="text/css" href="assets/plugins/bootstrap/css/bootstrap.css"/>
> 
>     <!-- HTML5 Shiv 和 Respond.js 用于让 IE8 支持 HTML5元素和媒体查询 -->
>     <!-- 注意： 如果通过 file://  引入 Respond.js 文件，则该文件无法起效果 -->
>     <!--[if lt IE 9]>
>     <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.0/html5shiv.js"></script>
>     <script src="https://oss.maxcdn.com/libs/respond.js/1.3.0/respond.min.js"></script>
>     <![endif]-->
> </head>
> <body>
> 
> <h1>Hello, world!</h1>
> 
> <script src="assets/plugins/jquery-3.3.1.min.js"></script>
> <script src="assets/plugins/bootstrap/js/bootstrap.min.js" ></script>
> <script src="assets/plugins/bootstrap/css/bootstrap-theme.css" ></script>
> 
> </body>
> </html>
> ```
>
> 



此时在IDEA中运行项目即可看到效果：

![1571153152852](assets/1571153152852.png)

为了验证Bootstrap的css和js是否生效，我们可以打开控制台观察结果：如果引入的语句没有报404错误，那么说明已经正确引入了。



