---
title: 16-Spring Security oAuth2
time: 2020年9月3日

---

[toc]

## 一、Spring Security oAuth2 简介

### 1.1 什么是 oAuth

oAuth 协议为用户资源的授权提供了一个安全的、开放而有简易的标准。与以往的授权方式不同之处是，oAuth 的授权不会使第三方触及到用户的账号信息（如用户名和密码），也就是说，第三方无需使用用户的用户名与密码就可以申请并获得该用户资源的授权，因此 oAuth 是安全的。

### 1.2 什么是 Spring Security

Spring Security 是一个安全框架，前身是 Acegi Security，能够为 Spring 企业应用系统提供声明式的安全访问控制。Spring Security 基于 Servlet 过滤器、IoC 和 AOP，为 Web 请求和方法调用提供身份确认和授权处理，避免了代码耦合，减少了大量重复代码工作。

> oAuth2 是一种协议，具体的实现有：① Spring Security； ② oauth2-shiro

## 二、oAuth2

### 2.1 为什么需要 oAuth2

我们假设你有一个“云笔记”产品，并提供了“云笔记服务”和“云相册服务”，此时用户需要在不同的设备（PC、Android、iPhone、TV、Watch）上去访问这些“资源”（笔记，图片）

那么用户如何才能访问属于自己的那部分资源呢？此时传统的做法就是提供自己的账号和密码给我们的“云笔记”，登录成功后就可以获取资源了。但这样的做法会有以下几个问题：

- “云笔记服务”和“云相册服务”会分别部署，难道我们要分别登录吗？
- 如果有第三方应用程序想要接入我们的“云笔记”，难道需要用户提供账号和密码给第三方应用程序，让他记录后再访问我们的资源吗？
- 用户如何限制第三方应用程序在我们“云笔记”的授权范围和使用期限？难道把所有资料都永久暴露给它吗？
- 如果用户修改了密码收回了权限，那么所有第三方应用程序会全部失效。
- 只要有一个接入的第三方应用程序遭到破解，那么用户的密码就会泄露，后果不堪设想。

为了解决如上问题，oAuth 应用而生。

### 2.2 名词解释

- **第三方应用程序（Third-party application）：** 又称之为客户端（client），比如上节中提到的设备（PC、Android、iPhone、TV、Watch），我们会在这些设备中安装我们自己研发的 APP。又比如我们的产品想要使用 QQ、微信等第三方登录。对我们的产品来说，QQ、微信登录是第三方登录系统。我们又需要第三方登录系统的资源（头像、昵称等）。对于 QQ、微信等系统我们又是第三方应用程序。
- **HTTP 服务提供商（HTTP service）：** 我们的云笔记产品以及 QQ、微信等都可以称之为“服务提供商”。
- **资源所有者（Resource Owner）：** 又称之为用户（user）。
- **用户代理（User Agent）：** 比如浏览器，代替用户去访问这些资源。
- **认证服务器（Authorization server）：** 即服务提供商专门用来处理认证的服务器，简单点说就是登录功能（验证用户的账号密码是否正确以及分配相应的权限）
- **资源服务器（Resource server）：** 即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。简单点说就是资源的访问入口，比如上节中提到的“云笔记服务”和“云相册服务”都可以称之为资源服务器。

![image-20200510124130587](16-Spring-Security-oAuth2.assets/image-20200510124130587.png)

> 简单来说，就是服务消费方想要访问『服务提供方』的资源，但是『提供方』需要对『消费方』进行验证，同时又不能把自己的账号密码提供给『消费方』，此时就诞生了 oAuth2 协议；这个协议通过在『提供方』实现一些动作，然后将认证和授权信息返回给『消费方』，从而达到上述的标准。

### 2.3 交互过程

oAuth 在 "客户端" 与 "服务提供商" 之间，设置了一个授权层（authorization layer）。"客户端" 不能直接登录 "服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端" 登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。"客户端" 登录授权层以后，"服务提供商" 根据令牌的权限范围和有效期，向 "客户端" 开放用户储存的资料。

![img](16-Spring-Security-oAuth2.assets/Lusifer_201904010001.png)

> 着重观察（客户端、认证服务器以及资源服务器）三者，就可以了解到：『认证服务器』的设置，利用了 AOP 的思想，『认证服务器』有两个作用：认证与授权。所以，在用户请求『资源服务器』之前，会先发出两个请求，这两个请求的作用就是“认证”与“授权”。
>
> 本质上，oAuth2 协议就是将原来“在资源服务器”上做的认证与授权，分离出来一个『认证服务器』专门来实现。然后『认证服务器』会返回一个“令牌”，从而让『资源服务器』确认身份。

