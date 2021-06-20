## 手把手教你在本地搭建 Nacos 服务注册和配置管理中心

原创 鸭血粉丝 [Java极客技术](javascript:void(0);) *4月9日*

收录于话题

\#nacos

1个

每天早上七点三十，准时推送干货



![Java极客技术](http://mmbiz.qpic.cn/mmbiz_png/tWOhQMr1wdDwibGUW8HOfmZfVuVryhfO8P8R3vJGrHBmHybX2F0GgHUfL4O9ibP4pYKPNTKQW8um3D6bnqibjLOsA/0?wx_fmt=png)

**Java极客技术**

Java 极客技术由一群热爱 Java 的技术人组建，专业输出高质量原创的 Java 系列文章，优秀的 Java 程序员都在这里。

606篇原创内容



公众号

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tWOhQMr1wdB1YeGzVicuM4pnpyXha1aayNoktaxNxOVHf5zyKia6icMQ7icUOL6aGWibrVML8f17NmYbiamNV2lEPlicA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Hello 大家好，我是阿粉，现如今微服务早就成为每个互联网公司必备的架构，在微服务中服务的注册发现和统一配置中心是不可或缺的重要角色。现有的服务注册中心主要有 `ZooKeeper`、`Eureka`、`Consul` ，`Etcd`、`Nacos`，而统一配置中心主要有 `Apollo`，`Spring Cloud Config`，`Disconf`，`Nacos` 可以看到服务注册中心和统一配置管理有很多，而且每一种都是网上比较流行的，使用人数也偏多，但是我们可以发现只有 `Nacos`是集两者于一身的。这篇文章主要给大家介绍一下 `Naocs` 的基本信息以及安装步骤，后面的文章再带大家更深入的了解一下`Nacos`。

## Nacos 是什么

打开网站 Nacos 的首页，我们可以看到下面的内容，写到 Nacos 是一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

> Nacos 致力于帮助我们发现、配置和管理微服务，提供了一组简单易用的特性集，帮助我们快速实现动态服务发现、服务配置、服务元数据及流量管理。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tWOhQMr1wdB1YeGzVicuM4pnpyXha1aayMq8JUBgibelk3P9AtCVFk7mVibrNB89kBIGNBe2YqeZNqrFOP76TwGrQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## Nacos 本地搭建 

Nacos 的安装可以通过安装包或者源码来时候，这里我们为了以后方便调试，所以选择采用源码来安装。

1. 第一步在 GitHub 上面下载源码，这里可以使用 HTTP 协议也可以使用 SSH，阿粉这里为了简单就直接采用 HTTP 协议来下载了：`git clone https://github.com/alibaba/nacos.git`；
2. 下载完成过后，我们使用 IDEA 的 File》Open 打开项目，如下图
3. ![图片](https://mmbiz.qpic.cn/mmbiz_png/tWOhQMr1wdB1YeGzVicuM4pnpyXha1aayFSqTab0XuWk2LTfYChkJ4Yg0h2uAfmCKvExffgmNdK6yWulKuShwxA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
4. 刚打开后我们会看到有很多个模块，从模块的命名我们大概可以猜到有些模块是干嘛的，比如有 `auth` 权限模块，`config` 配置模块等这篇文章我们主要看安装和使用，后面慢慢深入了解具体的细节。
5. ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
6. 拉取下来过后，我们需要创建数据库，我们找到`console` 模块下的数据库脚本和配置文件，如下图，在 `MySQL` 数据库中创建一个名为`nacos` 的数据库，然后执行`schema.sql` 脚本，创建相应的表结构，同时在 `application.properties` 文件中找到对应配置数据源的地方，配置好账号密码。这里对于 `MySQL` 的数据库的搭建和创建不在这篇文章的讨论范围里面就不提了。
7. ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
8. 配置好数据库过后，在控制台执行 `mvn` 命令`mvn -Prelease-nacos -Dmaven.test.skip=true -Drat.skip=true clean install -U`，编译一下，然后执行 `console` 模块下面的`Nacos.java` 中的 `main` 方法，启动后我们会看到下图，显示的是集群模式启动，因为我们是本地单机启动，所以可以增加一个 `JVM` 参数`-Dnacos.standalone=true` 来设置单机模式。按照下图设置好过后，重新启动即可。
9. ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
10. ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)
11. 启动成功过后，我们可以访问地址`http://127.0.0.1:8848/nacos`，我们可以看到如下页面，包含配置管理、服务管理、以及权限，命名空间和集群的管理。阿粉这里搭建的是最新的版本，旧版本是没有权限管理的。我们可以看到整个页面非常的简洁，其中配置管理服务管理基本上覆盖了我们常用的所有功能。
12. ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 核心角色

上面我们已经搭建好了 `Nacos` 平台，现在给大家介绍一下，`Nacos` 的几个核心角色。

1. 命名空间`Namespace` ：用于进行租户粒度的配置隔离，不同的命名空间下，可以存在相同的 `Group` 或`Data ID` 的配置。`Namespace` 的常用场景之一是不同环境的配置的区分隔离，例如开发测试环境和生产环境的资源（如配置、服务）隔离等。
2. 分组`group`：每个命名空间下还可以进行分组，相同集群的服务可以设置在同一个分组里面，如果没有手动指定默认分组为`DEFAULT_GROUP`
3. 配置集 `Data ID`：用于唯一区分不同配置文件，可以根据配置的信息不同，分别设置，比如可以单独配置 `Redis` 的配置文件，以及 `MySQL` 的配置文件，可以尽量细分复用，这样在有调整的时候只需要改一处即可。
4. 更详细的名词解释可以参考官网https://nacos.io/zh-cn/docs/concepts.html

## 使用

上面介绍了 Nacos 的本地搭建，搭建好了过后我们就可以使用了，因为 Nacos 不仅支持服务发现也支持统一配置管理，这篇文章我们先创建两个命名空间 `test1` 和 `test2`，然后分别添加两个配置文件`com.example.properties` 和`com.example2.properties`，创建的步骤比较简单。首先点击命名空间菜单，然后在右上角新增命名空间，接着输入相关信息就好了，这里新版本的 `Nacos` 支持手动填入命名空间的 ID，旧版本是不支持的，会随机生成一个字符串，这里建议手动填入具有含义的ID。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

创建完命名空间过后，就可以在此命名空间下创建相应的配置文件了，这篇文章主要给大家介绍一下 Nacos 的本地部署过程以及简单的系统使用，后面的文章结合代码来实现一下服务的注册发现和配置的动态更新，敬请期待。



## 最后说两句（求关注）

**最近大家应该发现微信公众号信息流改版了吧，再也不是按照时间顺序展示了。这就对阿粉这样的坚持的原创小号主，可以说非常打击，阅读量直线下降，正反馈持续减弱。**

所以看完文章，哥哥姐姐们给阿粉来个**在看**吧，让阿粉拥有更加大的动力，写出更好的文章，拒绝白嫖，来点正反馈呗~。

如果想在第一时间收到阿粉的文章，不被公号的信息流影响，那么可以给Java极客技术设为一个**星标**。

最后感谢各位的阅读，才疏学浅，难免存在纰漏，如果你发现错误的地方，留言告诉阿粉，阿粉这么宠你们，肯定会改的~

最后谢谢大家支持~

最最后，重要的事再说一篇~

快来关注我呀~
快来关注我呀~
快来关注我呀~

**【号外】Java 极客作者团队招新啦！！！扫码添加鸭血粉丝微信，加入我们，一起进步。**

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![Java极客技术](http://mmbiz.qpic.cn/mmbiz_png/tWOhQMr1wdDwibGUW8HOfmZfVuVryhfO8P8R3vJGrHBmHybX2F0GgHUfL4O9ibP4pYKPNTKQW8um3D6bnqibjLOsA/0?wx_fmt=png)

**Java极客技术**

Java 极客技术由一群热爱 Java 的技术人组建，专业输出高质量原创的 Java 系列文章，优秀的 Java 程序员都在这里。

606篇原创内容



公众号

往期精彩回顾









[一点点的给你分析RabbiMQ的几种消息模型【文末有惊喜】](http://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247494914&idx=1&sn=7595f15d3916e51ac03b01c522c87a30&chksm=c28682c3f5f10bd50d01653cbcca7663f714b5c54521fecb1d17cf46f8140b255ecf574fd48e&scene=21#wechat_redirect)

[Spring Boot 学习笔记，这个太全了！](http://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247494524&idx=1&sn=20023574f0a53a5a6851ba3423380716&chksm=c28684bdf5f10dab3d003ede539f55a8b77eb6c4114dc9b8dad145e4db44705e1975d4da5b0f&scene=21#wechat_redirect)







![img](https://mmbiz.qlogo.cn/mmbiz_jpg/fPADUR9p6acau4uticGBhaiaeH6q8T7g9Zw7BHRDpx9B7BdKKgWZvWOyJRWL0fTZyfJwiaAH4pFKtlb7Cv8vovcCA/0?wx_fmt=jpeg)

鸭血粉丝

原创不易，众筹一碗鸭血粉丝汤

![赞赏二维码](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247494944&idx=1&sn=5f853d42120a224ac17a563328d6f11d&key=bf0a475117e7a82ee023ccca59b8316fd5a8a2f4a8949fa0d3028591644ef0a7075011378bf3505edb1797d6b01fa601cb070906eaeb480d78f774886c6e1a310eb7c2ef7f004a2959795ae0d809b8d912574c20bd4f4bfa39e521548711a9e1f9eef384cd554c4df07f1857d19e8453593b8c409ce57f2e1033a94ff9c9bafc&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=Ad9gMm5V7eN5dcLpIzYOMZE%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2)[喜欢作者](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247494944&idx=1&sn=5f853d42120a224ac17a563328d6f11d&key=bf0a475117e7a82ee023ccca59b8316fd5a8a2f4a8949fa0d3028591644ef0a7075011378bf3505edb1797d6b01fa601cb070906eaeb480d78f774886c6e1a310eb7c2ef7f004a2959795ae0d809b8d912574c20bd4f4bfa39e521548711a9e1f9eef384cd554c4df07f1857d19e8453593b8c409ce57f2e1033a94ff9c9bafc&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=Ad9gMm5V7eN5dcLpIzYOMZE%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2##)

[1](javascript:;) 人喜欢

![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM50icmo8BicWIuoGTicibcR6UxNHoIXtkJ6uVOOxGicN8MqH0Q/132)

阅读 1347

分享收藏

赞17在看9

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/HsUHg9qicbJVgKSZoCzlfuRSva0VKV6aak97wE6IvQ1UYEJTy4CeUicE4UGn1VXx2EtNO6iakXBfaCxjz4hQsSVaEsqxD65KRBz/96)

  BigBigWolf

  

  Consul也是可以用作配置中心的呀

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)

  

  可以的哦