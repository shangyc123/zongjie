## 图解 Kafka，一目了然！

Kafka 是主流的消息流系统，其中的概念还是比较多的，下面通过图示的方式来梳理一下 Kafka 的核心概念，以便在我们的头脑中有一个清晰的认识。

## **基础**

Kafka 是一套流处理系统，可以让后端服务轻松的相互沟通，是微服务架构中常用的组件。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicauStG0kMRjibaxNpSQiaIfq3zEThf8sXWhh2BajVrey4FS4cE0ibR5wRA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **生产者消费者**

生产者服务 Producer 向 Kafka 发送消息，消费者服务 Consumer 监听 Kafka 接收消息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicROXPOc8dRCiaLiaP0N8WlRgJpYQicyEVWPDREZSLCEFTT4dmQW5HickRew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一个服务可以同时为生产者和消费者。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicErTWZH5oKxoZFozibuFIRO0PwnDZ433LzOE2ZD4cVcZXibibPw5mPYaoQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **Topics 主题**

Topic 是生产者发送消息的目标地址，是消费者的监听目标。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5Tjqcmoichbds9OT6XA1Vgjq86103WzlDk1bVPFwFASUiaEU1N1puhp0s9qmvwvw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

一个服务可以监听、发送多个 Topics。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5Tjqcmoic9VNic1y2WM5OTT2kxWMv7W0NzKlQNG5ZGpIZ2bCJdBDgyVY9zx2iaLhQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Kafka 中有一个【consumer-group（消费者组）】的概念。

这是一组服务，扮演一个消费者。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicCXCEiaZrwpNp1LGuCqPNXic96DddpndDrCcEqNr8e7IqUnNXEnfdOsAw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果是消费者组接收消息，Kafka 会把一条消息路由到组中的某一个服务。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicEq46rljUkYkoRJyUunBkGpFv5EuOGqBopyNU25uSOFY1GjP3zNquFg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这样有助于消息的负载均衡，也方便扩展消费者。

Topic 扮演一个消息的队列。

首先，一条消息发送了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicuppBR4X9S8H9gCG7jiboHbFwicOtXJSI1dBnKQ73x8HFZBxDbYsfndxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

然后，这条消息被记录和存储在这个队列中，不允许被修改。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5Tjqcmoic94x1qGvRNDZMY0MiaayOn6OGVia8qny7yJ0wtmXeXY2EEbzhr47OOjnw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

接下来，消息会被发送给此 Topic 的消费者。

但是，这条消息并不会被删除，会继续保留在队列中。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicRjxquLK3icNWVpHOAicgOEsKn34rO1oDvv8XGslt8EC0X5HWiaxp1pNFA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

继续发送消息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicibMX0mYOzyAdZ6zwmhIfnZelS6TZGhwo5Rwnuw17SVUhzjDmURyI2ng/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

像之前一样，这条消息会发送给消费者、不允许被改动、一直呆在队列中。

（消息在队列中能呆多久，可以修改 Kafka 的配置）

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicdextM1lTUkyNZveFSdBTfYm5ibBes8t3Y7WThCo5Skib5qrHJM6ejVKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoiceC6jqOkYFjtPoeOAo1wvsbFtDel2RHtibficibBLdAia9N7kXOGBibtKb6g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **Partitions 分区**

上面 Topic 的描述中，把 Topic 看做了一个队列，实际上，一个 Topic 是由多个队列组成的，被称为【Partition（分区）】。

这样可以便于 Topic 的扩展。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoichnToavnycnkmcu8u3ibIJVo2jicc3EDEUhk7h8T8aRyVCXqpxEIciaKcg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

生产者发送消息的时候，这条消息会被路由到此 Topic 中的某一个 Partition。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicRE2C7icnPMYibX0znPumSqrSlXwKibSFWGSfnqdibXSKQlTA0cS234AmkQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

消费者监听的是所有分区。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoiczCEwBDibWC1vz3adhaRoC4tQia4S0WjN38TqiaWuzCV4p6h7icNJBLdbOg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

生产者发送消息时，默认是面向 Topic 的，由 Topic 决定放在哪个 Partition，默认使用轮询策略。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicKzFIfLcEgicGicmpmUFVD4LYgLYSUua8gT8OE5r10VWmnH6bBscIJpuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

也可以配置 Topic，让同类型的消息都在同一个 Partition。

例如，处理用户消息，可以让某一个用户所有消息都在一个 Partition。

例如，用户1发送了3条消息：A、B、C，默认情况下，这3条消息是在不同的 Partition 中（如 P1、P2、P3）。

在配置之后，可以确保用户1的所有消息都发到同一个分区中（如 P1）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoiciaQjQmnl41Ar8CboQn8XZVIILicbUIBDLNrQkwM9LLGQl9W4NZobXGpQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这个功能有什么用呢？

这是为了提供消息的【有序性】。

消息在不同的 Partition 是不能保证有序的，只有一个 Partition 内的消息是有序的。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5Tjqcmoicx1YpULtrqbaOgSCGBhntg8uu3yicUz7h7UNbwA10Eb2Ku2gvMKCTPCA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5Tjqcmoicg73MqiaWljQyKfU38WnmEu3gTgWyOEVoaSHvFpyTxkicMv6qDTLSMauQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **架构**

Kafka 是集群架构的，ZooKeeper是重要组件。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicavMQoMgwicia0YnyJGGb5fpz5eMo7LIS08sMTnChljTpoeoicEzAwXYXQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ZooKeeper 管理者所有的 Topic 和 Partition。

Topic 和 Partition 存储在 Node 物理节点中，ZooKeeper负责维护这些 Node。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicBuF4Z1fOpC2MGUp8Ufn1UlglztAhU8weJK2Io9dBWl8qQTj39xC6xw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

例如，有2个 Topic，各自有2个 Partition。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicB4BfyIMhF376sMnh3spMH64JRU2PHuWSyibORs0u5Kl67EyYCwK2Y2Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这是逻辑上的形式，但在 Kafka 集群中的实际存储可能是这样的：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicYupnR1QbkT1KESxeI8qn7W1ZVG2c6qH7Hj7HgktTWpX33RdIxBUvxw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Topic A 的 Partition #1 有3份，分布在各个 Node 上。

这样可以增加 Kafka 的可靠性和系统弹性。

3个 Partition #1 中，ZooKeeper 会指定一个 Leader，负责接收生产者发来的消息。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicrlL0sSDv7QZhKiabCWoTyMwHsYiajqMg2VVkaLECe5yhSLzZ2NYSrCVg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其他2个 Partition #1 会作为 Follower，Leader 接收到的消息会复制给 Follower。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicbWicLghtibmBYRNl5HsHgflblx5f3pz16EMpkuftw7kOdUE38AKicE2icw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这样，每个 Partition 都含有了全量消息数据。

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicZWeZdL1jYGzmgRZsGEic485XibAE0eaIBTeACXyibtia5eLSv3nrfKwXoA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

即使某个 Node 节点出现了故障，也不用担心消息的损坏。

Topic A 和 Topic B 的所有 Partition 分布可能就是这样的：

![图片](https://mmbiz.qpic.cn/mmbiz_png/JfTPiahTHJhqkujUna87NNp1I5TjqcmoicgdEE27xcxDaguTibDIf6T9icEibCNh5ThWhXqHDrHwjviaiaNyroiaFL29qQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

