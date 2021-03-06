---
title: 11-Alibaba Nacos
time: 2020年7月29日

---

[toc]

## 一、Spring Alibaba Nacos 简介



## 二、Alibaba Nacos 安装部署

### 2.1 后台部署

下载复制项目到服务器机器中：

```shell
git clone https://github.com/nacos-group/nacos-docker.git
cd nacos-docker
```

使用单机模式启动：

```shell
docker-compose -f example/standalone-mysql-8.yaml up -d
```

在启动过程中可以查看日志：

```shell
docker-compose -f example/standalone-mysql-8.yaml logs -f
```

> 注意：在最新版本中，*standalone*有两个 yaml 文件，粗略浏览，发现两个文件的数据库版本不相同。我们可以使用 docker-compose 指令启动任意一个。

### 2.2 进入 Nacos 控制台

Nacos 控制台如下：$IP 地址/端口号$/nacos

我们可以使用 docker ps 查看端口号：

```shell
root@ubuntu:/usr/local/docker/nacos/nacos-docker# docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS              PORTS                                            NAMES
31b715976ec4        nacos/nacos-server:1.3.1   "bin/docker-startup.…"   8 minutes ago       Up 8 minutes        0.0.0.0:8848->8848/tcp, 0.0.0.0:9555->9555/tcp   nacos-standalone-mysql
2681a2d17f67        nacos/nacos-mysql:8.0.16   "docker-entrypoint.s…"   8 minutes ago       Up 8 minutes        0.0.0.0:3306->3306/tcp, 33060/tcp                mysql
```

可以看到，nacos 的控制台端口为 8848。

输入对应的 IP 地址和端口号以后，出现以下界面：

![img](11-Alibaba-Nacos.assets/FrTiikLO9cmZrsgZJq4GY8GyEMGz@.webp)

- **账号：** nacos
- **密码：** nacos

登陆以后：

![img](11-Alibaba-Nacos.assets/Fi1sfbBlAEaxgEIxdjfDssxR6IwU@.webp)



## 三、服务注册与发现

本节通过一个简单的例子来讲解 nacos 如何实现服务注册与发现。

### 3.1 概述

![img](11-Alibaba-Nacos.assets/FijfCmGEHHA6WnI5NwXkzzdn1S15@.webp)

如图：

1. 两个 APP 分别注册到 Nacos 中去，此时 Nacos 的 IP 列表池中保存了这两个 APP 的 IP 地址以及端口；
2. 当客户端访问$ APP_{consumer} $的时候， $APP_{consumer} $会通过 Nacos 来寻找到对应的 $APP_{provider} $的地址，并返回给  $APP_{consumer} $。此后， $APP_{consumer}$ 就可以通过这个地址调用到对应的服务了（**一般情况下，只有$  APP_{consumer} $ 可以把客户端直接访问，而且访问方式是 REST，而 $APP_{provider}$ 则是无法通过客户端直接访问的**）

### 3.2 创建服务提供者 Provider

pom 文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.chen</groupId>
        <artifactId>00-hello-spring-cloud-alibaba-dependencies</artifactId>
        <version>1.0-SNAPSHOT</version>
        <!--<relativePath>../pom.xml</relativePath>-->
    </parent>

    <artifactId>01-hello-spring-cloud-alibaba-provider</artifactId>
    <packaging>jar</packaging>
    <url>http://www.funtl.com</url>
    <inceptionYear>2018-Now</inceptionYear>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <developers>
        <developer>
            <id>liwemin</id>
            <name>Lusifer Lee</name>
            <email>lee.lusifer@gmail.com</email>
        </developer>
    </developers>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring-cloud-alibaba.verion}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.spring.cloud.alibaba.provider.ProviderApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

配置文件如下所示：

```yml
spring:
  application:
    name: service-provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.150.129:8848

server:
  port: 8070
```

