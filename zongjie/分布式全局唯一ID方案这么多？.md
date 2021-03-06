## 分布式全局唯一ID方案这么多？

前段时间阿粉想着如何去优化我们公司中已经存在的分布式中的唯一ID，而提起唯一的ID，相信如果不是从事传统行业的人，肯定都有所了解，分布式架构下，唯一ID生成方案，是我们在设计一个系统，尤其是数据库使用分库分表的时候常常会遇见的问题，尤其是当我们进行了分库分表之后，对这个唯一ID的要求也就越来越高。那么唯一ID方案都有哪些呢？

### 分布式全局唯一ID

往往一谈分布式，总是会 **色变**,因为在很多面试的时候，都会问你，会不会分布式？你们项目的架构是怎么做的，做的如何？你们既然使用了分布式，那么你们的分布式事务是怎么处理的，你们分布式全局唯一 ID 使用的是什么算法来实现的？

往往谈到这个的时候，很多面试的朋友就会很尴尬，我都是直接用的，我好像完全没有注意过。当你意识到这一点的时候，往往接下来的问题，你回答的就会开始磕磕绊绊，于是面试凉了。

并发越大的系统，数据就越大，数据越大就越需要分布式，而大量的分布式数据就越需要唯一标识来识别它们，而这些唯一标识，我们就称之为分布式全局唯一的ID。

### Redis实现全局唯一ID

阿粉的项目说实话，还不是特别的差劲。于是阿粉就开始想着，这分布式的全局唯一ID，为啥生成的时候都是使用 UUID ，要么就是自增主键呢？

于是阿粉准备使用 Redis 来生成分布式全局唯一ID。

#### Redis实现全局唯一ID原理

因为 Redis 的所有命令是单线程的，所以可以利用 Redis 的原子操作 INCR 和 INCRBY，来生成全局唯一的ID。方式一：StringRedisTemplate

```
public class Activity {
    private Long id;
    private String name;
    private BigDecimal price;
}
```

上面是我们的活动的实体类，马上就要 618 了,各位做电商的是不是开始准备搞事情了？可以学习一下用一下试试，我们活动中有 id ，活动的名称 name ，还有对应活动设置好的价格 price 等等，字段可能还会有很多，我们需要的暂时就列出这么多。

```
public class IdGeneratorService {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    private static final String ID_KEY = "id:generator:activity";

    public Long incrementId() {
        return stringRedisTemplate.opsForValue().increment(ID_KEY);
    }
long id = idGeneratorService.incrementId(); 调用生成
```

但是看起来是不是总是感觉好像有点 low ,我们是不是就要准备来整的高大上一点，毕竟代码就像一个程序员的内裤，虽然自己看着有洞感觉没啥，但是别人看到是不是就很不爽了，那就整个他们看起来比较高大上一点的。

方式二:

为什么会有方案二，那是因为我们的 Redis 很多时候都不是只有一个 Redis，都是搭建的集群，既然是集群，我们就要开始合理的利用上集群。

那么我们就要开始考虑到集群方面的知识了，那么我们的思路就有了。于是出现了：集群中每个节点预生成生成ID；然后与redis的已经存在的ID做比较。如果大于，则取节点生成的ID；小于的话，取Redis中最大ID自增。

这个时候我们还需要一段 lua 脚本来保证我们实现的ID是唯一的，这才是真正的本质，不然我们实现的ID在高端，不唯一，有个锤子用

核心脚本:

```
local function get_max_seq()
    local key = tostring(KEYS[1])
    local incr_amoutt = tonumber(KEYS[2])
    local seq = tostring(KEYS[3])
    local month_in_seconds = 24 * 60 * 60 * 30
    if (1 == redis.call(\'setnx\', key, seq))
    then
        redis.call(\'expire\', key, month_in_seconds)
        return seq
    else
        local prev_seq = redis.call(\'get\', key)
        if (prev_seq < seq)
        then
            redis.call(\'set\', key, seq)
            return seq
        else
        --[[
            不能直接返回redis.call(\'incr\', key),因为返回的是number浮点数类型,会出现不精确情况。
            注意: 类似"16081817202494579"数字大小已经快超时lua和reids最大数值,请谨慎的增加seq的位数
        --]]
            redis.call(\'incrby\', key, incr_amoutt)
            return redis.call(\'get\', key)
        end
    end
end
return get_max_seq()
```

