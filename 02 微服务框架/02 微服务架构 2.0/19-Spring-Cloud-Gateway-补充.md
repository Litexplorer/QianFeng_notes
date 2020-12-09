---
title: 19-Spring-Cloud-Gateway 补充
time: 2020年12月9日

---

[toc]

## 〇、介绍

该笔记主要记录了 SpringCloudGateway 的常用配置。

视频在线观看地址为：https://www.bilibili.com/video/BV1QZ4y1V7X3?t=57&p=2

## 一、什么是 Spring Cloud Gateway

Spring Cloud Gateway 作为 Spring Cloud 生态系统中的网关，目标是替代 Netflix Zuul。其不仅提供了统一的路由方式，并且还基于 Filter 链方式提供了网关基本的功能。

## 二、常用网关解决方案

### 2.1 Nginx + Lua

Nginx 是由 IgorSysoev 为俄罗斯访问量第二的 Rambler.ru 站点开发的、一个高性能的 HTTP 和方向代理服务器。Nginx 一方面可以做反向代理，另一刚面可以做静态资源服务器。

> - Nginx 适合做门户网关，是作为整个全局的网关，对外的处于最外层的；而 Gateway 属于业务网关，主要用来对应不同的客户端提供服务，用于聚合业务。各个微服务独立部署，职责单一，对外提供服务的时候，需要有一个东西把业务聚合；
> - Gateway 可以实现熔断、重试等功能，而 Nginx 不具备；

### 2.2 Kong

Kong 是 Mashape 提供的一款 API 管理软件，它本身是基于 Nginx + Lua 的，但比 Nginx 提供了更简单的配置方式，数据采用了 ApacheCassandra/PostgreSQL 存储，并且提供了一些优秀的插件，比如验证、日志、调用频次限制等。Kong 非常有人的地方就是提供了大量的插件来扩展应用没通过设置不同的插件可以为服务器提供各种增强的功能。

> 有点：由于基于 Nginx ，所以在性能上和稳定性上表现非常优秀。Kong 作为一款商业软件，在 Nginx 上做了很多扩展工作，而且还有很多付费的商业插件。Kong 本身也有付费的企业版，其中包括了技术支持、使用培训服务以及 API 分析插件。
>
> 缺点：Kong 如何结合目前已有的 Spring Cloud 服务治理体系？

### 2.3 Traefik

Traefik 是一个开源的 GO 语言开发的、为了让微服务部署更加便捷而诞生的现代 HTTP 反向代理、负载均衡的工具。它支持多种后台（Docker、Swarm、Kubernetes、Marathon、Mesos、Consul、Etcd、Zookeeper 等）来自动化、动态地应用它的配置文件。Traefik 拥有一个基于 AngularJS 编写的简单网站界面，支持 RESTFul API、配置文件热更新，无需重启进程。

> 相比 Spring Cloud Gateway 而言，Traefik 更适合 Kubernetes。