这里的 `spring.cloud.nacos.discovery.server-addr`填写的值是当前 Nacos 服务端的地址。

启动类如下所示：

```java
@SpringBootApplication
@EnableDiscoveryClient	// ① 加上这个注解，可以被 Nacos 发现
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

启动成功以后，我们访问 Nacos UI 界面，可以看到以下的信息：

![image-20200729182052519](11-Alibaba-Nacos.assets/image-20200729182052519.png)

### 3.3 服务消费者 consumer

pom 文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>00-hello-spring-cloud-alibaba-dependencies</artifactId>
        <groupId>com.chen</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>03-hello-spring-cloud-alibaba-consumer</artifactId>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <developers>
        <developer>
            <id>liwemin</id>
            <name>Lusifer Lee</name>
            <email>lee.lusifer@gmail.com</email>
        </developer>
    </developers>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
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

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.spring.cloud.alibaba.consumer.ConsumerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

配置文件如下所示：

```yml
spring:
  application:
    name: service-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.150.129:8848

server:
  # 服务端口
  port: 8080

management:
  # 端点检查（健康检查）
  endpoints:
    web:
      exposure:
        include: "*"
```

启动类如下所示：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

> 总结：
>
> 1. 如果想要使用 nacos 的服务注册与发现功能，需要在 pom 文件中加入 spring-cloud-starter-alibaba-nacos-discovery 的依赖；
> 2. 通过两个例子可以发现：无论是『服务消费者』还是『服务提供者』，注册到 Nacos 的基本配置都是一致的，需要： ① 加入依赖； ② yml 配置文件制定 Nacos 的地址； ③ 在启动类中加入注解 @EnableDiscoveryClient

### 3.4 提供方与消费方通信

我们首先在提供方中创建一个控制器 $C_1$，在 $C_1 $中只需要直接输出一个简单的字符串即可：

```java
@RequestMapping(value = "/echo/{string}", method = {RequestMethod.GET})
public String echo(@PathVariable String string) {

    return "This is Provider message : " + string;
}
```

接着在消费方创建一个控制器 $C_2$，然后在该控制器中调用 $C_1 $的接口即可：

```java
@RequestMapping(value = "/echo/{str}", method = {RequestMethod.GET})
public String echo(@PathVariable String str) {
    // 使用服务名请求服务提供者
    return restTemplate.getForObject("http://service-provider/echo/" + str + "从『服务消费者』转到『服务提供者』……", String.class);
}
```

> 这里我们使用了 RestTemplate 来实现调用，调用的代码有以下的特点：
>
> 1. 协议为 HTTP 协议，更具体地说，是 REST调用；
> 2. 直接通过服务名 *service-provider* 进行调用，而不是通过 IP 地址和端口；

> 上面的代码中，我们还缺少了 `RestTemplate  `的配置以及注入。
>
> 1. RestTemplate  配置类：该类的目的是创建一个名为 `restTemplate `的 Bean：
>
>    ```java
>    @Configuration
>    public class ConsumerConfiguration {
>    
>        @Bean
>        @LoadBalanced
>        public RestTemplate restTemplate() {
>            return new RestTemplate();
>        }
>    }
>    ```
>
> 2. 注入到控制器 $C_2 $中：
>
>    ```java
>    private final RestTemplate restTemplate;
>    
>    @Autowired
>    public AbcController(RestTemplate restTemplate) {
>        this.restTemplate = restTemplate;
>    }
>    ```
>
>    在上面的过程中，我们使用了关键字 `private final ` 以保证线程安全。更加具体的原因，请查看：[【Spring】浅谈spring为什么推荐使用构造器注入](https://www.cnblogs.com/joemsu/p/7688307.html)。



编写完成以下的代码以后，我们访问消费者的接口，会得到以下的提示语：

![image-20200729183809615](11-Alibaba-Nacos.assets/image-20200729183809615.png)

说明我们通过 `RestTemplate `实现了服务名调用。



## 四、 使用 Feign 通信

### 4.1 什么是 Feign

Feign 是一个声明式的伪 HTTP 客户端，它使得写 HTTP 客户端变得更简单。使用 Feign，只需要创建一个接口并注解。它具有可插拔的注解特性，可使用 Feign 注解和 JAX-RS 注解。Feign 支持可插拔的编码器和解码器。Feign 默认集成了 Ribbon，Nacos 也很好的兼容了 Feign，默认实现了负载均衡的效果

- Feign 采用的是基于接口的注解
- Feign 整合了 Ribbon



### 4.2 步骤

使用 Feign 需要在调用方中添加依赖，然后通过注解声明调用哪一个请求即可。

**添加依赖：**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



**开在启动类中开启 Feign 功能**

通过 `@EnableFeignClients` 注解开启 Feign 功能：



**创建业务类，声明调用**

我们当前项目中创建一个接口，在接口中加入声明式调用即可：

```java
/**
 * @Author: ChromeChen
 * @Description: 该接口用于 Feign 声明式调用
 * @Date: Created in 14:17 2020/8/7 0007
 * @Modified By:
 */
