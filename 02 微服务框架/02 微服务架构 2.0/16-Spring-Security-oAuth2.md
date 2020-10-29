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

## 三、Spring Security OAuth2 令牌的访问与刷新

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
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.chen</groupId>
    <artifactId>01-spring-security-oauth2</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <url>http://www.funtl.com</url>

    <modules>
        <module>oauth2</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>



    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.chen</groupId>
                <artifactId>dependencies</artifactId>
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
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.chen</groupId>
    <artifactId>dependencies</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <url>http://www.funtl.com</url>

    <properties>
        <spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
    </properties>


    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
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
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.chen</groupId>
        <artifactId>01-spring-security-oauth2</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>oauth2</artifactId>
    <packaging>pom</packaging>

    <modules>
        <!-- 工程模块请随着项目的不断完善自行添加 -->
    </modules>

</project>

```

以后，我们的 oAuth2 的项目都在该项目下创建。

### 5.3 基于内存存储令牌

本次例子基于『内存存储令牌』的模式，用于实现『授权码模式』的简单例子。

我们首先在 01-oauth2-server 文件夹下创建一个新的文件夹 oauth2-server，在该文件夹下创建 pom 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.chen</groupId>
        <artifactId>oauth2</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>oauth2-server</artifactId>


    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <!-- Spring Security -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.oauth2.OAuth2ServerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

注意刷新 idea 项目的所能够识别的依赖：

![image-20201007123455596](16-Spring-Security-oAuth2.assets/image-20201007123455596.png)

如果没有识别到依赖，那可能需要重启 IDE。

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

创建一个类继承 `WebSecurityConfigurerAdapter `，并在子类上添加以下注解：

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

### 5.4 基于内存存储令牌的——服务端 + 客户端

在这个案例中，我们会在上面的『基于内存存储令牌』的基础上，将“手动获取令牌”，修改为“通过一个 SpringBoot 服务获取令牌”。

（未完成）

### 5.5 基于 JDBC 存储令牌

#### 5.5.1 流程图

![img](16-Spring-Security-oAuth2.assets/Lusifer_201904030001.png)

#### 5.5.2 准备工作

在进行操作以前，我们需要准备好以下的工作：

1. 系统中存在 oAuth2 相关的表；具体哪些表可以参考：https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql
   除了需要这些表以外，我们还需要在表中配置相关的信息
2. 确保项目可以正确地读取数据库（也就是配置数据源、实体类等信息）



#### 5.5.3 准备工作：创建 oAuth2 相关的表

在数据库中运行以下 SQL 语句：

```mysql
CREATE TABLE `clientdetails` (
  `appId` varchar(128) NOT NULL,
  `resourceIds` varchar(256) DEFAULT NULL,
  `appSecret` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `grantTypes` varchar(256) DEFAULT NULL,
  `redirectUrl` varchar(256) DEFAULT NULL,
  `authorities` varchar(256) DEFAULT NULL,
  `access_token_validity` int(11) DEFAULT NULL,
  `refresh_token_validity` int(11) DEFAULT NULL,
  `additionalInformation` varchar(4096) DEFAULT NULL,
  `autoApproveScopes` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`appId`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_access_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication_id` varchar(128) NOT NULL,
  `user_name` varchar(256) DEFAULT NULL,
  `client_id` varchar(256) DEFAULT NULL,
  `authentication` blob,
  `refresh_token` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`authentication_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_approvals` (
  `userId` varchar(256) DEFAULT NULL,
  `clientId` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `status` varchar(10) DEFAULT NULL,
  `expiresAt` timestamp NULL DEFAULT NULL,
  `lastModifiedAt` timestamp NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_client_details` (
  `client_id` varchar(128) NOT NULL,
  `resource_ids` varchar(256) DEFAULT NULL,
  `client_secret` varchar(256) DEFAULT NULL,
  `scope` varchar(256) DEFAULT NULL,
  `authorized_grant_types` varchar(256) DEFAULT NULL,
  `web_server_redirect_uri` varchar(256) DEFAULT NULL,
  `authorities` varchar(256) DEFAULT NULL,
  `access_token_validity` int(11) DEFAULT NULL,
  `refresh_token_validity` int(11) DEFAULT NULL,
  `additional_information` varchar(4096) DEFAULT NULL,
  `autoapprove` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`client_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_client_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication_id` varchar(128) NOT NULL,
  `user_name` varchar(256) DEFAULT NULL,
  `client_id` varchar(256) DEFAULT NULL,
  PRIMARY KEY (`authentication_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_code` (
  `code` varchar(256) DEFAULT NULL,
  `authentication` blob
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `oauth_refresh_token` (
  `token_id` varchar(256) DEFAULT NULL,
  `token` blob,
  `authentication` blob
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

```

然后执行新的脚本：

```mysql
INSERT INTO `oauth_client_details`(`client_id`, `resource_ids`, `client_secret`, `scope`, `authorized_grant_types`, `web_server_redirect_uri`, `authorities`, `access_token_validity`, `refresh_token_validity`, `additional_information`, `autoapprove`) VALUES ('client01', NULL, '$2a$10$oE9w0jg8BShqATIwhG2cyOPI7qFbA4sVTfNBYm.RPtHTojYJVCZy6', 'app', 'authorization_code', 'http://www.baidu.com', NULL, NULL, NULL, NULL, NULL);

```

上面脚本的意思是：在 oauth_client_details 表中插入一条记录，该记录的 client_secret 明文值为 123456.回调地址为 http://www.baidu.com。

上面的脚本可以相当替换了 AuthorizationServerConfiguration 中的内存配置。

接下来，我们就可以配置 Spring Security oAuth2 了。

#### 5.5.4 引入依赖

我们本次例子中需要连接数据库，因此需要引入数据库连接的相关依赖：

1. 在 dependencies 项目中引入：

   ```xml
   1. 在 properties 中引入
   <hikaricp.version>3.2.0</hikaricp.version>
   <mysql.version>8.0.21</mysql.version>
   
   2. 在 dependencyManagement 中引入
   <dependency>
       <groupId>com.zaxxer</groupId>
       <artifactId>HikariCP</artifactId>
       <version>${hikaricp.version}</version>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-jdbc</artifactId>
       <exclusions>
           <!-- 排除 tomcat-jdbc 以使用 HikariCP -->
           <exclusion>
               <groupId>org.apache.tomcat</groupId>
               <artifactId>tomcat-jdbc</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
       <version>${mysql.version}</version>
   </dependency>
   ```

2. 在 oauth-server 项目中引入：

   ```xml
   <dependency>
       <groupId>com.zaxxer</groupId>
       <artifactId>HikariCP</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-jdbc</artifactId>
       <exclusions>
           <!-- 排除 tomcat-jdbc 以使用 HikariCP -->
           <exclusion>
               <groupId>org.apache.tomcat</groupId>
               <artifactId>tomcat-jdbc</artifactId>
           </exclusion>
       </exclusions>
   </dependency>
   <dependency>
       <groupId>mysql</groupId>
       <artifactId>mysql-connector-java</artifactId>
   </dependency>
   ```

   注意刷新项目依赖：
   ![image-20201007124107187](16-Spring-Security-oAuth2.assets/image-20201007124107187.png)

#### 5.5.5 配置认证服务器

创建一个类，继承 AuthorizationServerConfigurerAdapter，然后在该类中配置数据源等项相关信息：

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.client.JdbcClientDetailsService;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JdbcTokenStore;

import javax.sql.DataSource;

/**
 * @Author: ChromeChen
 * @Description:
 * @Date: Created in 12:41 2020/10/7 0007
 * @Modified By:
 */
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Bean
    @Primary
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public TokenStore tokenStore() {
        // 基于 JDBC 实现，令牌保存到数据库
        return new JdbcTokenStore(dataSource());
    }

    @Bean
    public ClientDetailsService clientDetailsService() {
        // 基于 JDBC 实现，需要事先在数据库配置客户端信息
        return new JdbcClientDetailsService(dataSource());
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        // 设置令牌
        endpoints.tokenStore(tokenStore());
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        // 读取客户端配置
        clients.withClientDetails(clientDetailsService());
    }
}

```

#### 5.5.6 服务器安全配置

创建一个类，继承 WebSecurityConfigurerAdapter，在子类中配置相关信息：

```java

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        // 设置默认的加密方式
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {

        auth.inMemoryAuthentication()
                // 在内存中创建用户并为密码加密
                .withUser("user01").password(passwordEncoder().encode("123456")).roles("USER")
                .and()
                .withUser("admin01").password(passwordEncoder().encode("123456")).roles("ADMIN");

    }
}
```

通过上面的配置我们可以看出：

1. 我们的用户名和密码都是基于『内存』方式来存储的，没有涉及到数据库；
2. `@EnableGlobalMethodSecurity `注解开启了全局拦截器；

#### 5.5.7 在配置文件中补充数据库相关信息：

```yml
spring:
  application:
    name: oauth-server
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    jdbc-url: jdbc:mysql://localhost:3307/oauth2?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root
    hikari:
      minimum-idle: 5
      idle-timeout: 600000
      maximum-pool-size: 10
      auto-commit: true
      pool-name: MyHikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
server:
  port: 8090

```

#### 5.5.8 启动并排查数据库报错

启动项目，然后观察启动日志。

1. 如果发现出现以下错误：

   ```
   java.sql.SQLException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
   
   ```

   观察日志，我们可以发现是时区的问题，我们需要在 jdbc-url的值后面添加参数：`serverTimezone=GMT`

当启动成功以后，我们可以发现数据库连接池也初始化成功：

![image-20201007130107153](16-Spring-Security-oAuth2.assets/image-20201007130107153.png)

#### 5.5.9 访问并获取 token

访问地址：

```
http://localhost:8090/oauth/authorize?client_id=client01&response_type=code
```

并登陆，并使用代码中的用户名密码登录：

![image-20201007130353197](16-Spring-Security-oAuth2.assets/image-20201007130353197.png)

点击『Sign in』以后会跳转到这个页面：

![image-20201007130421962](16-Spring-Security-oAuth2.assets/image-20201007130421962.png)

我们点击『Approve』并点击按钮『Authorize』，将会跳转到指定的 URL 中（该 URL 在数据库中指定），该 URL 后面会附带一个 code：

![image-20201007130600144](16-Spring-Security-oAuth2.assets/image-20201007130600144.png)

接着，我们通过 curl 或者 Postman 来请求 token：

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'grant_type=authorization_code&code=Mx0djV' "http://client01:secret01@localhost:8090/oauth/token"
```

即可得到 token。

![image-20201007131529225](16-Spring-Security-oAuth2.assets/image-20201007131529225.png)

数据库中 oauth_access_token 也出现了相应的记录

![image-20201007131649887](16-Spring-Security-oAuth2.assets/image-20201007131649887.png)

#### 5.5.10 总结

『基于 JDBC 存储令牌』相比『基于内存存储令牌』，就是在数据库中存储了令牌信息。存储的位置是 oauth2 的指定表中。增加了对应的表以后，我们需要在项目代码中增加指定的依赖，用于连接数据库，并把 `AuthorizationServerConfiguration `授权方式从内存改为数据库中读取。注意连接数据库的数据源需要被 OAuth 2 接管。

## 六、RBAC——基于角色的访问控制

### 6.1 概述

在上面的章节中，我们实现了『用户的认证』功能，但是『授权』功能还没有完成。

在实际的开发中，我们往往都是使用『RBAC』模型进行访问控制。这个模型在 Windows 系统以及我们即将要学习的 Kubernetes 中都有所体现。

> 标准的 RBAC 模型实际上只需要三张表：user，role 以及 permission ，另外两张关联表是用于关联表作用。
>
> 在标准的 RBAC 模型下，我们还能够继续扩展：
>
> 1. 由于每个用户都需要和角色关联，我们可以创建一个『用户组』的表，由『用户组』直接和『角色』关联，而『用户』则关联到『用户组』下；
> 2. 如果需要动态展示不同的菜单，我们可以多添加一张表『菜单』，『菜单』和『角色』相关联，这样就可以动态展示菜单了；
>
> 通过上面的两个扩展可以看出：无论是“标准的 RBAC 模型”，还是在基础上扩展的“用户组”，或是在基础上扩展“菜单”，几乎所有表都和『角色』表相关联。这就是这个模型的名称由来：RBAC—— Role Base Access Control。 

### 6.2 操作流程

本格章节直接使用 “标准 RBAC 模型”实现授权操作。

具体步骤如下：

1. 初始化 RBAC 相关表；
2. 在数据库中配置好“用户”、“角色”、“权限”相关配置信息；
3. 配置 Web 安全
   - 配置使用自定义认证与授权；
4. 通过 GET 请求访问认证服务器，获取授权码；
   - `/oauth/authorize`
5. 通过 POST 请求利用授权码访问认证服务器，获取令牌：
   - `/oauth/token`

> 默认端点 URL
>
> - `/oauth/authorize`：授权端点
> - `/oauth/token`：令牌端点
> - `/oauth/confirm_access`：用户确认授权提交端点
> - `/oauth/error`：授权服务错误信息端点
> - `/oauth/check_token`：用于资源服务访问的令牌解析端点
> - `/oauth/token_key`：提供公有密匙的端点，如果你使用 JWT 令牌的话



> MD5 不是加密，而是一种校验机制；

### 6.3 操作步骤

#### 6.3.1 初始化数据库

在数据库中执行以下脚本，用于初始化 RBAC 相关的表：

```mysql
CREATE TABLE `tb_permission` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父权限',
  `name` varchar(64) NOT NULL COMMENT '权限名称',
  `enname` varchar(64) NOT NULL COMMENT '权限英文名称',
  `url` varchar(255) NOT NULL COMMENT '授权路径',
  `description` varchar(200) DEFAULT NULL COMMENT '备注',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=44 DEFAULT CHARSET=utf8 COMMENT='权限表';
insert  into `tb_permission`(`id`,`parent_id`,`name`,`enname`,`url`,`description`,`created`,`updated`) values 
(37,0,'系统管理','System','/',NULL,'2019-04-04 23:22:54','2019-04-04 23:22:56'),
(38,37,'用户管理','SystemUser','/users/',NULL,'2019-04-04 23:25:31','2019-04-04 23:25:33'),
(39,38,'查看用户','SystemUserView','',NULL,'2019-04-04 15:30:30','2019-04-04 15:30:43'),
(40,38,'新增用户','SystemUserInsert','',NULL,'2019-04-04 15:30:31','2019-04-04 15:30:44'),
(41,38,'编辑用户','SystemUserUpdate','',NULL,'2019-04-04 15:30:32','2019-04-04 15:30:45'),
(42,38,'删除用户','SystemUserDelete','',NULL,'2019-04-04 15:30:48','2019-04-04 15:30:45');

CREATE TABLE `tb_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父角色',
  `name` varchar(64) NOT NULL COMMENT '角色名称',
  `enname` varchar(64) NOT NULL COMMENT '角色英文名称',
  `description` varchar(200) DEFAULT NULL COMMENT '备注',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='角色表';
insert  into `tb_role`(`id`,`parent_id`,`name`,`enname`,`description`,`created`,`updated`) values 
(37,0,'超级管理员','admin',NULL,'2019-04-04 23:22:03','2019-04-04 23:22:05');

CREATE TABLE `tb_role_permission` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_id` bigint(20) NOT NULL COMMENT '角色 ID',
  `permission_id` bigint(20) NOT NULL COMMENT '权限 ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=43 DEFAULT CHARSET=utf8 COMMENT='角色权限表';
insert  into `tb_role_permission`(`id`,`role_id`,`permission_id`) values 
(37,37,37),
(38,37,38),
(39,37,39),
(40,37,40),
(41,37,41),
(42,37,42);

CREATE TABLE `tb_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) NOT NULL COMMENT '用户名',
  `password` varchar(64) NOT NULL COMMENT '密码，加密存储',
  `phone` varchar(20) DEFAULT NULL COMMENT '注册手机号',
  `email` varchar(50) DEFAULT NULL COMMENT '注册邮箱',
  `created` datetime NOT NULL,
  `updated` datetime NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`) USING BTREE,
  UNIQUE KEY `phone` (`phone`) USING BTREE,
  UNIQUE KEY `email` (`email`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='用户表';
insert  into `tb_user`(`id`,`username`,`password`,`phone`,`email`,`created`,`updated`) values 
(37,'admin','$2a$10$9ZhDOBp.sRKat4l14ygu/.LscxrMUcDAfeVOEPiYwbcRkoB09gCmi','15888888888','lee.lusifer@gmail.com','2019-04-04 23:21:27','2019-04-04 23:21:29');

CREATE TABLE `tb_user_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NOT NULL COMMENT '用户 ID',
  `role_id` bigint(20) NOT NULL COMMENT '角色 ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=38 DEFAULT CHARSET=utf8 COMMENT='用户角色表';
insert  into `tb_user_role`(`id`,`user_id`,`role_id`) values 
(37,37,37);

```

数据库的密码的明文为：123456.

#### 6.3.2 在项目代码中添加 tk.mybatis 的依赖

我们需要在 dependencies 和 oauth-server 模块中添加以下代码：

```xml
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper-spring-boot-starter</artifactId>
                <version>2.1.5</version>
            </dependency>
```

然后刷新项目。

当引入了依赖以后，我们需要在启动类中加入包扫描相关的配置信息：

```java
package com.chen.oauth2.server;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;

/**
 * @Author: ChromeChen
 * @Description:
 * @Date: Created in 12:18 2020/10/7 0007
 * @Modified By:
 */
@SpringBootApplication
@MapperScan(basePackages = "com.chen.oauth2.server.mapepr")
public class Oauth2ServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(Oauth2ServerApplication.class, args);
    }
}
```

注意：需要使用 tk.mybatis 的包扫描器。

修改 application.yml 配置文件，添加以下配置信息：

```yaml
mybatis:
  mapper-locations: classpath:mapper/*.xml
```

此时，启动项目，测试是否报错。如果没有报错，那么说明集成 tk.mybatis 成功。

#### 6.3.3 实现“认证”与“授权”的代码

使用 Oauth2 时，我们需要把认证与授权的工作交给框架去做。在移交工作以前，我们需要将“认证”的信息，与“授权”的信息都查询出来，然后交给框架。



新建一个 `UserDetailsService` 的实现类 `UserDetailsServiceImpl`，重写该类的 `loadUserByUsername `方法。

```java
package com.chen.oauth2.server.config.service;

import com.chen.oauth2.server.domain.TbPermission;
import com.chen.oauth2.server.domain.TbUser;
import com.chen.oauth2.server.service.ITbPermissionService;
import com.chen.oauth2.server.service.ITbUserService;
import org.assertj.core.util.Lists;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Author: ChromeChen
 * @Description:
 * @Date: Created in 14:56 2020/10/29 0029
 * @Modified By:
 */
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private ITbUserService tbUserService;
    @Autowired
    private ITbPermissionService tbPermissionService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 查询用户信息
        TbUser tbUser = tbUserService.getByUsername(username);
        List<GrantedAuthority> grantedAuthorities = Lists.newArrayList();
        if (tbUser != null) {
            List<TbPermission> tbPermissions = tbPermissionService.selectByUserId(tbUser.getId());
            // 声明用户授权
            tbPermissions.forEach(tbPermission -> {
                if (tbPermission != null && tbPermission.getEnname() != null) {
                    GrantedAuthority grantedAuthority = new SimpleGrantedAuthority(tbPermission.getEnname());
                    grantedAuthorities.add(grantedAuthority);
                }
            });
        }

        // 由框架完成认证工作
        return new User(tbUser.getUsername(), tbUser.getPassword(), grantedAuthorities);
    }
}

```

从上面的步骤中可以看出：在该方法返回以前，首先需要：

1. 查询用户的信息：该步骤的具体代码实现位于 `TbUserService `中；
2. 根据用户 ID 查询权限信息（权限配置信息在上一个章节的数据库脚本中），具体代码实现位于 `TbPermisstionService `中；

在获取了用户信息 + 权限信息以后，返回一个 `User `对象，由框架完成认证工作。

> 1. `TbUserService  `的实现代码：
>
>    ```java
>    @Service
>    public class TbUserServiceImpl implements ITbUserService {
>    
>        @Autowired
>        private TbUserMapper userMapper;
>    
>        @Override
>        public TbUser getByUsername(String username) {
>            Example example = new Example(TbUser.class);
>            example.createCriteria().andEqualTo("username", username);
>            return userMapper.selectOneByExample(example);
>        }
>    }
>    ```
>
> 2. `TbPermisstionService  `的实现代码：
>
>    ```java
>    @Service
>    public class TbPermissionService implements ITbPermissionService {
>    
>        @Autowired
>        private TbPermissionMapper permissionMapper;
>    
>        @Override
>        public List<TbPermission> selectByUserId(Long userId) {
>            return permissionMapper.selectByUserId(userId);
>        }
>    }
>    ```
>
>    xml 代码如下：
>
>    ```xml
>    <select id="selectByUserId" resultType="com.chen.oauth2.server.domain.TbPermission">
>        SELECT
>        p.*
>        FROM
>        tb_user AS u
>        LEFT JOIN tb_user_role AS ur ON u.id = ur.user_id
>        LEFT JOIN tb_role_permission AS rp ON ur.role_id = rp.role_id
>        LEFT JOIN tb_permission AS p ON rp.permission_id = p.id
>        WHERE u.id = #{userId}
>    </select>
>    
>    ```
>
>    

#### 6.3.4 修改服务器安全配置

新建 `WebSecurityConfigurerAdapter `的子类 `WebSecurityConfiguration`，并在该类上添加以下注解：

- `@Configuration`
- `@EnableWebSecurity`
- `@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)`：全局方法拦截；

该配置类充当了全局拦截器。

类中的具体配置信息如下：

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        // 设置默认的加密方式
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public UserDetailsService userDetailsService() {
        return new UserDetailsServiceImpl();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 使用自定义认证与授权
        auth.userDetailsService(userDetailsService());
    }
}
```

> 通过对比 RBAC 和基于内存存储用户名密码 这两种方式，可以发现：
>
> 1. 基于 RBAC 方式实际上就是把用户名密码与权限的信息转移到了『数据库』中，从数据库中查询信息。
> 2. 由于用户名密码存放在了 user 表中，而权限信息保存在了 permission 表中，因此，为了业务解耦，需要生成这两个模块相应的 Mapper 以及业务操作代码；



#### 6.3.5 获取 token

启动项目，然后获取 token。

具体步骤请查看 [5.5.9 访问并获取 token](#5.5.9 访问并获取 token)

> Session 在设计之处，就不能不允许存放大量内容。互联网应用经常会出现高并发场景，所以：互联网应用必须和 Session 分离

#### 6.3.6 总结

本个 demo 是在《基于 jdbc 存储令牌》的基础上，逐步过渡到《基于 RBAC 角色的访问控制》。本质上，就是将『基于内存存储的“认证”与“授权”』的信息，移送到了『数据库』中，由此需要做好数据库的初始化工作、项目连接数据库的工作、业务代码查询数据库的工作。

关于如何更加顺利地过渡，可以使用『渐进式』过渡方式：首先初始化数据库的内容，然后使用 SQL 测试；接着在项目中添加和数据库连接有关的功能，并完成简单测试；然后生成相对应的 PO、Mapper 以及业务类；最后完成代码的修改。









