---
title:01-单体应用：my-shop笔记
time:2019年10月23日
---

[toc]

## 一、简介

记录项目my-shop的编码笔记。



## 二、

### 2.1 创建项目

首先创建一个maven项目，项目名称为my-shop。接着在项目下添加webapp目录（用于存放web资源文件）；在webapp目录下，我们创建一个assets目录用于存放静态资源文件；在webapp目录下，创建一个WEB-INF文件夹，里面再创建一个web.xml文件，文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">

</web-app>
```



此外，在java目录下，我们先按照三层架构创建好各个层级的包：bean、dao、service、web。



目录结构最后如下所示：
![1571835623873](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1571835623873.png)



在项目的pom文件中，添加以下依赖：

```xml
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>servlet-api</artifactId>
  <version>3.1.0</version>
</dependency>
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>jsp-api</artifactId>
  <version>2.0</version>
</dependency>
<!--  lombok插件  -->
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.16.10</version>
</dependency>
```



至此，项目雏形的搭建完毕。



### 2.2 添加静态资源文件

本次我们使用Bootstrap框架的模板AdminLTE，版本是2.4.3版本。

将该模板压缩包解压到项目之外的一个路径（假设解压后的模板项目名称为$template$），并把解压出来的三个文件夹复制到webapp\assets文件夹下：

![1571835985675](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1571835985675.png)



### 2.3 为项目添加登录页面

我们打开template目录的首页，并进入到login子模块中：

![1571836148122](assets/1571836148122.png)

![1571836163353](assets/1571836163353.png)

我们仿照这个登录页面的源码进行修改。假设





我们打开登录页面对应的本地文件的源码，并逐行复制。在粘贴到my-shop项目以前，看看待粘贴的代码的结构以及用途。

> tips：在刚开始使用别人的模板时，最好是一段一段地复制，而不要直接把所有代码复制到自己的项目中去。因为将大段代码分成多次复制，可以在复制的过程中观察源码的结构，熟悉源码的内容，这样在自己项目中的代码出现问题时，更加容易排查。



最后，我们得到的登录页面的源码如下：

```jsp
<!DOCTYPE html>
<html>
<!--head begin-->
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>登录页面</title>
    <!-- Tell the browser to be responsive to screen width -->
    <meta content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no" name="viewport">
    <!-- Bootstrap 3.3.7 -->
    <link rel="stylesheet" href="assets/bower_components/bootstrap/dist/css/bootstrap.min.css">
    <!-- Font Awesome -->
    <link rel="stylesheet" href="assets/bower_components/font-awesome/css/font-awesome.min.css">
    <!-- Ionicons -->
    <link rel="stylesheet" href="assets/bower_components/Ionicons/css/ionicons.min.css">
    <!-- Theme style -->
    <link rel="stylesheet" href="assets/dist/css/AdminLTE.min.css">
    <!-- iCheck -->
    <link rel="stylesheet" href="assets/plugins/iCheck/square/blue.css">

    <!-- HTML5 Shim and Respond.js IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
    <script src="https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
    <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->
</head>
<!--head end-->
<!--body begin-->
<body class="hold-transition login-page">
<!-- jQuery 3 -->
<script src="assets/bower_components/jquery/dist/jquery.min.js"></script>
<!-- Bootstrap 3.3.7 -->
<script src="assets/bower_components/bootstrap/dist/js/bootstrap.min.js"></script>
<!-- iCheck -->
<script src="assets/plugins/iCheck/icheck.min.js"></script>
<script>
    $(function () {
        $('input').iCheck({
            checkboxClass: 'icheckbox_square-blue',
            radioClass: 'iradio_square-blue',
            increaseArea: '20%' /* optional */
        });
    });
</script>

<div class="login-box">
    <div class="login-logo">
        <a href="#"><b>我的</b>商城</a>
    </div>

    <div class="login-box-body">
        <p class="login-box-msg">请输入用户名和密码
        <form action="../../index2.html" method="post">
            <div class="form-group has-feedback">
                <input type="email" class="form-control" placeholder="邮箱">
                <span class="glyphicon glyphicon-envelope form-control-feedback"></span>
            </div>
            <div class="form-group has-feedback">
                <input type="password" class="form-control" placeholder="密码">
                <span class="glyphicon glyphicon-lock form-control-feedback"></span>
            </div>
            <div class="row">
                <div class="col-xs-8">
                    <div class="checkbox icheck">
                        <label>
                            <input type="checkbox"> 记住我
                        </label>
                    </div>
                </div>
                <div class="col-xs-4">
                    <button type="submit" class="btn btn-primary btn-block btn-flat">登录</button>
                </div>
            </div>
        </form>

    </div>