## 三、Spring Security哦Auth2 令牌的访问与刷新

### 3.1 Access Token

Access Token 是客户端访问资源服务器的令牌，拥有**这个令牌代表着得到用户的授权**。然而，这个授权应该是**临时性**的，有一定效期。这是因为，Access Token 在使用的过程中可能会泄露，给 Access Token 限定一个较短的有效期可以降低因 Access Token 泄露而带来的风险。

然而引入了有效期以后，客户地段试用期起来就不那么方便了。每当 Access Token 过期，客户端就必须重新向用户索要授权，这样用户可能每隔几天，甚至每天都要进行授权操作。这是一件非常影响用户体验的事情。那么有没有什么办法可以避免这种情况呢？

于是，oAuth 2.0 引入了 Refresh Token 机制。

### 3.2 Refresh Token

Refresh Token 的作用是用来刷新 Access Token：认证服务器提供一个刷新接口，例如：

```
http://www.funtl.com/refresh?refresh_token=&client_id=
```



传入 `refresh_token `和 `client_id `，认证服务器验证通过以后，返回一个 Access Token。为了安全，oAuth 2.0 引入了两个措施：

1. oAuth 2.0 要求：Refresh Token 一定是保存在客户端的服务器上，而决不能放在狭义的客户端（例如 App、PC 端软件）上。调用 `refresh `接口的时候，一定是从服务器到服务器的访问。
2. oAuth 2.0 引入了 `client_secret `机制：即每一个 `client_id `都对应一个 `client_secret`。这个 `client_secret `会在客户端申请 `client_id  `时，随 `client_id  `一起分配给客户端。客户端必须把 `client_secret `妥善保管在服务器上，决不能泄露，刷新 Access Token 时，需要验证这个 `client_secret`。

实际上的刷新接口类似于：

```text
http://www.funtl.com/refresh?refresh_token=&client_id=&client_secret=
```

以上就是 Refresh Token 机制。Refresh Token 的有效期非常长，会在用户授权时，随 Access Token 一起重定向到回调 URL，传递给客户端。

## 四、Spring Security oAuth2 客户端授权模式

### 4.1 概述

客户端必须得到用户的授权，才能获得令牌。oAuth 2.0 定义了四种授权方式：

1. implicit：简化模式，不推荐使用
2. authorization code：授权码模式
3. resource owner password credentials：密码模式
4. client credentials：客户端模式；

> 多看看 08:35 视频

### 4.2 简化模式

简化模式适用于**纯静态页面应用**。所谓纯静态页面应用，也就是应用没有在服务器上执行代码的权限（通常是把代码托管在别人的服务器上），只有前端 JS 代码的控制权。

这种场景下，应用是**没有持久化存储的能力**的。因此，按照 oAuth2.0 的规定，这种应用是拿不到 Refresh Token 的。其整个授权流程如下：

![img](16-Spring-Security-oAuth2.assets/Lusifer_201904010002.png)

> 该模式下，`access_token `容易泄露，而且不可刷新

### 4.3 授权码模式

授权码模式适用于有自己的服务器的应用，它是一个一次性的临时凭证，用来换取 `access_token` 和 `refresh_token`。认证服务器提供了一个类似这样的接口：

```text
https://www.funtl.com/exchange?code=&client_id=&client_secret=
```

需要传入 `code`、`client_id` 以及 `client_secret`。验证通过后，返回 `access_token` 和 `refresh_token`。一旦换取成功，`code` 立即作废，不能再使用第二次。流程图如下：

![img](16-Spring-Security-oAuth2.assets/Lusifer_201904010003.png)

这个 code 的作用是保护 token 的安全性。上一节说到，简单模式下，token 是不安全的。这是因为在第 4 步当中直接把 token 返回给应用。而这一步容易被拦截、窃听。引入了 code 之后，即使攻击者能够窃取到 code，但是由于他无法获得应用保存在服务器的 `client_secret`，因此也无法通过 code 换取 token。而第 5 步，为什么不容易被拦截、窃听呢？这是因为，首先，这是一个从服务器到服务器的访问，黑客比较难捕捉到；其次，这个请求通常要求是 https 的实现。即使能窃听到数据包也无法解析出内容。

