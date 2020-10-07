---
title: X01-关于 Spring 项目工程的依赖
time: 2020年10月7日

---

[toc]

## 一、简介

介绍了 Spring 项目架构的基本步骤。

项目架构为：

1. 总工程项目；
2. 一个专门的依赖项目，管理整个工程的所有版本依赖；
3. 其他模块；

## 二、具体步骤以及注意点

依赖关系：

1. 总工程的 pom 文件位于最外层；
2. 依赖项目位于总工程项目下，但是**<u>不是</u>**总工程的子项目；
3. 总工程 pom 文件需要添加一个 dependencyManagement，在这个里面指定依赖项目
4. 其余的项目均为总工程项目的子项目；

### 2.1 总工程项目

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

    <modules>
    </modules>

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    </properties>


    <!-- 这里指定依赖项目 -->
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
</project>

```

### 2.2 依赖项目

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

    <!-- 这里放置通用的依赖 -->
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

</project>

```



