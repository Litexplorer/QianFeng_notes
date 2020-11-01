---
title: 19-Spring-Cloud-Alibaba-全家桶整合
time: 2020年11月1日

---

[toc]

## 〇、介绍

笔记来源于教程 [Spring Cloud Alibaba](https://www.funtl.com/zh/spring-cloud-alibaba/#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91)。视频在线观看地址为：https://www.bilibili.com/video/BV1ht411h772?p=1

## 一、Spring Cloud Alibaba 简介

### 1.1 概述

**2018 年 10 月 31 日的凌晨，这个伟大的日子里，Spring Cloud Alibaba 正式入驻了 Spring Cloud 官方孵化器，并在 Maven 中央库发布了第一个版本。**

[Spring Cloud for Alibaba 0.2.0 released](https://spring.io/blog/2018/10/30/spring-cloud-for-alibaba-0-2-0-released)

> The Spring Cloud Alibaba project, consisting of Alibaba’s open-source components and several Alibaba Cloud products, aims to implement and expose well known Spring Framework patterns and abstractions to bring the benefits of Spring Boot and Spring Cloud to Java developers using Alibaba products.

> Spring Cloud for Alibaba，它是由一些阿里巴巴的开源组件和云产品组成的。这个项目的目的是为了让大家所熟知的 Spring 框架，其优秀的设计模式和抽象理念，以给使用阿里巴巴产品的 Java 开发者带来使用 Spring Boot 和 Spring Cloud 的更多便利。

Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用微服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里微服务解决方案，通过阿里中间件来迅速搭建分布式应用系统。

### 1.2 主要功能

- 服务限流降级：默认支持 Servlet、Feign。RestTemplate、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控；
- 服务注册与发现：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。
- 分布式配置管理：支持分布式系统中的外部化配置，配置更改时自动刷新；
- 消息驱动能力：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力；
- 阿里云对象存储：阿里云提供的海量、安全、低成本、高可用的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- 分布式任务调度：提供妙极、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格服务。网格任务支持海量子任务均匀分配到所逼的 worker（schedux-client）上执行；

### 1.3 组件

- **Sentinel**：把流量作为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。
- **Nacos**：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。
- **RocketMQ**：一款开源的分布式消息系统，基于高可用分布式集群技术，提供低延时的、高可靠的消息发布与订阅服务。
- **Alibaba Cloud ACM**：一款在分布式架构环境中对应用配置进行集中管理和推送的应用配置中心产品。
- **Alibaba Cloud OSS**: 阿里云对象存储服务（Object Storage Service，简称 OSS），是阿里云提供的海量、安全、低成本、高可靠的云存储服务。您可以在任何应用、任何时间、任何地点存储和访问任意类型的数据。
- **Alibaba Cloud SchedulerX**: 阿里中间件团队开发的一款分布式任务调度产品，提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。



## 五、Spring Cloud Gatewa

### 5.1 什么是 Spring Cloud Gateway

Spring Cloud Gateway 是 Spring 官方基于 Spring 5.0，Spring Boot 2.0 和 Project Reactor 等技术开发的网关，Spring Cloud Gateway 旨在为微服务架构提供一种简单而有效的统一的 API 路由管理方式。**Spring Cloud Gateway 作为 Spring Cloud 生态系中的网关，目标是替代 Netflix ZUUL**，其不仅提供统一的路由方式，并且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/埋点，和限流等。

![img](19-Spring-Cloud-Alibaba-全家桶整合项目.assets/3f25fcd95769a54eb391931449d5298f.jpg)

> Spring Cloud Gateway 底层基于 Webflux，而不是 SpringMVC。而 Webflux 是基于 Netty，可以实现异步非阻塞的功能。

### 5.2 Spring Cloud Gateway 功能特征

- 基于 Spring Framework 5，Project Reactor 和 Spring Boot 2.0；
- 动态路由；
- Predicates 和 Filters 作用于特定路由；
- 继承 Hystrix 断路器；
- 集成 Spring Cloud DiscoveryClient
- 易于编写 Predicates 和 Filters；
- 限流；
- 路径重写；

### 5.3 Spring Cloud Gateway 工作流程

![img](19-Spring-Cloud-Alibaba-全家桶整合项目.assets/22e4eccf2cbe09332678c04b8de98ebe.jpg)

客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。Handle 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。

过滤器之间使用虚线分开是因为：过滤器可能会发送代理请求之前（`pre`）或之后（`post`）执行业务逻辑。