有了这个 code，token 的安全性大大提高。因此，oAuth2.0 鼓励使用这种方式进行授权，而简单模式则是在不得已情况下才会使用。

> 一般来说，『授权码模式』应用场景是在：① 两家公司的业务需要对接； ② 同一家公司的不同产品线需要对接的时候；

### 4.4 密码模式

密码模式中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向『服务提供商』索要授权。这种模式和传统的（用户名/密码）模式类似。在这种模式中，用户必须把自己的密码给客户端，但是客户端不得存储密码。这通常用在用户对客户端高度新人的情况下，比如：客户端是操作系统的一部分。

一个典型的例子就是同一个企业内部的不同产品要使用本企业的 oAuth2.0 体系。在有些情况下，产品希望能够定制化授权页面。由于是同个企业，不需要向用户展示“xxx将获取以下权限”等字样并询问用户的授权意向，而只需进行用户的身份认证即可。这个时候，由具体的产品团队开发定制化的授权界面，接收用户输入账号密码，并直接传递给鉴权服务器进行授权即可。

![img](16-Spring-Security-oAuth2.assets/Lusifer_2019040104250001.png)

有一点需要特别注意的是，在第 2 步中，认证服务器需要对客户端的身份进行验证，确保是受信任的客户端。

### 4.5 客户端模式

如果信任关系再进一步，或者调用者是一个后端的模块，没有用户界面的时候，可以使用客户端模式。鉴权服务器直接对客户端进行身份验证，验证通过后，返回 token。

> 也就是：连账号密码都不用输入了。

![img](16-Spring-Security-oAuth2.assets/Lusifer_2019040104270001.png)

## 五、Spring Security oAuth 2.0 工程

### 5.1 准备工作：创建项目工程以及依赖管理项目

