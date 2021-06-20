## 面经分享 | 2年经验，1个月拿下阿里P6 Offer

[程序员小灰](javascript:void(0);) *5月21日*

以下文章来源于艾小仙 ，作者艾小仙

[![艾小仙](http://wx.qlogo.cn/mmhead/Q3auHgzwzM6urlFIKVroicBlo3mp2RyM0DICW3nnDIm5W844LyWVPoQ/0)**艾小仙**一个愤世嫉俗，脱离低级趣味的人](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653226863&idx=2&sn=555f6eb904c8a8f1b13d636bef3ee300&key=bf0a475117e7a82edd93a81106430f99a9aad7592e23ae0dd57af048b9be96cfb422cd50019d2df3f32e1b3f0049f2b428cbdbfe10c0d0b4a1b774b6bca2652768e159f3b5b17d3caf0c25772ffe1b3c7883fb3e8ef080dc6d0aa831187723162fab9285d2bba74d4c4ea92ef2fd6b94474b134daa9d874f6f14b97f70c0f810&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AV0HAv1q7JS%2F69wT%2Boaku4w%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2#)

这些面试题来自于我的老乡读者分享，很厉害，2年经验，面试几个月拿下了N个Offer，包括滴滴、有赞和阿里这些一二线公司。

> 内容完全来自读者自己，引用部分为读者自身回答描述，感谢分享。阿里的在最后部分。

# 酷乐家

1. redis集群使用，宕机怎么处理
2. mysql分库分表，数据怎么做分片
3. mysql索引了解和优化经历
4. Zk数据同步，崩溃恢复
5. docker k8s常用打包命令
6. docker和虚拟机的区别
7. volatile和synchronize关键字
8. hashMap特性
9. Zk锁之间的节点同步

# 兑吧科技

### 一面

1. 业务中redis集群的使用

2. mysql索引，explain命令的结果

3. 场景问题：生产日志每天打10个G，怎么防止日志打满磁盘。

4. 场景问题：设计不同用户角色的菜单和功能权限。

5. 本地内存的了解。场景题：怎么实现更新所有集群中的机器中的本地内存。

   > 我回答了rocketMQ的广播模式

### 二面

1. 场景问题：大量的http请求怎么识别客户的身份

   > 根据IP和用户行为做用户标签。
   >
   > 和电信移动等营业商合作，根据手机号码直接获取用户信息

2. 日志区别生产和线上

   > 标志识别（时间戳或者ip）

3. reactor模型的了解，它和面向结构编程的本质区别

4. JVM的调优经历和垃圾回收

5. synchronized锁和concurrent包下面的常见工具类的了解。

6. 场景问题：怎么设计一个高并发的应用，从单体演变为分布式系统的过程。

# 同花顺

### 一面

1. 项目经验
2. redis数据分片机制
3. 看的书籍，介绍印象深刻的
4. 用list手写一个线程安全的队列。

### 二面

1. 介绍一下项目亮点，痛点和解决的过程。

2. 怎么解决分布式事务，用rocketmq是否存在消息丢失的场合，怎么做补偿

3. 业务中数据模型，设计的时候怎么抽象表结构的。

4. 分布式锁对扣减库存的性能影响，并发量大用乐观锁将压力转移到mysql是否合理

5. 日常怎么处理微服务之间的链路追踪，traceId和大数据日志采集的实现原理

6. redis的缓存穿透、击穿、雪崩怎么处理，布隆过滤器

7. redis单线程效率高的原因

   > redis多路复用

8. netty的reactor模型

9. 限流策略，sentinel的了解

10. 在工作中的不足，怎么弥补？

# 顺丰科技

### 一面

1. 项目复杂度的体现和用到的设计模式，下游服务的认知，订单系统拆单逻辑。

2. 分布式锁，查库存和扣减库存原子性

3. 分布式事务，订单和库存的事务一致性

4. 多个系统之间数据同步， 发布订阅，初始化数据，增量binglog同步。

5. 增量数据除了binglog怎么同步？

   > 通过version版本号，每次主动拉取更新数据查询version比自己大的数据，进行更新。

6. Spring IOC，AOP

7. AOP是在什么时候加载的，容器怎么加载bean

8. Spring三级缓存怎么解决循环依赖

9. mybatis拦截器动态代理底层怎么找到XML，怎么执行sql

10. 快递订单中，有收件人和寄件人，怎么查询某个用户的快递订单(这个用户有可能是收件人也可能是寄件人)

    > 查询：where 收件人=xx or 寄件人 =xx
    >
    > 用union 或者union all查询，建立组合索引

11. 敏捷开发的了解

12. 最近看什么书，辞职理由

### 二面

1. 项目架构的描述、数据库表结构的设计

2. 网关层怎么做限流

   > 漏斗、令牌桶、滑动窗口(sentinel)

3. RMQ保证事务一致性

4. redis集群

5. 算法：有一棵树，每个节点可能有多个子节点，描述这种数据结构并且遍历

   > 第一种思路：子节点属性用集合描述：Listchilden;递归遍历
   >
   > 第二种思路：当作变形的二叉树，左子树是一个单一节点的部分，右子树是一颗大树，遍历也要递归，复杂度很高。

# 有赞

### 一面

1. 项目经验
2. 垃圾回收算法，对CMS的了解
3. 初始标记和并发标记的区别
4. Spring相关
5. redis  MySQL  RMQ
6. 分库分表的运用
7. JVM锁
8. hashmap1.7和1.8的特效，1.8是否线程安全
9. chm
10. epoll poll select的区别 文件描述符

### 二面

1. 业务场景
2. redis分布式锁
3. 分布式事务
4. rocketmq

# 乌鸫科技

### 一面

1. sentinel结合dubbo做限流
2. 怎么实现单例，怎么破坏单例

### 二面

1. 线程池
2. 类加载机制，分别加载了哪些对象
3. tomcat加载了哪个类加载器
4. jvm内存模型
5. 垃圾回收算法，CMS，三色标记法
6. JVM调优
7. 分布式ID生成算法
8. 秒杀场景设计，解决超卖的设计
9. MySQL索引设计，`select * from table where a=1,b= 1 and c>2 and d=3 order by e`怎么索引建立
10. redis分布式锁怎么实现

# 阿里集团

### 一面

1. 分布式锁的多种设计
2. mysql分库分表
3. 秒杀的设计
4. DDD领域设计
5. 分布式事务常见的解决方案
6. redis集群
7. hashMap、chm
8. 事务的特性

### 二面

1. 业务场景50分钟，深挖业务的设计，数据库模型的设计，设计的缺陷和替代的设计方案。
2. mysql索引的优化、隔离级别、mvcc、redo和binglog写入时机，同步机制。
3. JVM优化经历
4. rocketmq的深入
5. volatile和synchronize
6. dubbo的流程，对称加密和非对称加密、spi、负载均衡算法
7. 算法leetcode随机抽一题，不需要运算通过，基本不卡人

### 三面

纯业务的理解和行业领域的认识50分钟

### 四面

1. 项目的价值，在工作中的职责，做了哪些事情

2. 开源框架的了解，netty的了解，怎么实现一个聊天客户端

3. 设计一个rpc框架，dubbo怎么用线程实现同步调用，具体需要几个线程

   > 答不出

4. 看的书籍，收获、个人职业规划等

# 总结

> 2年的话感觉问的东西不是很难，把你那些我要进大厂的东西看一看，是一个知识概览。然后大厂对于每个知识会问的比较深入，看的话要平时多看书，也可以看看极客时间的各个文章。
>
> 项目经验很重要，一般在各个中台的核心业务，例如交易 商品 营销 支付，做过这些东西并且能总结出项目的内容和难点大厂一般都要。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![程序员小灰](http://mmbiz.qpic.cn/mmbiz_png/NtO5sialJZGrOaned81pMkwib5Voibzes9ibatWlia3ZiceXRbsEWCZbyeOdoQTP1b4licNGR2qbzfaicvstXFztqQJ0wg/0?wx_fmt=png)

**程序员小灰**

一群喜爱编程技术和算法的小仓鼠。

372篇原创内容



公众号

阅读 3514

分享收藏

赞3在看1

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/ONuibvBotppXdZAvKCf2fFic95r3BcdpTcrkhSaicUmKjybjd2p9DFs7daM60SUyjmsvXrgo2ciazOsev5q0iazyibGHtshJBqQJI5/96)

  林光光

  2

  

  我一年半进的p6

- ![img](http://wx.qlogo.cn/mmopen/AbruuZ3ILClUdl756Smz5CUwkkOOK4xTlEvImlmBYOKTkT4iawvQHkGK43qYZU2wOVHUhwd0M3NWtfRyqShXTV8mGxx6tuu78/96)

  黑暗中行走

  

  我亏就亏在没有高并发和复杂系统的项目经验，平时再埋头苦学，也不如实际经历两年。

- ![img](http://wx.qlogo.cn/mmopen/uSanqRFLmDFsTVYoBVFPvqe6mdbCnbgadbgfZ27iaHoiaFYD8Y5DvRH3bRrGScWH6cKFpiblQBPCzpyc1sz38KXZpDt28GxoDHD/96)

  Gladys

  

  有没有测试开发的面经呀？

- ![img](http://wx.qlogo.cn/mmopen/Q3auHgzwzM4JEm7t83LjicbCkBhcQicX8hCtfsuPznjuicLwxdwteK7sZy4gLx3yQLwmSOYaWUb8YeSJc2liaq1skFnb7dLoNe3rX2dXibGiaAwR0/96)

  🇸

  

  应届生，CTO事业群2面挂了，好难受。还是自己水平不够。

- ![img](http://wx.qlogo.cn/mmopen/xgM6gEBXk7JpPrloh2wR4trk3EK9YhOEiad9hib81RXlvvu2JerRPwbruUwkllsU5xFNP89YAZibS5XpN1icjWA6kQY9mwxbu2WZ/96)

  Aluo

  

  牛逼，才两年就进了真优秀啊