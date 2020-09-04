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