创建工程项目 01-hello-spring-security-oauth2 并在 pom 文件中添加以下代码：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.chen</groupId>
    <artifactId>01-hello-spring-security-oauth2</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <modules>
        <module>01-spring-security-oauth2-dependencies</module>
        <module>02-oauth2</module>
        <module>01-oauth2-server</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <dependencyManagement>
        <dependencies>
            <!-- ① -->
            <dependency>
                <groupId>com.chen</groupId>
                <artifactId>01-spring-security-oauth2-dependencies</artifactId>
                <version>${project.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <profiles>
        <profile>
            <id>default</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <spring-javaformat.version>0.0.7</spring-javaformat.version>
            </properties>

            <build>
                <plugins>
                    <plugin>
                        <groupId>io.spring.javaformat</groupId>
                        <artifactId>spring-javaformat-maven-plugin</artifactId>
                        <version>${spring-javaformat.version}</version>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <configuration>
                            <includes>
                                <include>**/*Tests.java</include>
                            </includes>
                            <excludes>
                                <exclude>**/Abstract*.java</exclude>
                            </excludes>
                            <systemPropertyVariables>
                                <java.security.egd>file:/dev/./urandom</java.security.egd>
                                <java.awt.headless>true</java.awt.headless>
                            </systemPropertyVariables>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-enforcer-plugin</artifactId>
                        <executions>
                            <execution>
                                <id>enforce-rules</id>
                                <goals>
                                    <goal>enforce</goal>
                                </goals>
                                <configuration>
                                    <rules>
                                        <bannedDependencies>
                                            <excludes>
                                                <exclude>commons-logging:*:*</exclude>
                                            </excludes>
                                            <searchTransitive>true</searchTransitive>
                                        </bannedDependencies>
                                    </rules>
                                    <fail>true</fail>
                                </configuration>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-install-plugin</artifactId>
                        <configuration>
                            <skip>true</skip>
                        </configuration>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <configuration>
                            <skip>true</skip>
                        </configuration>
                        <inherited>true</inherited>
                    </plugin>
                </plugins>
            </build>
        </profile>
    </profiles>

    <repositories>
        <repository>
            <id>spring-milestone</id>
            <name>Spring Milestone</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-snapshot</id>
            <name>Spring Snapshot</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-milestone</id>
            <name>Spring Milestone</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>spring-snapshot</id>
            <name>Spring Snapshot</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>

</project>
```

① 这个依赖项目是全局统一管理项目

然后创建项目 01-spring-security-oauth2-dependencies ，并在 pom 文件中添加以下代码：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>01-hello-spring-security-oauth2</artifactId>
        <groupId>com.chen</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>

    <artifactId>01-spring-security-oauth2-dependencies</artifactId>

    <properties>
        <spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframeworl.cloud</groupId>
                <artifactId>spring-cloud-denpendencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>


    <repositories>
        <repository>
            <id>spring-milestone</id>
            <name>Spring Milestone</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-snapshot</id>
            <name>Spring Snapshot</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>

    <pluginRepositories>
        <pluginRepository>
            <id>spring-milestone</id>
            <name>Spring Milestone</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </pluginRepository>
        <pluginRepository>
            <id>spring-snapshot</id>
            <name>Spring Snapshot</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </pluginRepository>
    </pluginRepositories>

</project>
```

至此，我们创建了父工程项目以及依赖管理项目。

### 5.2 创建 oAuth 2.0 统一父工程

在 上面的父工程中创建项目 01-oauth2-server ，并在 pom 文件中添加以下代码：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>01-hello-spring-security-oauth2</artifactId>
        <groupId>com.chen</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>02-oauth2</artifactId>
    <packaging>pom</packaging>

    <modules>
        <module>01-oauth2-server</module>
    </modules>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

</project>
```

以后，我们的 oAuth2 的项目都在该项目下创建。

### 5.3 基于内存存储令牌

本次例子基于『内存存储令牌』的模式，用于实现『授权码模式』的简单例子。

#### 5.3.1 操作流程

![img](16-Spring-Security-oAuth2.assets/Lusifer_201904030001.png)

#### 5.3.2 配置认证服务器

创建一个类继承 `AuthorizationServerConfigurerAdapter `并在子类上添加注解

1. `@Configuration`
2. `@EnableAuthorizationServer `（表示开启认证服务）

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        // 配置客户端
        clients
                // 将 clientId 和 clientSecret 保存到内存中
                .inMemory()
                .withClient("client01")
                .secret(passwordEncoder.encode("secret01"))
                // 授权类型
                .authorizedGrantTypes("authorization_code")
                // 授权范围
                .scopes("app")
                // 注册回调地址
                .redirectUris("http://www.funtl.com");
    }
}

```

可以看到，上面重写的 `configure `方法中，在内存中设置了一个账号 `client01`，这个就是服务器为某个 APP 设置的认证账号，认证的类型是 `authorization_code `类型。当认证成功以后，认证服务器会重定向到 `http://www.funtl.com`。

#### 5.3.3 服务器安全配置

创建一个类继承 WebSecurityConfigurerAdapter ，并在子类上添加以下注解：

1. `@EnableWebSecurity`  
2. `@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)`

```java
@Configuration
@EnableWebSecurity
// ① 这里相当于设置了一个全局拦截器
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("user01").password(passwordEncoder().encode("123456")).roles("USER")
                .and()
                .withUser("admin").password(passwordEncoder().encode("123456")).roles("ADMIN");
    }
}
```

上面的 configure 方法中，设置了两个账号：user01 与 admin ；



经过上面的步骤，Spring Security oAuth 2.0 已经整合成功。

#### 5.3.4 启动并测试

启动项目，然后在浏览器中打开以下地址：

```
http://10.4.62.239:8090/oauth/authorize?client_id=client01&response_type=code
```

此时，此时会去到一个认证的页面：

![image-20200914183856603](16-Spring-Security-oAuth2.assets/image-20200914183856603.png)

使用账号 user01 或 admin 登陆以后，会跳转到授权页面：

![image-20200914183958717](16-Spring-Security-oAuth2.assets/image-20200914183958717.png)

这个页面就是在获得用户的授权。

得到用户的授权以后，系统会重定向到指定的网址，并且在网址上带有 code：

![image-20200914184208540](16-Spring-Security-oAuth2.assets/image-20200914184208540.png)

经过上面的操作，我们已经获得了 code，接下来的操作就是获得 token。

**获得 token**

我们打开 postman，通过 POST 方式请求以下 URL：

```
http://client01:secret01@10.4.62.239:8090/oauth/token
```

![image-20200914184358660](16-Spring-Security-oAuth2.assets/image-20200914184358660.png)

① 第一个表示的是认证服务器为用户分配的 clientId；第二个表示的是对应的密码；

点击请求以后，会得到以下的返回：

![image-20200914184527909](16-Spring-Security-oAuth2.assets/image-20200914184527909.png)

这时候，我们已经得到了 token，下一步，就是拿着这个 token 去请求资源服务器。



















