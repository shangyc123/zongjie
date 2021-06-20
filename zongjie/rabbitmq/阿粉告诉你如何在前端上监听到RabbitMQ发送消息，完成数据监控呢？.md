## 阿粉告诉你如何在前端上监听到RabbitMQ发送消息，完成数据监控呢？

原创 鸭血粉丝 [Java极客技术](javascript:void(0);) *5月5日*

  

每天早上七点三十，准时推送干货

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tWOhQMr1wdCNesO2CZfm7icCXk2Z2DaAoVExrvFdBWfjtEricQUllkrd9dbmqallriaL5VSytkQeBbiaj4ahTlxibjw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



之前还记得阿粉给大家讲了关于RabbitMQ的经典实用还有整合到SpringBoot项目中的案例么？最近一段时间，阿粉的朋友问我说，公司安排他让他研究一下如何在前端实现对RabbitMQ发送消息的实时监控，而这也涉及到了阿粉的知识盲区，于是阿粉就开始了学习的道路，接下来就跟着阿粉一起来学习一下这关于如何在前端监听到RabbitMQ发送消息，以便实现自己项目中的功能吧。



#### RabbitMQ支持的协议

1. stomp协议

stomp协议即Simple (or Streaming) Text Orientated Messaging Protocol，简单(流)文本定向消息协议，它提供了一个可互操作的连接格式，允许STOMP客户端与任意STOMP消息代理（Broker）进行交互。STOMP协议由于设计简单，易于开发客户端，因此在多种语言和多种平台上得到广泛地应用。

而我们在接下来的文章里面主要讲stomp如何对RabbitMQ实现监听。

stomp协议的前身是TTMP协议（一个简单的基于文本的协议），专为消息中间件设计。

这句话就说出了，专门为了消息中间件设计的，其实他并不是针对RabbitMQ在前端使用的，而是针对整个**消息中间件**的使用。

2.mqtt协议

还有一种经常使用的，就是mqtt协议了，mqtt协议全称（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于发布/订阅（Publish/Subscribe）模式的轻量级通讯协议，该协议构建于TCP/IP协议上，由IBM在1999年发布，目前最新版本为v3.1.1。

mqtt协议是属于在应用层协议的，这样也就是说只要是支持TCP/IP协议栈的地方，都可以使用mqtt.

#### RabbitMQ开通stomp协议

安装RabbitMQ的教程阿粉就不再给大家讲了，毕竟百度上有很多文章来告诉大家如何去安装RabbitMQ，不管是Linux还是Windows的，大家只要注意的一点就是，首先先安装erlang 语言支持，不然你安装RabbitMQ是安装不上的。

开通Stomp协议：

```
rabbitmq-plugins enable rabbitmq_web_stomp
rabbitmq-plugins enable rabbitmq_web_stomp_examples
#重启
service rabbitmq-server stop && service rabbitmq-server start
```

当我们开启之后，在我们的RabbitMQ中使能够看到的，如图：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

大家可以看到，我们正确开启之后，在RabbitMQ的控制台上，我们能够看到http/web-stomp 的端口是15674。

接下来我们就要开始写一个案例进行测试。

#### 前端Stomp监听RabbitMQ

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

如果这个时候我们发送一条消息到消息队列，那么接下来他就会在页面上展示出我们需要的内容。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

我们看看代码是怎么写的吧。

```
 if (typeof WebSocket == 'undefined') {
        console.log('不支持websocket')
    }

    // 初始化 ws 对象

    var ws = new WebSocket('ws://localhost:15674/ws');

    // 获得Stomp client对象
    var client = Stomp.over(ws);

    // 定义连接成功回调函数
    var on_connect = function(x) {
        //data.body是接收到的数据
        client.subscribe("/Fanout_Exchange/testMessage", function(data) {
            var msg = data.body;
            alert("收到数据：" + msg);
        });
    };

    // 定义错误时回调函数
    var on_error =  function() {
        console.log('连接错误，请重试');
    };

    // 连接RabbitMQ
    client.connect('guest', 'guest', on_connect, on_error, '/');
    console.log(">>>RabbitMQ已连接，测试正式开始");
```