@FeignClient(value = "service-provider")	// ① 
public interface EchoService {

    @GetMapping(value = "/echo/{string}")	// ②
    String echo(@PathVariable("string") String string);
}

```

①：调用哪一个服务；

②：调用的具体方法；



**测试是否成功：**

我们启动服务提供者以及服务消费者两个项目，然后通过访问消费者的请求，可以得到从提供者返回的消息：

![image-20200807142731408](11-Alibaba-Nacos.assets/image-20200807142731408.png)



### 4.3 通过 Feign 实现负载均衡

#### 4.3.1 IDEA 如何运行多个服务实例

- 修改 `service-provider` 服务的端口号如 8071，并启动多个实例，IDEA 中依次点击 **Run** -> **Edit Configurations** 并勾选 **Allow parallel run** 以允许 IDEA 多实例运行项目

![img](11-Alibaba-Nacos.assets/Fpde-B7pVpH_PpKzuQu6gPMA0-Ck@.webp)

- 再次启动 `service-provider` 实例，IDEA Spring Boot 面板可以看到

![img](11-Alibaba-Nacos.assets/FpAamUxYdshLFlDgc-Nkpbysdu3s@.webp)

- Nacos Server 控制台可以看到 `service-provider` 有 2 个实例

![img](11-Alibaba-Nacos.assets/FlTZdqtUY6LRWl-rWBdDFSJNZMs8@.webp)



### 4.3.2 实现负载均衡

#### 4.3.2.1 服务提供者添加方法

首先需要在服务提供者项目 service-provider 中添加方法：

```java
@Value("${server.port}")
private String port;
@GetMapping("/echo/lb")
public String lb(HttpServletRequest request, HttpServletResponse response ) {

    return "当前的服务的端口是：" + port;
}
```

控制器中返回了当前服务实例的端口。

#### 4.3.2.2 服务消费者添加方法

服务消费者中需要调用服务提供者的方法，以及提供对外接口：

**Feign 调用**

在 service-consumer 的 EchoService 中添加以下 Feign 调用：

```java
@GetMapping("/echo/lb")
String lb();
```

**对外接口：**

```java
@GetMapping("/echo/lb")
public String lb(HttpServletRequest request, HttpServletResponse response ) {

    return "从服务提供者返回的消息为：" + echoService.lb();
}
```

#### 4.3.2.3 测试

我们启动的实例为：

1. 1 个 `service-consumer` 服务；
2. 2 个 `service-provide` 服务，端口分配分别为：8070 以及 8071；

启动完成以后状态如下：

![image-20200807152838668](11-Alibaba-Nacos.assets/image-20200807152838668.png)

接着我们访问服务消费者的接口，可以得到以下的消息：

![image-20200807153019220](11-Alibaba-Nacos.assets/image-20200807153019220.png)

![image-20200807153025822](11-Alibaba-Nacos.assets/image-20200807153025822.png)

> 上面通过不断刷新页面，我们会发现端口号每次都会有所变化。

控制台中 service-consumer 会出现以下日志

![image-20200807151844074](11-Alibaba-Nacos.assets/image-20200807151844074.png)

#### 4.3.2.3 总结

Feign 调用只需要在服务调用方添加，当服务提供者与服务消费者同时注册到同一个注册中心的时候，它们就可以实现通信；Feign 调用时，默认会使用的负载均衡算法为：『轮询算法』。



> 常见的负载均衡的策略：
>
> 参考[常见的负载均衡算法](https://www.jianshu.com/p/1e56ace862e7)；



## 五、Alibaba Nacos 分布式配置中心

### 5.1 概述

在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件

### 5.2 什么是 Nacos Config

Nacos 提供用于存储配置和其他元数据的 key/value 存储，为分布式系统中的外部化配置提供服务器端和客户端支持。使用 Spring Cloud Alibaba Nacos Config，您可以在 Nacos Server 集中管理你 Spring Cloud 应用的外部属性配置。

Spring Cloud Alibaba Nacos Config 是 Spring Cloud Config Server 和 Client 的替代方案，客户端和服务器上的概念与 Spring Environment 和 PropertySource 有着一致的抽象，在特殊的 bootstrap 阶段，配置被加载到 Spring 环境中。当应用程序通过部署管道从开发到测试再到生产时，您可以管理这些环境之间的配置，并确保应用程序具有迁移时需要运行的所有内容。

### 5.3 接入配置中心

#### 5.3.1 添加依赖

在服务消费者 consumer 项目中添加以下依赖：

```xml
 <!-- 分布式配置中心 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

