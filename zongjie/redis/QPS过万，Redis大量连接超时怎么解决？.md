## QPS过万，Redis大量连接超时怎么解决？

[程序员小灰](javascript:void(0);) *4月9日*

以下文章来源于艾小仙 ，作者艾小仙

[![艾小仙](http://wx.qlogo.cn/mmhead/Q3auHgzwzM6urlFIKVroicBlo3mp2RyM0DICW3nnDIm5W844LyWVPoQ/0)**艾小仙**一个愤世嫉俗，脱离低级趣味的人](https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653224986&idx=2&sn=6a0b26304fe208615b2ecd04bf51b0d8&key=b15bee76b5b937f97b0ba0b4a275a7392cd0a51b63958100c595bad9a8f67d9cd0bcf42dbfc888361974ec4e896ea6b1c6594ad659a659a3908b8082c1e05514e05794a70d71d8f3bb81e8301211d18602afee889d49b46f5997908dc918fe77bf001850af2c612a020426b3341e16e7fe1bf1be2dba22c1302b32baf293fe1d&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AbC%2FjI0DNgiLQMYjIwCBUc4%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2#)

之前负责的一个服务总是在高峰时刻和压测发生大量的redis连接超时的异常`redis.clients.jedis.exceptions.JedisConnectionException`，根据原有的业务规则，首先会从数据库查询，然后缓存到redis中，超时时间设置为3分钟。

并且由于业务的特性，本身未做降级、限流等处理措施，而在巅峰的QPS基本上快达到20000的样子，虽然这个现象只是单纯的一个异常，并不会导致整个主链路的流程不可用，但是我们还是要找出问题的原因并且解决。

首先我们找到负责Redis同学排查，他们告诉我Redis现在很稳定，没有问题，以前现在未来都不会出问题，出了问题肯定是你们自己的问题。

... ...

说的好有道理，我竟无力反驳，真是让人两个头一个大。

这样的话，我们就只能去找找自身的原因了。

# 排查思路

### 查看异常分布

首先根据经验，我们看看自己的服务器的情况，看下异常到底出现在哪些机器，通过监控切换到单机维度，看看异常是否均匀分布，如果分布不均匀，只是少量的host特别高，基本可以定位到出现问题的机器。

诶，这就很舒服了，这一下子就找到了问题，只有几台机器异常非常高。

*不过不能这样，我们继续说排查思路......*

### Redis情况

再次按照经验，虽然负责redis的同学说redis贼稳定巴拉巴拉，但是我们本着怀疑的态度，不能太相信他们说的话，这点很重要，特别是工作中，同学们，不要别人说啥你就信啥，要本着柯南的精神，发生命案的时候每个人都是犯罪嫌疑人，当然你要排除自己，坚定不移的相信这肯定不是我的锅！

好了，我们看看redis集群是否有节点负载过高，比如以常规经验看来的80%可以作为一个临界值。

如果有一个或少量节点超过，则说明可能存在**热key**问题，如果大部分节点都超过，则说明存在**redis整体压力大**问题。

另外可以看看是否有慢请求的情况，如果有慢请求，并且时间发生问题的时间匹配，那么可能是存在**大key**的问题。

嗯... ...

redis同学说的没错，redis稳如老狗。

### CPU

我们假设自己还是很无助，还是没发现问题在哪儿，别急，接着找找别人的原因，看看CPU咋样，可能运维偷偷滴给我们把机器配置给整差了。

我们看看CPU使用率多高，是不是超过80%了，还是根据经验，我们之前的服务一般高峰能达到60%就不错了。

再看看CPU是不是存在限流，或者存在密集的限流、长时间的限流。

如果存在这些现象，应该就是运维的锅，给我们**机器资源不够啊**。

### GC停顿

得嘞，运维这次没作死。

再看看GC咋样。

频繁的GC、GC耗时过长都会让线程无法及时读取redis响应。

这个数字怎么判断呢？

通常，我们可以这样计算，再次按照我们一塌糊涂的经验，**每分钟GC总时长/60s/每分钟GC个数**，如果达到ms级了，对redis读写延迟的影响就会很明显。

为了稳一手，我们也要对比下和历史监控级别是否差不多一致。

好了，打扰了，我们继续。

### 网络

网络这块我们主要看**TCP重传率**，这个基本在大点的公司都有这块监控。

> TCP重传率=单位时间内TCP重传包数量/TCP发包总数

我们可以把TCP重传率视为网络质量和服务器稳定性的一个只要衡量指标。

还是根据我们的经验，这个TCP重传率越低越好，越低代表我们的网络越好，如果TCP重传率保持在0.02%（以自己的实际情况为准）以上，或者突增，就可以怀疑是不是**网络问题**了。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

比如这张图一样，要是和心电图一样，基本上网络问题就没跑了。

### 容器宿主机

有一些机器有可能是虚拟机，CPU的监控指标可能不准确，特别是对于IO密集型的情况会有较大差异。可以通用其他手段来查询宿主机的情况。

# 最后

根据一系列的骚操作，我们根据定位到的机器然后排查了一堆情况，最终定位到是**网络问题**，有单独的几台机器在高峰时期TCP重传率贼高，最后根据运维提供的解决方案：【重启有问题的机器】，我们很顺利的就解决了这个问题。

![程序员小灰](http://mmbiz.qpic.cn/mmbiz_png/NtO5sialJZGrOaned81pMkwib5Voibzes9ibatWlia3ZiceXRbsEWCZbyeOdoQTP1b4licNGR2qbzfaicvstXFztqQJ0wg/0?wx_fmt=png)

**程序员小灰**

一群喜爱编程技术和算法的小仓鼠。

372篇原创内容



公众号

阅读 6882

分享收藏

赞18在看10

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/IbdMVz3ckESSSIv8AazbcUVcoKfwjL9U2TToLJ1mLZ4DcSn4hDNVibhcmAdVpUtKIElynEjPuybS6kwJZA2kT9pl8GRnUvVlH/96)

  NULL

  27

  

  redis同学说的没错，redis稳如老狗。 😃

- ![img](http://wx.qlogo.cn/mmopen/ajNVdqHZLLDCFd8QYteYLjHMFQW4ckwzt4Usy9rN4X94Nh5FKVGuL6YU6vYe1q7pTTiczvn4ueoz9caGBe2REJw/96)

  tree

  14

  

  重启能恢复正常，应该是关机将所有tcp链接都断开了，但这样开机一段时间后，链接数上来应该还是有这个问题吧

- ![img](http://wx.qlogo.cn/mmopen/9ojSnBX2WWZoC5bJZetzyGAStlrqhwTejbKwBxyY0fCia1ZAoiaZAyRKplWcbAibZuEIqWy2w2ND7VRibrv6OdTdgssscwWXQXia9/96)

  ꧁꫞꯭锡纸烫꯭꫞꧂

  13

  

  重启解决一切bug，不重启一切皆bug

- ![img](http://wx.qlogo.cn/mmopen/uSanqRFLmDFsTVYoBVFPvoVfNwNZm6YQOfkfRuSJM0Q9YiclP1uL0nfEWrIqyREcjTSJhJjmzpu6K2icia9zjhgpY0Q1gAe2gsE/96)

  Max_Qiu

  10

  

  IT：没有什么问题是重启不能解决的，如果有，就重装系统

- ![img](http://wx.qlogo.cn/mmopen/CUNOxJmRnOfxCZvFBlN1WeFdNrbq8YwKKFGSHMBFwcZ00fQTQ6IVdJ5WNnkGpydbckbg4LjBB2e5x2Oj4hN88yKPVtJknr68/96)

  念.似水流年

  3

  

  没有什么问题是重启不能解决的，如果有，那就重启两次。

- ![img](http://wx.qlogo.cn/mmopen/AbruuZ3ILCnRI5TUMk3bXWflTA0ApfFRNvRzibzwvjLEv78KfWQnfdPPiayTXGOICOBLLEq8h6oorNgUS77dKOcMO0siadicXrNw/96)

  微信

  3

  

  重启大法

- ![img](http://wx.qlogo.cn/mmopen/IbdMVz3ckETb3tO2KuGy1BUMkfaZSIm9pmiaEgU1eicA5gmGEUyhicZx7k9HiawgceklicibZa3EH1Ls8LkgcCzFZ1O4zIDrvVBOtu/96)

  文

  2

  

  吓我一跳，原来是网管的锅，没重启

- ![img](http://wx.qlogo.cn/mmopen/ONuibvBotppXuhW0Xic8f6ynBAB5GDlgAlibL6FJwrehLU99aXpVibnJicsOkZrYopmALZ6w6dJAlaA0QOLXsnKLNZHxzFSBxWZSr/96)

  洛水

  1

  

  坚信不是我的锅！

- ![img](http://wx.qlogo.cn/mmopen/Q3auHgzwzM5DUnFyK1JLKN83DZXUGWkDA0s5PYjJbpkkG4XxlelTbySLsOvzH78HichmLpSmADfCgSCbWmR3UdSaI2a1XQaDCicGZOcEwGN98/96)

  木一番

  1

  

  重启解决99%的问题

- ![img](http://wx.qlogo.cn/mmopen/9ojSnBX2WWaD8xGwHwhR9Bt6c7XYtQ3RZZqkOqJfnZ2wRrCz0Lh7pgtbWiaalRpZhG6ibAjRcfMH6yIsKgnb3O2PsFMXy0PF8w/96)

  Ben

  

  重启升级换设备

- ![img](http://wx.qlogo.cn/mmopen/3MPDlsvxNY0ibNHFrSBgCiaOpoTWZb0QZuKgM6wwbYFrzoTBJGltOJI0VjfmiaZad41JPphE7ialMQVHxmmsGkdibrxIPvpQNxctL/96)

  zharker

  

  你这没有解决问题吧 你只是解决了有问题的机器

- ![img](http://wx.qlogo.cn/mmopen/9ojSnBX2WWYBIAFL98suHJYYtUjFAvOGTcdhoaPGMgArkNq8V2K9uCH5WLdZW35Z6VjLicibYrLXsicFEibMulOUU2x8em2fCiaCT/96)

  文杰

  

  我还以为redis会有啥问题。

- ![img](http://wx.qlogo.cn/mmopen/ONuibvBotppXuhW0Xic8f6yq4tUXib7X97FDZQqVZdibjibAOVQEbUV91LbQ80g88N4r7zv9dpQs1Lvmib2jcGNrlA3ndW9KXRC7cJ/96)

  LHJ

  

  重启大法

- ![img](http://wx.qlogo.cn/mmopen/PiajxSqBRaELMmxdA9BDhpYN5t9a5fQXq1BKeUhEfCUkZFw6RKMvPNkWJ8e0Via4d89n2dwzKhQAKpd72vpx8D4w/96)

  望穿天堂

  

  2万qps对redis不算啥