---
title:03 Spring Security oAuth2 笔记
time: 2020年5月10日


---

[toc]

## 〇、 视频介绍

视频来源于千锋教育《Spring Security oAuth2》，在线观看地址为：https://www.bilibili.com/video/BV1Kb411u7VP



## 一、简介

Spring Security 是一个框架，而 oAuth 则是一个标准（或者称之为“协议”）。

实现 oAuth2.0 的框架有：

- Spring Security oAuth2；
- Shiro oAuth2；



### 1.1 什么是 oAuth

oAuth 协议为用户资源的授权提供了一个安全的、开放而有简易的标准。与以往的授权方式不同之处是：oAuth 的授权不会使第三方触及到用户的账号信息（如用户名和密码），即第三方无需使用用户的用户名与密码就可以申请获得用户资源的授权，因此 oAuth 是安全级的。



### 1.2 什么是 Spring Security

Spring Security 是一个**<u>安全框架</u>**，前身是 Acegi Security，能够为 Spring 企业应用系统提供声明式的安全访问控制。Spring Security 基于 Servlet 过滤器、IoC 和 AOP，为 Web 请求和方法调用提供身份确认和授权处理，避免了代码耦合，减少了大量重复代码工作。



## 二、 为什么需要 OAut2

### 2.1 应用场景

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

![image-20200510124130587](./03-Spring-Security-oAuth2-笔记.assets/image-20200510124130587.png)

> 注意：**第三方应用程序**：包括自家的 APP 也是；你自己的 APP 也是第三方



### 2.3 交互过程

oAuth 在 "客户端" 与 "服务提供商" 之间，设置了一个授权层（authorization layer）。"客户端" 不能直接登录 "服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端" 登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。"客户端" 登录授权层以后，"服务提供商" 根据令牌的权限范围和有效期，向 "客户端" 开放用户储存的资料。

![img](./03-Spring-Security-oAuth2-笔记.assets/Lusifer_201904010001.png)





## 三、开放平台

### 3.1 交互模型

涉及到以下几种角色：

- 资源拥有者：用户
- 客户端：APP
- 服务提供方：包含两个角色
  - 认证服务器
  - 资源服务器