而这里面写的内容就比较有意思了，因为之前很多人都会发现，不管怎么写，都是不行，那是因为没有完全的理解，阿粉最后总结了一下关于Stomp的使用。

#### 总结

1.**/exchange/(exchangeName)**

- 对于 SUBCRIBE frame，destination 一般为/exchange/(exchangeName)/[/pattern] 的形式。该 destination 会创建一个唯一的、自动删除的、名为(exchangeName)的 queue，并根据 pattern 将该 queue 绑定到所给的 exchange，实现对该队列的消息订阅。
- 对于 SEND frame，destination 一般为/exchange/(exchangeName)/[/routingKey] 的形式。这种情况下消息就会被发送到定义的 exchange 中，并且指定了 routingKey。

2./queue/(queueName)

- 对于 SUBCRIBE frame，destination 会定义(queueName)的共享 queue，并且实现对该队列的消息订阅。
- 对于 SEND frame，destination 只会在第一次发送消息的时候会定义(queueName)的共享 queue。该消息会被发送到默认的 exchange 中，routingKey 即为(queueName)。

3./amq/queue/(queueName)

- 这种情况下无论是 SUBCRIBE frame 还是 SEND frame 都不会产生 queue。但如果该 queue 不存在，SUBCRIBE frame 会报错。
- 对于 SUBCRIBE frame，destination 会实现对队列(queueName)的消息订阅。
- 对于 SEND frame，消息会通过默认的 exhcange 直接被发送到队列(queueName)中。

4./topic/(topicName)

- 对于 SUBCRIBE frame，destination 创建出自动删除的、非持久的 queue 并根据 routingkey 为(topicName)绑定到 amq.topic exchange 上，同时实现对该 queue 的订阅。
- 对于 SEND frame，消息会被发送到 amq.topic exchange 中，routingKey 为(topicName)。

关于如何在前端监听RabbitMQ消息，你学会了么？



**
**

**代码参考**

RabbitMQ开通STOMP通道



**喜欢就****分享**

**认同就****点赞**

**支持就****在看**

**一键四连，你的offer也四连**![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



![Java极客技术](http://mmbiz.qpic.cn/mmbiz_png/tWOhQMr1wdDwibGUW8HOfmZfVuVryhfO8P8R3vJGrHBmHybX2F0GgHUfL4O9ibP4pYKPNTKQW8um3D6bnqibjLOsA/0?wx_fmt=png)

**Java极客技术**

Java 极客技术由一群热爱 Java 的技术人组建，专业输出高质量原创的 Java 系列文章，优秀的 Java 程序员都在这里。

606篇原创内容



公众号



![img](https://mmbiz.qlogo.cn/mmbiz_jpg/fPADUR9p6acau4uticGBhaiaeH6q8T7g9Zw7BHRDpx9B7BdKKgWZvWOyJRWL0fTZyfJwiaAH4pFKtlb7Cv8vovcCA/0?wx_fmt=jpeg)

鸭血粉丝

原创不易，众筹一碗鸭血粉丝汤

![赞赏二维码](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495409&idx=1&sn=4061e55c0c366608893ff2d50668e660&key=bf0a475117e7a82e0ec3fc12cdf431ea21b0572b148c61e5a08766e3b91c6af0dad30acf87561f5e951d1a4373a7ddc133af8489ea95382cda26db2abc26868a19e4feb8dbcffdfa04f1b273f5957c785c854162c97284f902075702dc3cf72c205d7114d625280e2c2c836d54d004831db757607627062cb2c568a410900bc1&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AdmvfVnSI8Z%2BwBIfDs3GO8o%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2)[喜欢作者](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495409&idx=1&sn=4061e55c0c366608893ff2d50668e660&key=bf0a475117e7a82e0ec3fc12cdf431ea21b0572b148c61e5a08766e3b91c6af0dad30acf87561f5e951d1a4373a7ddc133af8489ea95382cda26db2abc26868a19e4feb8dbcffdfa04f1b273f5957c785c854162c97284f902075702dc3cf72c205d7114d625280e2c2c836d54d004831db757607627062cb2c568a410900bc1&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AdmvfVnSI8Z%2BwBIfDs3GO8o%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2##)

阅读 1230

分享收藏

赞2在看4

写下你的留言