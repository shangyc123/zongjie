## 19张图带你梳理SpringCloud体系中的重要技术点！

[程序员闪充宝](javascript:void(0);) *4月13日*

收录于话题

\#SpringCloud1

\#微服务2

## **![图片](https://mmbiz.qpic.cn/mmbiz_png/eukZ9J6BEiaeKZo1HxC99MeZJibns8KalXe20ibibaH585hIsxVMER52C0PWCVbibficBoZF67ianGgvdqlYuN6kuVQkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**

> 作者：三分恶
>
> 来源：www.cnblogs.com/three-fighter/p/13485459.html

# 1、什么是微服务

## 1.1、架构演进

架构的发展历程是从单体式架构，到分布式架构，到SOA架构，再到微服务架构。

图1：架构演进

![图片](https://mmbiz.qpic.cn/mmbiz_png/eukZ9J6BEiaexg5kydYYwcIcicCPUNxdbt8N7p2qRbtHbJicy1ibFqPjjiaQrdDAtDYGOmxmhUQU7REqtiaM4LdaZf4Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)在这里插入图片描述

- 单体架构：未做任何拆分的Java Web程序

图2：单体架构示意图

![图片](https://mmbiz.qpic.cn/mmbiz_png/eukZ9J6BEiaexg5kydYYwcIcicCPUNxdbtxrZMgOL1zovHwJ8GgO445vIloic7VQVckan6tOlMKgNeKBYgk7md6hw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)在这里插入图片描述

- 分布式架构:按照业务垂直划分，每个业务都是单体架构，通过API互相调用。

图3：分布式架构示意图

![图片](https://mmbiz.qpic.cn/mmbiz_png/eukZ9J6BEiaexg5kydYYwcIcicCPUNxdbtRA13R5AkEcOW0KAXic4RqzRHQmpSddRuMib6yILV8ITwtUD0cstjiaCsQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)在这里插入图片描述

- SOA架构：SOA是一种面向服务的架构。其应用程序的不同组件通过网络上的通信协议向其它组件提供服务或消费服务，所以也是分布式架构的一种。

图4：SOA架构示意图

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

## 1.2、微服务架构

微服务架构在某种程度上是SOA架构的进一步的发展。

微服务目前并没有比较官方的定义。微服务 Microservices 之父，马丁.福勒，对微服务大概的概述如下：

> 就目前而言，对于微服务业界并没有一个统一的、标准的定义（While there is no precise definition of this architectural style ) 。但通常在其而言，微服务架构是一种架构模式或者说是一种架构风格，它提倡将单一应用程序划分成一组小的服务，每个服务运行独立的自己的进程中，服务之间互相协调、互相配合，为用户提供最终价值。服务之间采用轻量级的通信机制互相沟通（通常是基于 HTTP 的 RESTful API ) 。每个服务都围绕着具体业务进行构建，并且能够被独立地部署到生产环境、类生产环境等。另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建，可以有一个非常轻量级的集中式管理来协调这些服务。可以使用不同的语言来编写服务，也可以使用不同的数据存储。

图5：微服务定义思维导图

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

图6：微服务架构示意图

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

## 1.3、微服务解决方案

目前最流行的两种微服务解决方案是SpringCloud和Dubbo。

# 2、SpringCloud概览

## 2.1、什么是SpringCloud

Spring Cloud 作为 Java 言的微服务框架，它依赖于 Spring Boot ，有快速开发、持续交付和容易部署等特点。Spring Cloud 的组件非常多，涉及微服务的方方面面，井在开源社区 Spring、Netflix Pivotal 两大公司的推动下越来越完善。

SpringCloud是一系列组件的有机集合。

图7：SpringCloud技术体系

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

图8：SpringCloud技术体系思维导图

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

## 2.1、SpringCloud主要组件

### 2.1.1、Eureka

Netflix Eureka 是由 Netflix 开源的一款基于 REST 的服务发现组件，包括 Eureka Server 及 Eureka Client。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.2、Ribbon

Ribbon Netflix 公司开源的一个负载均衡的组件。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.3、Feign

Feign是是一个声明式的Web Service客户端。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.4、Hystrix

Hystrix是Netstflix 公司开源的一个项目，它提供了熔断器功能，能够阻止分布式系统中出现联动故障。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.5、Zuul

Zuul 是由 Netflix 孵化的一个致力于“网关 “解决方案的开源组件。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.6、Gateway

Spring Cloud Gateway 是 Spring 官方基于 Spring 5.0、 Spring Boot 2.0 和 Project Reactor 等 技术开发的网关， Spring Cloud Gateway 旨在为微服务架构提供简单、 有效且统一的 API 路由 管理方式。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.7、Config

Spring Cloud 中提供了分布式配置中 Spring Cloud Config ，为外部配置提供了客户端和服务器端的支持。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.8、 Bus

使用 Spring Cloud Bus, 可以非常容易地搭建起消息总线。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.9、OAuth2

Sprin Cloud 构建的微服务系统中可以使用 Spring Cloud OAuth2 来保护微服务系统。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

### 2.1.10、Sleuth

Spring Cloud Sleuth是Spring Cloud 个组件，它的主要功能是在分布式系统中提供服务链路追踪的解决方案。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

# 3、总结

本文中对架构的演进及Spring Cloud 构建微服务的基本组件进行了概览。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)在这里插入图片描述

------

博主水平有限，如有错漏，欢迎指出！

```
日志采集系统都用到哪些技术？
一步步安装Jumpserver堡垒机图文详解（官方教程版），从此爽歪歪！
零散的MySQL基础总是记不住？看这一篇如何拯救你相同字符串，但是equals为false？我多年的java白学了吗?
华为面试官：为什么 HashMap 的加载因子是0.75？
趣头条面试题：ThreadLocal是什么？怎么用？为什么用它？有什么缺点
@Autowire和@Resource注解使用的正确姿势，这些年我一直用错了！！
强大：MyBatis 流式查询华为面试官：为什么 HashMap 的加载因子是0.75？

点击阅读全文前往微服务电商教程
```

[阅读原文](https://mp.weixin.qq.com/s?__biz=MzIxMjU5NjEwMA==&mid=2247502857&idx=1&sn=791060a7eb4bf46004f6d89a61041a3c&key=2b0dcaaa74ed1dec90c2391e079d59bea8683fca217a9acd06af412a8fcbbd84596e6b8484a473a162f65694449e5480ca893a64598409e10852f341307fb52bded38f97fa13ff2027f6dfd4111113f4babd3e81667f7dbeaed721be691b29fd992b514f2d78dcb1e2dc5b75cca473aa1a9cd4f8af083feed6cf0febca2deac9&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AfoOS7v7PtoHSVeGPOAgMg8%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2##)

阅读 2956

分享收藏

赞9在看12

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/4gLCqK5MptOkpLJXUU8JcvovnbN2ibD8ZyPZeFRUHw5qcmM6ULiaRbCX1cGia1KiaWyXsmDUiaLh3YRhSxseVsAZicMA5I7Cnibia7b9/96)

  WeiG

  1

  

  花哨的xmind配图居然让我看出来了感觉