认证服务器以及 oAuth2 开放平台：参考[认证服务器](https://www.funtl.com/zh/spring-security-oauth2/%E5%BC%80%E6%94%BE%E5%B9%B3%E5%8F%B0.html#%E4%BA%A4%E4%BA%92%E6%A8%A1%E5%9E%8B)。



## 四、令牌的访问与刷新

### 4.1 Access Token

Access Token 是客户端**<u>访问资源服务器</u>**的令牌。拥有这个令牌**<u>代表着得到用户的授权</u>**。然而，这个授权应该是 **临时** 的，有一定有效期。这是因为，Access Token 在使用的过程中 **可能会泄露**。给 Access Token 限定一个 **较短的有效期** 可以降低因 Access Token 泄露而带来的风险。

> 然而引入了有效期之后，客户端使用起来就不那么方便了。每当 Access Token 过期，客户端就必须重新向用户索要授权。这样用户可能每隔几天，甚至每天都需要进行授权操作。这是一件非常影响用户体验的事情。
>
> 于是 oAuth2 引入了 Refresh Token 机制



### 4.2 Refresh Token

Refresh Token 的作用是用来刷新 Access Token。认证服务器提供一个刷新接口，例如：

```text
http://www.funtl.com/refresh?refresh_token=&client_id=
```

传入 `refresh_token` 和 `client_id`，认证服务器验证通过后，返回一个新的 Access Token。为了安全，oAuth2.0 引入了两个措施：

- oAuth2.0 要求，Refresh Token **一定是保存在客户端的服务器上** ，而绝不能存放在狭义的客户端（例如 App、PC 端软件）上。调用 `refresh` 接口的时候，一定是从服务器到服务器的访问。
- oAuth2.0 引入了 `client_secret` 机制。即每一个 `client_id` 都对应一个 `client_secret`。这个 `client_secret` 会在客户端申请 `client_id` 时，随 `client_id` 一起分配给客户端。**客户端必须把 `client_secret` 妥善保管在服务器上**，决不能泄露。刷新 Access Token 时，需要验证这个 `client_secret`。

实际上的刷新接口类似于：

```text
http://www.funtl.com/refresh?refresh_token=&client_id=&client_secret=
```

以上就是 Refresh Token 机制。Refresh Token 的有效期非常长，会在用户授权时，随 Access Token 一起重定向到回调 URL，传递给客户端。



## 五、客户端授权模式

### 5.1 概述

客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。oAuth 2.0 定义了四种授权方式：

1. implicit：简化模式，不推荐使用；
2. authorization code：授权码模式（官方推荐使用的模式）；
3. resource owner password credentials：密码模式；
4. client credentials：客户端模式；



### 5.2 简化模式

简化模式适用于**<u>纯静态页面应用</u>**。一般通过 JS 代码实现控制。

在者中场景下，应用是没有持久化存储的能力的。因此，按照 oAuth2.0 的规定，这种应用是拿不到 Refresh Token 的。授权流程如下所示：

![img](./03-Spring-Security-OAuth2-笔记.assets/Lusifer_201904010002.png)

这种模式下，`access_token` 容易泄露且不可刷新。



### 5.3 授权码模式

授权码模式适用于有自己的服务器的应用，它是一个一次性的临时凭证，用来换取 `access_token` 和 `refresh_token`。认证服务器提供了一个类似这样的接口：

```text
https://www.funtl.com/exchange?code=&client_id=&client_secret=
```

1

需要传入 `code`、`client_id` 以及 `client_secret`。验证通过后，返回 `access_token` 和 `refresh_token`。一旦换取成功，`code` 立即作废，不能再使用第二次。流程图如下：

![img](./03-Spring-Security-OAuth2-笔记.assets/Lusifer_201904010003.png)

这个 code 的作用是保护 token 的安全性。上一节说到，简单模式下，token 是不安全的。这是因为在第 4 步当中直接把 token 返回给应用。而这一步容易被拦截、窃听。引入了 code 之后，即使攻击者能够窃取到 code，但是由于他无法获得应用保存在服务器的 `client_secret`，因此也无法通过 code 换取 token。而第 5 步，为什么不容易被拦截、窃听呢？这是因为，首先，这是一个从服务器到服务器的访问，黑客比较难捕捉到；其次，这个请求通常要求是 https 的实现。即使能窃听到数据包也无法解析出内容。

有了这个 code，token 的安全性大大提高。因此，oAuth2.0 鼓励使用这种方式进行授权，而简单模式则是在不得已情况下才会使用



### 5.4 密码模式

暴露密码给对方。一般用于信任关系足够的，同时需要各自定制化授权页面的两个产品。例如：一家公司的两个产品线，在不同产品的授权页面上使用相同的账号名和密码进行登陆。

密码模式参考：[密码模式](https://www.funtl.com/zh/spring-security-oauth2/%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%8E%88%E6%9D%83%E6%A8%A1%E5%BC%8F.html#%E5%AF%86%E7%A0%81%E6%A8%A1%E5%BC%8F)



### 5.5 客户端模式

客户端模式参考：[客户端模式](https://www.funtl.com/zh/spring-security-oauth2/%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%8E%88%E6%9D%83%E6%A8%A1%E5%BC%8F.html#%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%A8%A1%E5%BC%8F)；



学习一项技术，需要了解它的前世今生。



## 六、创建案例工程

本次我们的案例功能会涉及到以下几个模块：

1. 父工程模块；
2. 依赖模块；
3. 认证服务器模块；



首先，我们来创建第一和第二个模块：

### 6.1 创建父工程

创建父工程的 pom 文件，文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.chen</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <url>http://www.funtl.com</url>

    <modules>
        <!-- 工程模块请随着项目的不断完善自行添加 -->
        <module>spring-securoty-oauth2-dependency</module>
        <module>spring-security-oauth2-server</module>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.build.outputEncoding>UTF-8</project.build.outputEncoding>
    </properties>


    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.chen</groupId>
                <artifactId>spring-securoty-oauth2-dependency</artifactId>
                <version>${project.parent.version}</version>
                <type>pom</type>
                <scope>pom</scope>
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



### 6.2 创建依赖模块

依赖模块是一个子模块，模块的 pom 文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.chen</groupId>
    <artifactId>spring-securoty-oauth2-dependency</artifactId>
    <version>1.0.0-SNAPSHOT</version>

    <properties>
        <spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
    </properties>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

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



### 6.3 创建认证服务器

#### 6.3.1 概述

在 Oauth2 的协议中，用户要想访问对应的资源，必须先经过认证服务器的授权。授权过程有两个步骤：

1. 用户在“授权页面”上输入正确的用户名和密码；此时认证服务器就会重定向到某一个 uri 中，并且带上授权码；（对应下图的第三步和第四步）
2. 我们再发送一个请求，这个请求需要带上授权码，此时就可以拿到 token了（对应第五步）。

![image-20200524145728680](03-Spring-Security-OAuth2-笔记.assets/image-20200524145728680.png)

上面的分析中，我们可以发现有以下几个东西需要确定：

1. 授权页面是哪个？
2. 授权页面上的用户名和密码是什么？
3. 登陆成功以后，重定向的页面是哪个？
4. 获取 token 的请求是哪一个？



#### 6.3.2 创建认证服务器工程

首先创建一个 pom 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>spring-security-oauth2</artifactId>
        <groupId>com.chen</groupId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>spring-security-oauth2-server</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <!-- Spring Security-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
            <version>2.1.4.RELEASE</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.chen.spring.security.oauth2.OAuth2ServerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

上面直接依赖 `spring-cloud-starter-oauth2`即可依赖 `spring-cloud`了。



接着，我们创建一个对应的启动类以及配置文件：

```java
/**
 * @Author: ChromeChen
 * @Description: 认证服务器，用于获取 token
 * @Date: Created in 14:08 2020/5/24 0024
 * @Modified By:
 */
@SpringBootApplication
public class OAuth2ServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(OAuth2ServerApplication.class, args);
    }
}

```

application.yml

```yml
server:
  port: 8088
spring:
  application:
    name: oauth2-server
```



接下来，创建一个配置类，配置“授权页面”对应的信息了：

```java
/**
 * @Author: ChromeChen
 * @Description: 授权页面的配置信息
 * @Date: Created in 14:23 2020/5/24 0024
 * @Modified By:
 */
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled =true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 本次 demo 在内存中配置对应的账号和密码；在实际项目中，账号和密码应该是从数据库中读取得到的
        auth.inMemoryAuthentication()
                .withUser("admin").password(passwordEncoder().encode("123456")).roles("ADMIN")
                .and()
                .withUser("user").password(passwordEncoder().encode("123456")).roles("USER");
    }
}
```

上面的代码中配置了两组用户名和密码，其中，密码使用了加密的方式。此时，当我们访问了授权页面的时候，输入对应的正确的用户名和密码就可以获取到授权码了。

- `@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)`：全局方法拦截



创建一个配置类，用于获取 token ：

```java

/**
 * @Author: ChromeChen
 * @Description:
 * @Date: Created in 14:13 2020/5/24 0024
 * @Modified By:
 */
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients
                .inMemory()
                .withClient("client0")
                .secret(passwordEncoder.encode("secret0"))      // client_id 和 secrity_id 需要指定，只有输入了正确的才能访问授权服务器
                .authorizedGrantTypes("authorization_code")
                .scopes("app")
                .redirectUris("http://www.funtl.com");  // 获得授权码以后，将会跳转到哪里
    }
}
```

上面的`configure`方法中，client_id 和 secret 的配置是为了防止任何设备都能够获取 token，也就是说，在请求的时候必须带上对应的 client_id 和 secret 才能获取到 token（secret 需要加密）。

另外，登陆成功以后跳转的页面也是在这个地方配置的。



这时候，启动我们的项目，会在控制台中看到以下输出的信息：

![image-20200524151521109](03-Spring-Security-OAuth2-笔记.assets/image-20200524151521109.png)

#### 6.3.3 获取 token 步骤

在项目启动以后，我们访问下面的 URL：

```text
http://localhost:8088/oauth/authorize?client_id=client0&response_type=code
```

- 请求的 URL 为：localhost:端口康/oauth/authorize；
- 参数 $client\_id$的值为 配置类 `AuthorizationServerConfiguration`中指定的 `client_id` 的值（在本例中是 $client0$）；
- ~~response_type 参数的值为类 AuthorizationServerConfiguration 中指定的~~

会出现以下页面：

![image-20200524152159256](03-Spring-Security-OAuth2-笔记.assets/image-20200524152159256.png)

输入对应的正确的用户名和密码即可跳转到有权页面中：

![image-20200524152311711](03-Spring-Security-OAuth2-笔记.assets/image-20200524152311711.png)

将 scope_app 的值修改为 Approve 以后，点击 Authorize 按钮，即可跳转到 URL 中：

![image-20200524152448871](03-Spring-Security-OAuth2-笔记.assets/image-20200524152448871.png)

我们可能看到请求链接中包含了一个授权码。至此，授权码我们已经拿到了。

下面我们看看如何获取 token：

首先打开 postman 并输入下面的地址：

```text
http://client0:secret0@localhost:8088/oauth/token
```

- 请求的地址为：http://client_id的值:secret的值@IP地址:端口号/oauth/token



![image-20200524152657309](03-Spring-Security-OAuth2-笔记.assets/image-20200524152657309.png)

1. 请求方式为 POST；
2. 请求时需要带上相关的参数，参数的发送方式为 x-www-form-urlencoded；
3. 带上注册码；

发送请求以后，可以得到以下结果：

![image-20200524153024506](03-Spring-Security-OAuth2-笔记.assets/image-20200524153024506.png)

此时，我们已经拿到了 token.

> 如果我们再次发送同样的请求，会出现以下页面：
>
> ![image-20200524153104017](03-Spring-Security-OAuth2-笔记.assets/image-20200524153104017.png)
>
> 提示：非法的授权码。这说明，授权码在获取 token 的时候只能使用一次。



### 6.4 改造认证服务器项目——从数据库中读取 client_id 和 secret

上面的项目中，我们使用的是从内存中加载 client_id 和 secret 的，现在我们将这些信息配置到数据库中。

#### 6.4.1 创建数据库表

数据库表如下所示：

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

执行完脚本样本，得到以下几张表：

![image-20200524154824018](03-Spring-Security-OAuth2-笔记.assets/image-20200524154824018.png)



#### 6.4.2 在认证服务器项目中配置数据库信息

添加依赖：

```xml
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

需要在 依赖模块 中添加，也需要在 认证服务器模块 中添加。上面两个依赖的具体版本没有指定。

> 添加了 HikariCP 依赖（**号称全球最快的数据库连接池**）以后，我们启动项目，可以在控制台中看到以下输出：
>
> ![image-20200524160103036](03-Spring-Security-OAuth2-笔记.assets/image-20200524160103036.png)



application.yml 文件如下：

```yml
server:
  port: 8088
spring:
  application:
    name: oauth2-server
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.jdbc.Driver
    jdbc-url: jdbc:mysql://localhost:3306/oauth2?useUnicode=true&characterEncoding=utf-8&useSSL=false
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
```

修改配置类`AuthorizationServerConfiguration`如下：

```java

/**
 * @Author: ChromeChen
 * @Description:
 * @Date: Created in 14:13 2020/5/24 0024
 * @Modified By:
 */
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    /**
     * 加载数据源
     * @return
     */
    @Bean
    @Primary
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource() {
        // 配置数据源（注意，我使用的是 HikariCP 连接池），以上注解是指定数据源，否则会有冲突
        return DataSourceBuilder.create().build();
    }

    /**
     * 令牌保存到数据库中
     * @return
     */
    @Bean
    public TokenStore tokenStore() {
        return new JdbcTokenStore(dataSource());
    }

    /**
     * 需要事先在数据库中配置客户信息
     * @return
     */
    @Bean
    public ClientDetailsService jdbcClientDetails() {
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
        clients.withClientDetails(jdbcClientDetails());
    }
}

```

注意点：

1. 使用了 Hikari 数据源时，如果不添加额外的配置，可能会和 Spring 默认的数据源冲突，详细的配置见上面的代码。

   > 冲突后产生的可能异常为：
   > ![image-20200524162600148](03-Spring-Security-OAuth2-笔记.assets/image-20200524162600148.png)



#### 6.4.3 在数据库中添加配置信息

在数据库表 oauth_client_details 中添加一行记录：

![image-20200524162504591](03-Spring-Security-OAuth2-笔记.assets/image-20200524162504591.png)



#### 6.4.4 测试

启动项目，按照上一个章节的步骤进行请求即可。

请求得到 token 以后，会在表 oauth_access_token 中增加一条记录

![image-20200524163831389](03-Spring-Security-OAuth2-笔记.assets/image-20200524163831389.png)