注意：不要忘记在 dependencies 项目中添加改依赖的版本管理。

> 或者使用以下依赖：
>
> ```xml
>  <!-- 分布式配置中心 -->
> <dependency>
>     <groupId>org.springframework.cloud</groupId>
>     <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
> </dependency>
> 
> ```
>
> 

#### 5.3.2 使用 Nacos 控制台发布配置

![img](11-Alibaba-Nacos.assets/Fv3i7ReI67lcDvL9hcfgxYletGGW@.webp)

然后添加以下配置：

![image-20200811170626733](11-Alibaba-Nacos.assets/image-20200811170626733.png)

配置的内容为：

```yaml
spring:
  application:
    name: service-consumer
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.150.129:8848

server:
  port: 8080

management:
  endpoints:
    web:
      exposure:
        include: "*"

user:
  name: "灶门炭治郎"
```

> 在上面的配置中，如果 Data ID 没有明确写后缀名成，那么默认后缀名称为 `.properties`。

#### 5.3.3 添加启动文件

在项目中添加配置文件 bootstrap.properties，然后添加以下配置：

```properties
spring.application.name=service-consumer-config
spring.cloud.nacos.config.server-addr=192.168.150.129:8848
spring.cloud.nacos.config.file-extension=yaml
```

最后启动项目即可。



#### 5.3.4 验证配置文件实时更新

首先我们在控制器中加入注解：`@RefreshScope` ，在控制器中添加方法：

```java
@Value("${user.name}")
private String username;

@GetMapping("/echo/config")
public String config(HttpServletRequest request, HttpServletResponse response ) {

    return String.format("配置文件中的值为：【%s】", username);
}
```

然后启动项目，输入对应的地址，输出：

![image-20200811171751237](11-Alibaba-Nacos.assets/image-20200811171751237.png)

我们在 Nacos 控制台中修改对应的键值，发布配置文件，

![image-20200811171912972](11-Alibaba-Nacos.assets/image-20200811171912972.png)

输出：

![image-20200811171926109](11-Alibaba-Nacos.assets/image-20200811171926109.png)



