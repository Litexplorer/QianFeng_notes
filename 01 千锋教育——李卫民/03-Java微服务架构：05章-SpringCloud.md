---
title:03-Java微服务架构：05章-SpringCloud
time:2019年10月17日
---

[toc]

## 〇、视频介绍





## 一、SpringCloud简介

SpringCloud是一个相对比较新的微服务框架，2016年才推出1.0的Release版本，但是其更新速度特别快，几乎每1~2个个月就有一次更新。虽然SpringCloud时间最短，但是相比Dubbo等RPC框架，SpringCloud提供的全套的分布式系统解决方案。

SpringCloud为开发者提供了在分布式系统（配置管理，服务发现，熔断，微代理，控制总线，一次性Token，全局锁，Leader选举，分布式Session，集群状态）中快速构建的工具，使用SpringCloud的开发者可以快速地启动服务或构建应用，同时能够快速和云平台资源进行对接。



## 二、入门

### 2.1 创建统一的依赖管理

新建一个项目hello-spring-cloud，并在该项目中添加一个文件夹hello-spring-cloud-dependencies，后者作为统一的依赖管理。在hello-spring-cloud-dependencies中加入一个pom文件，内容如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>


    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
    </parent>

    <!--打包方式使用“pom”方式-->
    <groupId>com.chen</groupId>
    <artifactId>hello-spring-cloud-dependencies</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>hello-spring-cloud-dependencies</name>
    <!--<url>http://www.funtl.com</url>-->
    <inceptionYear>2018-Now</inceptionYear>

    <properties>
        <!-- Environment Settings -->
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>

        <!-- Spring Settings：SpringCloud的版本号 -->
        <spring-cloud.version>2.0.4.RELEASE</spring-cloud.version>
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


    <build>
        <plugins>
            <!--  Compiler插件：指定jdk版本  -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <showWarnings>true</showWarnings>
                </configuration>
            </plugin>

            <!-- 打包 jar 文件时，配置 manifest 文件，加入 lib 包的 jar 依赖 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <addMavenDescriptor>false</addMavenDescriptor>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <configuration>
                            <archive>
                                <manifest>
                                    <!-- Add directory entries -->
                                    <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
                                    <addDefaultSpecificationEntries>true</addDefaultSpecificationEntries>
                                    <addClasspath>true</addClasspath>
                                </manifest>
                            </archive>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- resource -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
            </plugin>

            <!-- install -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-install-plugin</artifactId>
            </plugin>

            <!-- clean -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-clean-plugin</artifactId>
            </plugin>

            <!-- ant -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-antrun-plugin</artifactId>
            </plugin>

            <!-- dependency -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
            </plugin>
        </plugins>


        <pluginManagement>
            <plugins>
                <!-- Java Document Generate -->
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-javadoc-plugin</artifactId>
                    <executions>
                        <execution>
                            <phase>prepare-package</phase>
                            <goals>
                                <goal>jar</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>

                <!-- YUI Compressor (CSS/JS压缩) -->
                <plugin>
                    <groupId>net.alchim31.maven</groupId>
                    <artifactId>yuicompressor-maven-plugin</artifactId>
                    <version>1.5.1</version>
                    <executions>
                        <execution>
                            <phase>prepare-package</phase>
                            <goals>
                                <goal>compress</goal>
                            </goals>
                        </execution>
                    </executions>
                    <configuration>
                        <encoding>UTF-8</encoding>
                        <jswarn>false</jswarn>
                        <nosuffix>true</nosuffix>
                        <linebreakpos>30000</linebreakpos>
                        <force>true</force>
                        <includes>
                            <include>**/*.js</include>
                            <include>**/*.css</include>
                        </includes>
                        <excludes>
                            <exclude>**/*.min.js</exclude>
                            <exclude>**/*.min.css</exclude>
                        </excludes>
                    </configuration>
                </plugin>
            </plugins>
        </pluginManagement>


        <!-- 资源文件配置 -->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <excludes>
                    <exclude>**/*.java</exclude>
                </excludes>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
            </resource>
        </resources>
    </build>


<!--  第三方下载库  -->
    <repositories>
        <repository>
            <id>aliyun-repos</id>
            <name>Aliyun Repository</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>

        <repository>
            <id>sonatype-repos</id>
            <name>Sonatype Repository</name>
            <url>https://oss.sonatype.org/content/groups/public</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>

        <repository>
            <id>sonatype-repos-s</id>
            <name>Sonatype Repository</name>
            <url>https://oss.sonatype.org/content/repositories/snapshots</url>
            <releases>
                <enabled>false</enabled>
            </releases>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>

        <!--在使用非Release版本时，SpringCloud可能会引入一些快照版本的包，而这些包只会在Spring的仓库中，不会发布到其它地方，因此
            需要引入Spring仓库的地址-->
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>


</project>
```

关于标签的说明：

- parent：继承了 Spring Boot 的 Parent，表示我们是一个 Spring Boot 工程
- package：`pom`，表示该项目仅当做依赖项目，没有具体的实现代码（用来做项目依赖的）
- `spring-cloud-dependencies`：在 `properties` 配置中预定义了版本号为 `2.0.4.RELEASE` ，表示我们的 Spring Cloud 使用的是 Release 版
- build：配置了项目所需的各种插件（用最新的插件来替换maven默认的插件）
- repositories：配置项目下载依赖时的第三方库
  - 注意：Spring的仓库是一定需要被依赖进来的，具体的原因请查看注释。

> 在项目中，hello-spring-cloud根目录仅仅作为一个目录的形式存在，而不是一个工程（在分模块开发中就是一个工程存在）；而hello-spring-cloud-dependencies则是作为一个一个项目（或者说是一个服务）存在。



### 2.2 服务注册与发现

#### 2.2.1 概述

> 为了发送一个请求，您的代码需要知道服务实例的网络位置（IP地址与端口号），在运行于物理硬件上的传统应用中，服务实例的网络位置是相对静态的……
>
> ……
>
> 然而，在现代基于云的微服务应用中，这是一个更难解决的问题，……
>
> 服务实例具有动态分配的网络位置。此外，由于自动扩缩、故障与升级，整组服务实例会动态变更。因此，您的客户端代码需要使用更精准的服务发现机制。
>
> ——摘自《微服务——从设计到部署》

在这里，我们需要用的组件是 Spring Cloud Netflix 的 Eureka，Eureka 是一个服务注册和发现模块。



#### 2.2.2 搭建Eureka服务注册中心

**首先新建一个文件夹hello-spring-cloud-eureka，我们在这个文件夹内搭建Eureka服务注册中心。**

**接着在编写一个pom文件，文件内容如下：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent> ①
        <groupId>com.chen</groupId>
        <artifactId>hello-spring-cloud-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-eureka</artifactId>
    <packaging>jar</packaging> ②

    <name>hello-spring-cloud-eureka</name>
    <!--<url>http://www.funtl.com</url>-->
    <inceptionYear>2018-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>2.1.0.RELEASE</version> ③
        </dependency>
        <!-- Spring Cloud End -->



    </dependencies>

    <build>
        <plugins>
            <plugin> ④
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.chen.hello.spring.cloud.eureka.EurekaApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

①：父工程的id选择我们在[创建统一的依赖管理]()中创建的pom文件；

②：由于当前项目需要打包，所以我们选择的打包方式为`jar`；

③：引入SpringCloud版本（在视频教程和比克教程中没有引入版本号，但是在练习过程中如果不引入版本号的话会没有SpringCloud相关的jar包）；

④：指定入口类（视频说是必须指定，没验证）；入口类需要和当前项目的SpringBoot入口类的全限定类名一致；



编写入口类：

按照SpringBoot项目开发时编写入口类的步骤即可。编写完成入口类以后，在类上加上注解@`EnableEurekaServer`

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

> 关于Maven项目报名结构：
>
> 一般来说，包名结构的“基本包”需要包含项目的名称，例如，在本例中，项目名称是hello-spring-cloud-eureka，我们的“基本包”就设置为`com.chen.hello.spring.cloud.eureka`。接着再在基本包下创建具体和业务强相关的包。



编写application.yml文件

```yml
# 当前微服务的名称
spring:
  application:
    name: hello-spring-cloud-eureka

server:
  port: 8761
  
eureka:
  instance:
    hostname: localhost
  client:
#    如果当前的eureka服务是服务端的话，需要将下面两个属性的值改为false
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

上面配置较低Eureka服务端的端口配置为8761，并且`eureka.client.register-with-eureka`和 `eureka.client.fetch-registry`设置为`false`表示当前的Eureka服务是一个服务端。



启动项目以后，我们访问：`localhost:端口号`即可访问到以下界面：（注意：不用添加`/eureka/`）

![1571372642921](assets/1571372642921.png)



### 2.2.3 创建服务提供者

#### 2.2.3.1 概述

当 Client 向 Server 注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka Server 从每个 Client 实例接收心跳消息。 如果心跳超时，则通常将该实例从注册 Server 中删除。



#### 2.2.3.2 创建服务提供者

创建一个新的文件夹hello-spring-cloud-service-adm，并添加pom文件，文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.chen</groupId>
        <artifactId>hello-spring-cloud-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-service-admin</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-service-adm</name>
    <!--<url>http://www.funtl.com</url>-->
    <inceptionYear>2018-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.chen.hello.spring.cloud.service.adm.ServiceAdmApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

这个配置和Eureka的pom文件配置基本一致。



接着，我们需要编写启动类，并在启动类上添加注解`@EnableEurekaClien`：

```java
@SpringBootApplication
@EnableEurekaClient
public class ServiceAdmApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceAdmApplication.class, args);
    }
}
```



编写配置文件*application.yml*：

```yml
spring:
  application:
    name: hello-spring-cloud-service-admin

server:
  port: 8762

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/  
```

可以发现：

1. 服务提供者的eureka配置几乎和注册中心的eureka配置一致，而前者没有`eureka.client.register-with-eureka`和 `eureka.client.fetch-registry`的设置（即是使用默认设置`true`）；
2. `eureka.client.serviceUrl.defaultZone`表示向哪个注册中心注册；



接下来，我们可以在项目中新建一个控制器，用于向页面返回一个字符串：

```java
@RestController
public class AdminController {

    @Value("${server.port}")
    private String port;  ①

    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    public String sayHi(String message) {

        return String.format("你好， 你的消息是：%s I am from port: %s", message, port);
    }
}
```

①：当前微服务所在的端口（伏笔：在实现负载均衡时通过这个值来观察是否实现“负载”）；



启动服务，即可看到当前服务监听了8762端口：

![1571386049506](assets/1571386049506.png)



### 2.2.4 创建服务消费者（Ribbon方式）

#### 2.2.4.1  概述

 在微服务架构中，业务都会被拆分成一个独立的服务，服务与服务的通讯是基于 http restful 的。Spring cloud 有两种服务调用方式，一种是 ribbon + restTemplate，另一种是 feign。在这一篇文章首先讲解下基于 ribbon + rest。 



#### 2.2.4.2 负载均衡准备工作

负载均衡需要多个服务提供方（并运行“同样”的代码），因此，我们需要启动两个服务提供方。

> 在IDEA中启动相同SpringBoot项目的多个服务可以参考：

启动完成以后，我们可以在Eureka的管理界面中看到类似以下信息：

![1571386506328](assets/1571386506328.png)

②：这个警告的意思是：Eureka需要部署成为集群模式，（否则如果单点故障出现在Eureka服务中，那么这个系统就会奔溃）。



#### 2.2.4.3 创建服务消费者

**创建一个文件夹 hello-spring-cloud-web-admin-ribbon ，创建pom文件并在文件中添加以下内容：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.chen</groupId>
        <artifactId>hello-spring-cloud-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-web-admin-ribbon</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-web-admin-ribbon</name>
    <!--<url>http://www.funtl.com</url>-->
    <inceptionYear>2018-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <!-- Spring Cloud End -->

        <!-- 解决 thymeleaf 模板引擎一定要执行严格的 html5 格式校验问题 -->
        <dependency>
            <groupId>net.sourceforge.nekohtml</groupId>
            <artifactId>nekohtml</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.web.admin.ribbon.WebAdminRibbonApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

注意：在上面的内容中，加入了Thymeleaf相关的依赖和配置；（具体作用未知）



**编写启动类，并在启动类中添加注解  `@EnableDiscoveryClient`**  

```java
@SpringBootApplication
@EnableDiscoveryClient
public class WebAdminRibbonApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebAdminRibbonApplication.class, args);
    }
}
```

这个注解的作用是：在Eureka注册中心发现服务提供方；



**编写*application.yml*配置文件：**

```yml
spring:
  application:
    name: hello-spring-cloud-web-admin-ribbon
  thymeleaf:
    cache: false
    mode: LEGACYHTML5
    encoding: UTF-8
    servlet:
      content-type: text/html


server:
  port: 8764
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

```

上面配置文件添加了和Thymeleaf相关的配置。



**编写模板并开启负载均衡功能：**

```java
@Configuration
public class RestTemplateConfiguration {

    @Bean
    @LoadBalanced   // ①调用服务提供方时，会负载均衡地调用
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

①：见注释。



经过上面的配置，我们现在已经可以使用模板`restTemplate`调用服务提供方提供的服务了。接下来的步骤就是按照正常的开发流程开发即可：

（1）新建一个业务类，并在业务类中调用服务提供方提供服务：

```java
@Service
public class AdminService {

    @Resource
    private RestTemplate restTemplate;

    public String hi(String message) {
        return restTemplate
                .getForObject("http://HELLO-SPRING-CLOUD-SERVICE-ADMIN/hi?message=" + message,
                        String.class);  // ①
    }
}
```

①：使用`restTemplate.getForObject`方法进行调用，调用的第一个参数需要填写被调用方的地址，其地址的形式是：$http://Eureka中暴露的服务名称\ /具体api?参数$。第二个参数表示返回值类型的类类型。



（2）新建一个控制器，并在控制器中调用上面业务类中的方法：

```java
@RestController
public class AdminController {

    @Autowired
    private AdminService adminService;

    @RequestMapping(value = "hi", method = RequestMethod.GET)
    public String hi(@RequestParam String message) {
        String hi = adminService.hi(message);
        return hi;
    }
}
```



（3）启动该项目，启动完毕以后，（根据配置文件）该服务监听了8764端口。我们通过`http://localhost:8764/hi?message=某个参数`访问即可看到界面：

![1571387332734](assets/1571387332734.png)

不断刷新该界面，可以看到不同时刻访问的端口不相同：

![1571387413383](assets/1571387413383.png)

### 2.2.5 创建服务消费者（Feign方式）

#### 2.2.5.1 概述

Feign 是一个声明式的伪 Http 客户端，它使得写 Http 客户端变得更简单。使用 Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用 Feign 注解和 JAX-RS 注解。Feign 支持可插拔的编码器和解码器。Feign 默认集成了 Ribbon，并和 Eureka 结合，默认实现了负载均衡的效果

- Feign 采用的是基于接口的注解
- Feign 整合了 ribbon；



#### 2.2.5.2 创建服务消费者

在创建服务消费者以前，我们也是需要启动多个服务提供者实例。



1. 创建文件夹hello-spring-cloud-web-admin-feign，并在文件夹内添加pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.chen</groupId>
        <artifactId>hello-spring-cloud-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-web-admin-feign</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-web-admin-feign</name>
    <!--<url>http://www.funtl.com</url>-->
    <inceptionYear>2018-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <dependency> ①
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>2.1.0.RELEASE</version>
        </dependency>
        <!-- Spring Cloud End -->

        <!-- 解决 thymeleaf 模板引擎一定要执行严格的 html5 格式校验问题 -->
        <dependency>
            <groupId>net.sourceforge.nekohtml</groupId>
            <artifactId>nekohtml</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.chen.hello.spring.cloud.web.admin.feign.WebAdminFeignApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

①：添加了feign的依赖；



2. 编写application.yml文件

   ```yml
   spring:
     application:
       name: hello-spring-cloud-web-admin-feign
     thymeleaf:
       cache: false
       mode: LEGACYHTML5
       encoding: utf-8
       servlet:
         content-type: text/html
   
   
   server:
     port: 8765
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:8761/eureka/
   ```

   这个yml中也添加了Thymeleaf相关的内容；



3. 创建启动类，并添加两个注解：`@EnableFeignClients`和`@EnableDiscoveryClient`：

   ```java
   @SpringBootApplication
   @EnableFeignClients
   @EnableDiscoveryClient
   public class WebAdminFeignApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(WebAdminFeignApplication.class, args);
       }
   }
   ```



4. 创建service类，并添加注解：`@FeignClient`

   ```java
   @FeignClient("HELLO-SPRING-CLOUD-SERVICE-ADMIN")
   public interface AdminService {
   
       @RequestMapping(value = "hi", method = RequestMethod.GET)
       public String sayHi(@RequestParam("message") String message);    // 注意，必须带
   }
   ```

   注解`@FeignClien`t中的值指定服务提供方，在本例中`是HELLO-SPRING-CLOUD-SERVICE-ADMIN`

   > 注意：在参数中必须加上`@RequestParam("message")`（`message`指的是服务提供方的入参），否则会出现以下错误：feign.FeignException: status 405 reading AdminService#sayHi(String)异常



5. 创建控制器，并调用service层的方法：

   ```java
   @RestController
   public class AdminController {
   
       @Autowired
       private AdminService adminService;
   
       @RequestMapping(value = "/hi", method = RequestMethod.GET)
       public String hi(String message) {
           String hi = adminService.sayHi(message);
           return hi;
       }
   }
   ```



6. 启动该项目，启动完毕以后，（根据配置文件）该服务监听了8765端口。我们通过`http://localhost:8765/hi?message=某个参数`访问即可看到界面：
   ![1571457911095](assets/1571457911095.png)


   不断刷新这个请求，会发现不同时刻访问了不同的服务实例：
   ![1571457999913](assets/1571457999913.png)





















