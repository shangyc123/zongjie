## 带工作流的springboot后台管理项目，一个企业级快速开发解决方案

点击关注 👉 [JAVA技术](javascript:void(0);) *3月14日*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5Zvn1hEISlelqzC1V9eH01SEcQnJSTJgU7271aBtfoZNQsVbE8JCqX3g8cpunJyoCm1fuKq2eKsXuN38KZBib5A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

作者 | 卓源软件

整理 | 我是程序汪

一直以来，总有小伙伴问说：有没有什么好的项目推荐啊，想参考使用。

一般用途无非如下几种情况：

- 自学练手：从书本和博客的理论学习，过渡到实践练手
- 吸收项目经验，找工作写简历时能参考：毕竟有时候确实没有实际项目经验可写，那研究开源项目的经验就非常宝贵了
- 毕业设计：想找点参考、找点选题、找点灵感
- 甚至还有想接私活参考的：想找一个脚手架快速开发

后台管理类项目也是企业中最常见的实战项目类，掌握它是最基本的



------

# 后台管理类项目

**项目名称：** **JeeSite**

**项目介绍：**

这是个典型的SSM后台管理项目（不是有很多小伙伴让推荐SSM项目练手嘛），基于经典技术组合（Spring MVC、Shiro、MyBatis、Bootstrap UI等）开发，适合学习练手。

而且它作为一个典型的后台管理系统，要素基本都有，包括：组织机构、角色用户、权限授权、数据权限、内容管理、工作流等。

尤其要提的就是最后的**工作流模块**，它可以实现提工单、审核/审批等流程，这个在后台管理类项目里是必备的模块。

## 技术选型

- 主框架：Spring Boot 2.2、Spring Framework 5.2、Apache Shiro 1.6、J2Cache
- 持久层：Apache MyBatis 3.5、Hibernate Validator 6.0、Alibaba Druid 1.1
- 视图层：Spring MVC 5.2、Beetl 3.1（替换JSP）、Bootstrap 3.3、AdminLTE 2.4
- 前端组件：jQuery 3.4、jqGrid 4.7、layer 3.1、zTree 3.5、jquery validation
- 工作流引擎：Flowable 6.5、符合 BPMN 规范、在线流程设计器、中国式工作流
- 技术选型详情：http://jeesite.com/docs/technology/

##  

## 平台介绍

JeeSite 快速开发平台，不仅仅是一个后台开发框架，它是一个企业级快速开发解决方案，基于经典技术组合（Spring Boot、Spring MVC、Apache Shiro、MyBatis、Beetl、Bootstrap、AdminLTE）采用经典开发模式，让初学者能够更快的入门并投入到团队开发中去。在线代码生成功能，包括模块如：组织机构、角色用户、菜单及按钮授权、数据权限、系统参数、内容管理、工作流等。采用松耦合设计，模块增减便捷；界面无刷新，一键换肤；众多账号安全设置，密码策略；文件在线预览；消息推送；多元化第三方登录；在线定时任务配置；支持集群，支持SAAS；支持多数据源；支持读写分离、分库分表；支持微服务应用。

JeeSite 快速开发平台的主要目的是能够让初级的研发人员快速的开发出复杂的业务功能（经典架构会的人多），让开发者注重专注业务，其余有平台来封装技术细节，降低技术难度，从而节省人力成本，缩短项目周期，提高软件安全质量。

JeeSite 自 2013 年发布以来已被广大爱好者用到了企业、政府、医疗、金融、互联网等各个领域中，JeeSite 架构精良、易于扩展、大众思维的设计模式、工匠精神打磨每一个细节，深入开发者的内心，并荣获开源中国《最受欢迎中国开源软件》奖杯，期间也帮助了不少刚毕业的大学生，教师作为入门教材，快速的去实践。

JeeSite4 的升级，作者结合了多年总结和经验，以及各方面的应用案例，对架构完成了一次全部重构，也纳入很多新的思想。不管是从开发者模式、底层架构、逻辑处理还是到用户界面，用户交互体验上都有很大的进步，在不忘学习成本、提高开发效率的情况下，安全方面也做和很多工作，包括：身份认证、密码策略、安全审计、日志收集等众多安全选项供你选择。努力为大中小微企业打造全方位企业级快速开发解决方案。

##  

## 平台优势

JeeSite 整体架构清晰、稳定技术先进、源代码书写规范、经典技术会的人多、易于维护、易于扩展、安全稳定。

JeeSite 功能全，JeeSite 的知识点非常多，也非常少。因为她使用的都是一些通用的技术，通俗的设计风格，大多数基础知识点多数人都能掌握，所以每一个 JeeSite 的功能点都非常容易掌握。只要你学会使用这些功能和组件的应用，就可以顺利的完成系统开发了。

JeeSite 是一个低代码开发平台，具有较高的封装度、扩展性，封装不是限制你去做一些事情，而是在便捷的同时，也具有较好的扩展性，在不具备一些功能的情况下，JeeSite 提供了扩展接口，提供了原生调用方法。

大家都在用 Spring ，在学习 Spring 架构的优点，Spring 提供了较好的扩展性，可又有多少人去修改它的源代码呢，退一步说，大家去修改了 Spring 的源码，反而会对未来升级造成很大困扰，您说不是呢？这样的例子很多，所以不要纠结，JeeSite 也一样具备强大的扩展性。

发展至今 JeeSite 平台架构已经非常稳定，JeeSite 是一个专业的平台，是一个让你使用放心的平台。



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 源码地址

```
源码地址获取： 
扫描下方公众号回复 639
```

阅读 2174

分享收藏

赞4在看2