## 六、Alibaba Nacos 多环境配置

### 6.1 概述

我们在做项目开发的时候，生产环境和测试环境的一些配置可能会不一样，有时候一些功能也可能会不一样，所以我们可能会在上线的时候手工修改这些配置信息。Spring 为我们提供了 Spring Boot Profile 这个功能（Maven 也为我们提供了 Maven Profile）。我们只需要在启动的时候添加一个虚拟机参数，激活自己环境所要用的 Profile 就可以了。

操作起来很简单，只需要为不同的环境编写专门的配置文件，如：`application-dev.yml`、`application-prod.yml`， 启动项目时只需要增加一个命令参数 `--spring.profiles.active=` 环境配置即可

```
java -jar 1.0.0-SNAPSHOT.jar --spring.profiles.active=prod
```

### 6.2 什么是 Nacos Config Profile（未实践）

`spring-cloud-starter-alibaba-nacos-config` 在加载配置的时候，不仅仅加载了以 dataid 为 `${spring.application.name}.${file-extension:properties}` 为前缀的基础配置，还加载了 dataid 为 `${spring.application.name}-${profile}.${file-extension:properties}` 的基础配置。在日常开发中如果遇到多套环境下的不同配置，可以通过 Spring 提供的 `${spring.profiles.active}` 这个配置项来配置。

#### 6.2.1 使用 Nacos Config Profile

我们以 `service-provider` 项目为例，演示多环境配置效果，不要忘记依赖 Nacos Config Starter

```
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

#### 6.2.2 使用控制台发布配置

> **注意：** 控制台发布配置时不要加注释，否则打成 Jar 包后运行会报无法解析配置文件的错误

通过浏览器访问 http://192.168.141.132:8848/nacos ，访问 Nacos Server

- 发布一个名为 `service-consumer-config.yaml` 的 **测试环境** 配置

```
spring:
  application:
    # 服务名
    name: service-provider
  cloud:
    nacos:
      discovery:
        # 服务注册中心
        server-addr: 192.168.141.132:8848

server:
  # 服务端口
  port: 8070

management:
  # 端点检查（健康检查）
  endpoints:
    web:
      exposure:
        include: "*"
```

- 发布一个名为 `service-consumer-config-prod.yaml` 的 **生产环境** 配置

```
spring:
  application:
    # 服务名
    name: service-provider
  cloud:
    nacos:
      discovery:
        # 服务注册中心
        server-addr: 192.168.141.132:8848

server:
  # 修改了上面的端口号，区分配置的不同
  port: 8071

management:
  # 端点检查（健康检查）
  endpoints:
    web:
      exposure:
        include: "*"
```

#### 6.2.3 修改客户端配置

- 创建名为 `bootstrap.properties` 的配置文件并删除之前创建的 `application.yml` 配置文件

```
spring.application.name=service-provider-config
spring.cloud.nacos.config.server-addr=192.168.141.132:8848
spring.cloud.nacos.config.file-extension=yaml
```

- 创建名为 `bootstrap-prod.properties` 的配置文件

```
spring.profiles.active=prod
spring.application.name=service-provider-config
spring.cloud.nacos.config.server-addr=192.168.141.132:8848
spring.cloud.nacos.config.file-extension=yaml
```

#### 6.2.4 测试多环境配置

此时我们有两个配置文件，分别为 `bootstrap.properties` 和 `bootstrap-prod.properties` ，我们需要指定启动时加载哪一个配置文件

**Run** -> **Edit Configurations** -> **Active profiles:**

![img](11-Alibaba-Nacos.assets/FkNrV5jqJWruOQRCzH6Wb02d9Ctx@.webp)

运行项目并观察日志

![img](11-Alibaba-Nacos.assets/FrQRWv5fQfLWHi9A9Z5ltlFhWdft@.webp)

由上图可知，我们成功加载了不同环境的配置



​	