</div>
</body>
<!--body end-->
</html>
```

渲染后的页面如下：

![1571836521963](assets/1571836521963.png)





### 2.4 新建实体类`User`以及数据访问层接口类`UserDao`

实体类`User`代码：

```java
import lombok.Data;
import lombok.experimental.Accessors;

/**
 * @Author: ChromeChen
 * @Description:
 * @Date: Created in 20:49 2019-10-23
 * @Modified By:
 */
@Data
@Accessors(chain = true)
public class User {

    private String username;
    private String password;
    private String email;
}
```

注意：`@Accessors(chain = true)`这个注解可以实现链式编程。



数据访问层接口类`UserDao`代码：

```java
import com.chen.my.shop.bean.User;

/**
 * @Author: ChromeChen
 * @Description: 访问后台用户数据库接口
 * @Date: Created in 20:48 2019-10-23
 * @Modified By:
 */
public interface UserDao {

    User getUserByEmailAndPassword(String email, String password);
}
```

> 关于方法命名规范：方法的命名既需要满足语义规范（看到方法名就能猜到方法的用途），也不能过长。所以，方法名的规范应该是两者的平衡。
>
> 此外，在数据访问层接口中，方法名以get开头，说明这个方法返回的是一个对象（而不是数组），



> 当你发现自己的类不需要被继承（或者不想被继承）的时候，可以使用 `final `关键字。



## 三、使用tomcat容器加载Spring配置文件，并得到上下文对象实例

### 3.1 tomcat容器启动时自动加载Spring配置文件

在pom文件中添加以下依赖：

```xml
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-web</artifactId>
  <version>5.1.8.RELEASE</version>
</dependency>
```

然后在web.xml中添加以下配置信息：

```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:spring.xml</param-value>
</context-param>
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

上面的信息表示：当tomcat容器启动时，会去类路径下加载*spring.xml* 文件



### 3.2 获取 bean 实例

首先编写SpringContext模板代码，在这个模板代码中，实现了：①获取Spring上下文对象； ②利用Spring上下文对象获取 bean 实例； 

```java
import org.apache.commons.lang3.Validate;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;

/**
 * @Author: ChromeChen
 * @Description:
 * @Date: Created in 21:32 2019-10-25
 * @Modified By:
 */
public final class SpringContext implements ApplicationContextAware, DisposableBean {

    private static final Logger logger = LoggerFactory.getLogger(SpringContext.class);

    public static ApplicationContext context = null;


    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringContext.context = applicationContext;
    }

    @Override
    public void destroy() throws Exception {
        context = null;
        logger.info("销毁了 ApplicationContext 对象");
    }

    /**
     * 根据 beanId 获取 bean 实例
     * @param beanId
     * @param <T>
     * @return
     */
    public static <T> T getBean(String beanId) {
        Object bean = SpringContext.context.getBean("name");
        return (T) bean;
    }

    /**
     * 根据 clazz 获取 bean 实例
     * @param clazz
     * @param <T>
     * @return
     */
    public static <T> T getBean(Class<T> clazz) {
        return SpringContext.context.getBean(clazz);
    }

    /**
     * 断言 Context 已经注入
     */
    private static void assertContextInjected() {
        Validate.validState(context != null, "applicationContext 属性未注入，请在 spring-context.xml 配置中定义 SpringContext");
    }

    /**
     * 获取存储在静态变量中的 ApplicationContext
     * @return
     */
    public static ApplicationContext getApplicationContext() {
        assertContextInjected();
        return context;
    }
}
```

注意：需要先添加依赖：

```xml
 <!--<editor-fold desc="commons-lang3依赖">-->
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.5</version>
</dependency>
<!--</editor-fold>-->
```



Spring的配置文件如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--SpringContext实例, 一定要配置在首行！！！！！！！！！！-->
    <bean id="springContext" class="com.chen.my.shop.commons.context.SpringContext"/>
    <!--数据接口层实现类-->
    <bean class="com.chen.my.shop.dao.impl.UserDaoImpl" id="userDao"/>
    <!--  业务层的实现类  -->
    <bean class="com.chen.my.shop.service.impl.UserServiceImpl" id="userService"/>
