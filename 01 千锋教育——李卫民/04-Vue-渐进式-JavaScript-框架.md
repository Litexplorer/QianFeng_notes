---
title: 04-Vue 渐进式 JavaScript 框架
time: 2019年12月11日
---

[toc]



## 〇、视频介绍



## 一、概述



## 二、前端知识体系



#### 表现层（CSS）

CSS 层叠样式表是一门标记语言，并不是编程语言，因此不可以自定义变量，不可以引用等，换句话说就是不具备任何语法支持，它主要缺陷如下：

- 语法不够强大，比如无法嵌套书写，导致模块化开发中需要书写很多重复的选择器；
- 没有变量和合理的样式复用机制，使得逻辑上相关的属性值必须以字面量的形式重复输出，导致难以维护；

这就导致了我们在工作中无端增加了许多工作量。为了解决这个问题，前端开发人员会使用一种称之为 **【CSS 预处理器】** 的工具，提供 CSS 缺失的样式层复用机制、减少冗余代码，提高样式代码的可维护性。大大提高了前端在样式上的开发效率。

##### [#](https://funtl.com/zh/vue-prepare/#什么是-css-预处理器)什么是 CSS 预处理器

CSS 预处理器定义了一种新的语言，其基本思想是，用一种专门的编程语言，为 CSS 增加了一些编程的特性，将 CSS 作为目标生成文件，然后开发者就只要使用这种语言进行 CSS 的编码工作。转化成通俗易懂的话来说就是“**用一种专门的编程语言，进行 Web 页面样式设计，再通过编译器转化为正常的 CSS 文件，以供项目使用**”。

##### [#](https://funtl.com/zh/vue-prepare/#常用的-css-预处理器有哪些)常用的 CSS 预处理器有哪些

- SASS：基于 Ruby，通过服务端处理，功能强大。解析效率高。需要学习 Ruby 语言，上手难度高于 LESS。
- LESS：基于 NodeJS，通过客户端处理，使用简单。功能比 SASS 简单，解析效率也低于 SASS，但在实际开发中足够了，所以我们后台人员如果需要的话，建议使用 LESS。



### 2.2 了解 MVVM 模式

#### 2.2.1 为什么使用 MVVM 模式？

**需求：**

现在我们来考虑一个需求：如果我们现在有一个 input 框，同时有 n 个 div，现在需要实现：当用户在 input 框中输入任意内容的时候，这 n 个div 可以填入刚刚用户输入的那内容。

**传统做法：**

如果使用传统的 DOM 操作，那么就需要编写 n 个 DOM 操作的 js 代码。当 n 很大的时候，浏览器的渲染性能就会出现很大的瓶颈。

**新做法：**

如果使用 MVVM 模式的话，ViewModel 就相当于一个**观察者**，这个观察者可以实时监控 input 框中的内容变化，并实时将内容填写到 n 个 div 中。



### 2.3 Vue.js：MVVM 模式的实现者

