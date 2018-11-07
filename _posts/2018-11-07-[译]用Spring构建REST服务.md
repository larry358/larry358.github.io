---
title: 用Spring构建REST服务
categories:
- General
excerpt: |
  10月24日现在被称作程序员节，为了纪念这个节日，本博客正式启用，希望以后能不断更新。（当然也存在断更的可能，但愿不会如此。。。）
feature_text: |
  REST已经迅速成为构建web服务的事实标准，因为这些服务易于构建和消化。
feature_image: "https://picsum.photos/2560/600?image=733"
image: "https://picsum.photos/2560/600?image=733"
---

原文链接：[Building REST services with Spring](http://spring.io/guides/tutorials/rest/)

# 用Spring构建REST服务

&nbsp;&nbsp;&nbsp;&nbsp;REST已经迅速成为构建web服务的事实标准，因为这些服务易于构建和消化。

&nbsp;&nbsp;&nbsp;&nbsp;对于REST如何适用于这个微服务的世界有更加广泛的讨论，但是————在这个教程里————我们仅仅关注于构建RESTful的服务。

&nbsp;&nbsp;&nbsp;&nbsp;为什么要用REST？REST拥抱了Web的诸多规程，包括架构，益处和其余各个方面。由于它的作者罗伊·菲尔丁(Roy Fielding)参与到了许多管理Web运作的规范当中，这点并不令人感到惊讶。

&nbsp;&nbsp;&nbsp;&nbsp;有什么好处？Web及其核心协议HTTP，提供了一系列特性：

- 合适的行为(`GET`,`POST`,`PUT`,`DELETE`,...)
- 缓存
- 重定向和转发
- 安全（加密和认证）

&nbsp;&nbsp;&nbsp;&nbsp;这些都是构建可快速恢复的服务中的关键因素。然而这并非全部。Web是由许多小型规范构建的，因此它容易变革，而不至于被“标准战争”所阻碍。

&nbsp;&nbsp;&nbsp;&nbsp;开发者可以调用实现了各种各样的规范的第三方工具包，从而简单且方便地同时获得客户端和服务器的技术。

&nbsp;&nbsp;&nbsp;&nbsp;因此，在HTTP之上构建的RESTful API提供了构建灵活的API的方法，它们能够：

- 提供后端兼容性
- 可演进的API
- 可伸缩的服务
- 能保证安全的服务
- 广泛的服务，从声明式到非声明式

&nbsp;&nbsp;&nbsp;&nbsp;重要的是，应当意识到，尽管无处不在，但REST并非一项标准，*本质上*，它更是一种方法，一种风格，一系列帮助你构建Web级系统的，作用在体系上的*约束*。在这篇教程中，我们将采用Spring产品组合构建一个RESTful服务，来一窥REST不计其数的特性。

## 入门

&nbsp;&nbsp;&nbsp;&nbsp;在本教程的整个过程中，我们将采用[Spring Boot](https://spring.io/projects/spring-boot)。前往[Spring Initializr](https://start.spring.io/)并选择以下内容：

- Web
- JPA
- H2
- Lombok

&nbsp;&nbsp;&nbsp;&nbsp;之后选择"Generate Project"（生成项目）。将会下载一个`.zip`文件。解压缩之。你会在里面找到一个包含`pom.xml`构建文件的，基于Maven的简单项目。（注：你*可以*使用Gradle。本教程中的示例将会基于Maven。）