</beans>
```



此时，启动tomcat以后，已经可以将Spring文件加载并获得上下文对象了。



### 3.3 实现“记住我”功能

#### 技术选型

**Cookie**

Cookie 是指存储在用户本地终端上的数据，同时它是与具体的 Web 页面或者站点相关的。Cookie 数据会自动在 Web 浏览器和 Web 服务器之间传输，也就是说 HTTP 请求发送时，会把保存在该请求域名下的所有 Cookie 值发送给 Web 服务器，因此服务器端脚本是可以读、写存储在客户端的 Cookie 的操作。



**LocalStorage**

在 HTML5 中，新加入了一个 localStorage 特性，这个特性主要是用来作为本地存储来使用的，解决了 Cookie 存储空间不足的问题(Cookie 中每条 Cookie 的存储空间为 4k)，localStorage 中一般浏览器支持的是 5M 大小，这个在不同的浏览器中 localStorage 会有所不同。



**SessionStorage**

SessionStorage 与 LocalStorage 的唯一一点区别就是 LocalStorage 属于永久性存储，而 SessionStorage 属于当会话结束的时候，SessionStorage 中的键值对就会被清空。



**UserData、GlobalStorage、Google Gear**

这三种的使用都有一定的局限性，例如

- userData 是 IE 浏览器专属，它的容量可以达到 640K，这种方案可靠，不需要安装额外插件，只不过它仅在IE下有效
- globalStorage 适用于 Firefox 2+ 的浏览器，类似于 IE 的 userData
- google gear 是谷歌开发出的一种本地存储技术，需要安装 Gear 组件
- 

**Flash ShareObject（Flash Cookie）**

这种方式能能解决上面提到的 Cookie 存储的两个弊端，而且能够跨浏览器，应该说是目前最好的本地存储方案。不过，需要在页面中插入一个 Flash，当浏览器没有安装 Flash 控件时就不能用了。所幸的是，没有安装 Flash 的用户极少



本次使用的技术是Cookie。

**附：CookieUtil**

```java
import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.net.URLEncoder;

/**
 * Cookie 工具类
 * <p>Title: CookieUtils</p>
 * <p>Description: </p>
 *
 * @author Lusifer
 * @version 1.0.0
 * @date 2017/12/10 22:00
 */
public final class CookieUtils {

    /**
     * 得到Cookie的值, 不编码
     *
     * @param request
     * @param cookieName
     * @return
     */
    public static String getCookieValue(HttpServletRequest request, String cookieName) {
        return getCookieValue(request, cookieName, false);
    }