我们知道（不知道的请移步 【[了解前端 MVVM 模式](https://funtl.com/zh/vue-prepare/了解前端-MVVM-模式.html#了解前端-mvvm-模式)】） MVVM 表示如下：

- Model：模型层，在这里表示 JavaScript 对象
- View：视图层，在这里表示 DOM（HTML 操作的元素）
- ViewModel：连接视图和数据的中间件，**Vue.js 就是 MVVM 中的 ViewModel 层的实现者**

![img](assets/Lusifer201812170001.png)

在 MVVM 架构中，是不允许 **数据** 和 **视图** 直接通信的，只能通过 `ViewModel` 来通信，而 ViewModel 就是定义了一个 `Observer` 观察者

- ViewModel 能够观察到数据的变化，并对视图对应的内容进行更新
- ViewModel 能够监听到视图的变化，并能够通知数据发生改变

至此，我们就明白了，Vue.js 就是一个 MVVM 的实现者，他的核心就是实现了 `DOM 监听` 与 `数据绑定`



Vue 两大核心要素：

1. 数据驱动：实现 MVVM 模式；
2. 组件化开发：每个页面都是通过组件拼接完成；



总结：Vue.js 框架就是一个 MVVM 模式的实现者，MVVM 模式的核心是 ViewModel 层，ViewModel 就是一个观察者；







### 2.4 第一个 Vue.js 应用程序

**1. 我们首先在编辑器中新建一个 HTML，并添加一个 div ：**（TIPS：如果使用 VSCode 开发，那么可以在新建文件以后，把文件的语言模式设置为“HTML”，接着，先按<kbd>!</kbd>，再按<kbd>Tab</kbd>即可出现 HTML5 模板）

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>lesson-01</title>

</head>
<body>
    <div id="app">
    </div>
</body>
</html>
```



**2. 接着，引入 Vue.js **

在 html→head中加入以下代码：

```html
<!-- 引入 Vue.js -->
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
```



> 上面的 script 中的值指的是 Vue.js 的 CDN 地址，通常来说，CDN 地址包含了两个，分别是：
>
> - `<script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>`
>
> - `  <script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.min.js">  </script>`
>
>   如果在项目中引入的是第二个（也就是说带有 “min” 的 CDN 地址，那么我们就不能在页面上随意修改变量的值了（也可以说成：“min” 版本的不支持调试模式））



**3. 编写 Vue 的实例**

```html
 <script type="text/javascript">
     var vm = new Vue({
         el : '#app',
         data : {
             message : '你好，Vue！'
         }
     })
</script>
```

上面的代码中，`new Vue`会返回一个 Vue 对象；该对象有两个属性：`el`，对应了哪个页面元素（如果该页面元素使用了 `id` 属性，那么具体的语法为：`el : ‘#id’`）；以及`data`，里面定义了具体的数据，数据一般是以 json 形式呈现的。



> 注意：定义了 Vue 对象以后，我们想要获取 `message `的值，可以在浏览器控制台中输入` vm.message`，并按回车即可。可以发现：节点 `data `只是一个语法上的标签，在获取具体的数据时，是需要忽略的。



**4. 在视图中获取数据**

把 div 中的代码修改如下：

```xml
<div id="app">
    {{message}}
</div>
```



最后，我们在浏览器中运行页面，即可看到以下输出：

![image-20191222132251008](04-Vue-渐进式-JavaScript-框架.assets/image-20191222132251008.png)

我们在控制台中修改` vm.message` 的值，页面上会马上显示修改后的值：

![image-20191222132405622](04-Vue-渐进式-JavaScript-框架.assets/image-20191222132405622.png)



### 2.5 Vue 语法篇

我们下面来介绍几个常用的 Vue 语法：

- v-if(v-else-if、v-else)
- v-for



#### 2.5.1 条件判断

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>lesson-02</title>

    <script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
</head>
<body>
    
    <div id="app">
        <h1 v-if="ok">Yes</h1> ②
        <h1 v-else>No</h1>
    </div>

    <script type="text/javascript">
        var vm = new Vue({
            el : '#app', 
            data : {
                ok : true ①
            }
        })
    </script>
</body>
</html>
```

①：在数据中直接定义键值对；

②：v-if 标签定义在 HTML 标签中，v-if 的值直接使用 ① 处的键；



另外，①中可以定义包含中文的 键，在②处也可以直接引用。



#### 2.5.2 循环遍历

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>lesson-03</title>
     <!-- 引入 Vue.js -->
     <script src="https://cdn.jsdelivr.net/npm/vue@2.5.21/dist/vue.js"></script>
</head>
<body>

    <div id="app">
        <li v-for="item in list">{{item.id}}</li> ①
        <li v-for="item in list">{{item.name}}</li>
    </div>
    
    <script>
        var vm = new Vue({
            el : '#app', 
            data : {
                list : [{
                    id : 'id01', 
                    name : 'name01'
                }, {
                    id : 'id02', 
                    name : 'name02'
                }]
            }
        })
    </script>
</body>
</html>
```

① ：这是 Vue 的循环语句的语法，和 Thymeleaf 非常像。另外，我们还可以发现：当需要访问的数据在 Vue 的标签中时，不需要使用 `{{}}` 形式，而不在 Vue 的标签中时，则需要使用 `{{}}`来获取。

