以上的 lua 的脚本来自于Ydoing，一个博客的大佬，我们现在既然会使用他生成全局唯一的ID，那么是不是就得搞清楚为什么会选择 Redis 来实现分布式全局唯一的ID。

### Redis 的所有命令是单线程的

上一段开头，阿粉就说 Redis 的命令都是单线程的，相信如果你在面试官面前这么说，面试官肯定会问你一句，为什么 Redis 是单线程而不是多线程的呢？

Redis 基于 Reactor 模式开发了网络事件处理器，这个处理器被称为文件事件处理器。它的组成结构为4部分：多个套接字、IO多路复用程序、文件事件分派器、事件处理器。因为文件事件分派器队列的消费是单线程的，所以 Redis 才叫单线程模型。

当你说到这个 Reactor 模式的时候，如果大家深入研究过 Netty 的模型，就会发现，这个模式在 Netty 中也是有使用的，我们这时候是不是就得需要去官网上去瞅瞅看，为什么这么说。

#### 什么是Reactor模型

Reactor模型实际上都知道，就是一个多路复用I/O模型，主要用于在高并发、高吞吐量的环境中进行I/O处理。

而这种多路复用的模型所依赖的永远都是那么几个内容，事件分发器，事件处理器，还有调用的客户端，

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

Reactor模型是一个同步的I/O多路复用模型，我们还是先把这个同步的I/O多路复用模型给弄清楚了再看其他的。

这个相信大家肯定不是很熟悉，而阿粉在之前也给大家说了关于Netty中的Channel，文章地址发给大家，[用Socket编程？我还是选择了Netty](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495163&idx=1&sn=7136f3fbe19c5393c4a4863de9fcd4db&scene=21#wechat_redirect),在文章中，我们已经给大家说了关于Channel，而这种单线程的模型是什么样子的呢？

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

图已经给大家画出来了，丑是丑了点，但是意思还是表达出来了。

这种模型也就是说：Redis 单线程指的是网络请求模块使用了一个线程（所以不需考虑并发安全性），即一个线程处理所有网络请求，其他模块仍用了多个线程。

而面试官还会有一种问法，为什么使用 Redis 就会快。

这个相信大家肯定能回答出来，因为 Redis 是一种基于内存的存储数据，为什么内存快？

因为这种快速是针对存储在磁盘上的数据来说的，因为内存中的数据，断电之后，消失了，你下次来的时候，不还是需要从磁盘读取出来，然后保存，所以说在Redis速度快。扯远了，回来继续说 Redis的单线程。

我们来看看官网给我们的解释：

```
Redis is single threaded. How can I exploit multiple CPU / cores?
It's not very frequent that CPU becomes your bottleneck with Redis, as usually Redis is either memory or network bound. For instance, using pipelining Redis running on an average Linux system can deliver even 1 million requests per second, so if your application mainly uses O(N) or O(log(N)) commands, it is hardly going to use too much CPU.

However, to maximize CPU usage you can start multiple instances of Redis in the same box and treat them as different servers. At some point a single box may not be enough anyway, so if you want to use multiple CPUs you can start thinking of some way to shard earlier.

You can find more information about using multiple Redis instances in the Partitioning page.

However with Redis 4.0 we started to make Redis more threaded. For now this is limited to deleting objects in the background, and to blocking commands implemented via Redis modules. For future releases, the plan is to make Redis more and more threaded.
```

其实翻译过来大致就是说，当我们使用 Redis 的时候，CPU 成为瓶颈的情况并不常见，因为 Redis 通常是内存或网络受限的。

其实说白了，官网就是说我们 Redis 就是这么的快，并且正是由于在单线程模式的情况下已经很快了，就没有必要在使用多线程了。这整的是不是就有点恶心了。阿粉也说说自己的见解，毕竟这官网的话有点糊弄人的意思。

其实 Redis 使用单个 CPU 绑定一个内存，针对内存的处理就是单线程的,而我们使用多个 CPU 模拟出多个线程来，光在多个 CPU 之间的切换，然后再操作 Redis ，实际上就不如直接从内存中拿出来，毕竟耗时在这里摆着。

你认为的 Redis 为什么是单线程的？