    /**
     * 得到Cookie的值,
     *
     * @param request
     * @param cookieName
     * @return
     */
    public static String getCookieValue(HttpServletRequest request, String cookieName, boolean isDecoder) {
        Cookie[] cookieList = request.getCookies();
        if (cookieList == null || cookieName == null) {
            return null;
        }
        String retValue = null;
        try {
            for (int i = 0; i < cookieList.length; i++) {
                if (cookieList[i].getName().equals(cookieName)) {
                    if (isDecoder) {
                        retValue = URLDecoder.decode(cookieList[i].getValue(), "UTF-8");
                    } else {
                        retValue = cookieList[i].getValue();
                    }
                    break;
                }
            }
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return retValue;
    }

    /**
     * 得到Cookie的值,
     *
     * @param request
     * @param cookieName
     * @return
     */
    public static String getCookieValue(HttpServletRequest request, String cookieName, String encodeString) {
        Cookie[] cookieList = request.getCookies();
        if (cookieList == null || cookieName == null) {
            return null;
        }
        String retValue = null;
        try {
            for (int i = 0; i < cookieList.length; i++) {
                if (cookieList[i].getName().equals(cookieName)) {
                    retValue = URLDecoder.decode(cookieList[i].getValue(), encodeString);
                    break;
                }
            }
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return retValue;
    }

    /**
     * 设置Cookie的值 不设置生效时间默认浏览器关闭即失效,也不编码
     */
    public static void setCookie(HttpServletRequest request, HttpServletResponse response, String cookieName,
                                 String cookieValue) {
        setCookie(request, response, cookieName, cookieValue, -1);
    }

    /**
     * 设置Cookie的值 在指定时间内生效,但不编码
     */
    public static void setCookie(HttpServletRequest request, HttpServletResponse response, String cookieName,
                                 String cookieValue, int cookieMaxage) {
        setCookie(request, response, cookieName, cookieValue, cookieMaxage, false);
    }

    /**
     * 设置Cookie的值 不设置生效时间,但编码
     */
    public static void setCookie(HttpServletRequest request, HttpServletResponse response, String cookieName,
                                 String cookieValue, boolean isEncode) {
        setCookie(request, response, cookieName, cookieValue, -1, isEncode);
    }

    /**
     * 设置Cookie的值 在指定时间内生效, 编码参数
     */
    public static void setCookie(HttpServletRequest request, HttpServletResponse response, String cookieName,
                                 String cookieValue, int cookieMaxage, boolean isEncode) {
        doSetCookie(request, response, cookieName, cookieValue, cookieMaxage, isEncode);
    }

    /**
     * 设置Cookie的值 在指定时间内生效, 编码参数(指定编码)
     */
    public static void setCookie(HttpServletRequest request, HttpServletResponse response, String cookieName,
                                 String cookieValue, int cookieMaxage, String encodeString) {
        doSetCookie(request, response, cookieName, cookieValue, cookieMaxage, encodeString);
    }

    /**
     * 删除Cookie带cookie域名
     */
    public static void deleteCookie(HttpServletRequest request, HttpServletResponse response,
                                    String cookieName) {
        doSetCookie(request, response, cookieName, "", -1, false);
    }

    /**
     * 设置Cookie的值，并使其在指定时间内生效
     *
     * @param cookieMaxage cookie生效的最大秒数
     */
    private static final void doSetCookie(HttpServletRequest request, HttpServletResponse response,
                                          String cookieName, String cookieValue, int cookieMaxage, boolean isEncode) {
        try {
            if (cookieValue == null) {
                cookieValue = "";
            } else if (isEncode) {
                cookieValue = URLEncoder.encode(cookieValue, "utf-8");
            }
            Cookie cookie = new Cookie(cookieName, cookieValue);
            if (cookieMaxage > 0)
                cookie.setMaxAge(cookieMaxage);
            if (null != request) {// 设置域名的cookie
                String domainName = getDomainName(request);
//                System.out.println(domainName);
                if (!"localhost".equals(domainName)) {
                    cookie.setDomain(domainName);
                }
            }
            cookie.setPath("/");
            response.addCookie(cookie);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 设置Cookie的值，并使其在指定时间内生效
     *
     * @param cookieMaxage cookie生效的最大秒数
     */
    private static final void doSetCookie(HttpServletRequest request, HttpServletResponse response,
                                          String cookieName, String cookieValue, int cookieMaxage, String encodeString) {
        try {
            if (cookieValue == null) {
                cookieValue = "";
            } else {
                cookieValue = URLEncoder.encode(cookieValue, encodeString);
            }
            Cookie cookie = new Cookie(cookieName, cookieValue);
            if (cookieMaxage > 0)
                cookie.setMaxAge(cookieMaxage);
            if (null != request) {// 设置域名的cookie
                String domainName = getDomainName(request);
//                System.out.println(domainName);
                if (!"localhost".equals(domainName)) {
                    cookie.setDomain(domainName);
                }
            }
            cookie.setPath("/");
            response.addCookie(cookie);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 得到cookie的域名
     */
    private static final String getDomainName(HttpServletRequest request) {
        String domainName = null;

        String serverName = request.getRequestURL().toString();
        if (serverName == null || serverName.equals("")) {
            domainName = "";
        } else {
            serverName = serverName.toLowerCase();
            serverName = serverName.substring(7);
            final int end = serverName.indexOf("/");
            serverName = serverName.substring(0, end);
            final String[] domains = serverName.split("\\.");
            int len = domains.length;
            if (len > 3) {
                // www.xxx.com.cn
                domainName = "." + domains[len - 3] + "." + domains[len - 2] + "." + domains[len - 1];
            } else if (len <= 3 && len > 1) {
                // xxx.com or xxx.cn
                domainName = "." + domains[len - 2] + "." + domains[len - 1];
            } else {
                domainName = serverName;
            }
        }

        if (domainName != null && domainName.indexOf(":") > 0) {
            String[] ary = domainName.split("\\:");
            domainName = ary[0];
        }
        return domainName;
    }

}
```



#### 使用Cookie实现“记住我”功能

暂时不实现。















