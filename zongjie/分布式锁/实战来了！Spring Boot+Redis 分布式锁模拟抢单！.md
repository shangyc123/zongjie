## 实战来了！Spring Boot+Redis 分布式锁模拟抢单！

[MarkerHub](javascript:void(0);) *4月7日*

#### 小Hub领读：

嘿嘿，学一学如何使用redis来接近超买超卖问题！

------

> 作者：神牛003
>
> https://cnblogs.com/wangrudong003/p/10627539.html
>
> git：https://github.com/shenniubuxing3

本篇内容主要讲解的是 redis 分布式锁，这个在各大厂面试几乎都是必备的，下面结合模拟抢单的场景来使用她；本篇不涉及到的 redis 环境搭建，快速搭建个人测试环境，这里建议使用 docker；本篇内容节点如下：

## jedis 的 nx 生成锁

- 如何删除锁
- 模拟抢单动作 (10w 个人开抢)
- jedis 的 nx 生成锁

对于 java 中想操作 redis，好的方式是使用 jedis，首先 pom 中引入依赖：

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

对于分布式锁的生成通常需要注意如下几个方面：

- **创建锁的策略** redis 的普通 key 一般都允许覆盖，A 用户 set 某个 key 后，B 在 set 相同的 key 时同样能成功，如果是锁场景，那就无法知道到底是哪个用户 set 成功的；这里 jedis 的 setnx 方式为我们解决了这个问题，简单原理是：当 A 用户先 set 成功了，那 B 用户 set 的时候就返回失败，满足了某个时间点只允许一个用户拿到锁。搜索公众号：[MarkerHub](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455548310&idx=2&sn=28559512311234ab380c18f124ac5246&scene=21#wechat_redirect)，关注回复[vue]获取[前后端入门教程](https://mp.weixin.qq.com/s?__biz=MzI4OTA3NDQ0Nw==&mid=2455548310&idx=2&sn=28559512311234ab380c18f124ac5246&scene=21#wechat_redirect)！
- **锁过期时间** 某个抢购场景时候，如果没有过期的概念，当 A 用户生成了锁，但是后面的流程被阻塞了一直无法释放锁，那其他用户此时获取锁就会一直失败，无法完成抢购的活动；当然正常情况一般都不会阻塞，A 用户流程会正常释放锁；过期时间只是为了更有保障。

下面来上段 setnx 操作的代码：

```
public boolean setnx(String key, String val) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            if (jedis == null) {
                return false;
            }
            return jedis.set(key, val, "NX", "PX", 1000 * 60).
                    equalsIgnoreCase("ok");
        } catch (Exception ex) {
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return false;
    }
```

这里注意点在于 jedis 的 set 方法，其参数的说明如：

- NX：是否存在 key，存在就不 set 成功
- PX：key 过期时间单位设置为毫秒（EX：单位秒）

setnx 如果失败直接封装返回 false 即可，下面我们通过一个 get 方式的 api 来调用下这个 setnx 方法：

```
@GetMapping("/setnx/{key}/{val}")
public boolean setnx(@PathVariable String key, @PathVariable String val) {
     return jedisCom.setnx(key, val);
}
```

访问如下测试 url，正常来说第一次返回了 true，第二次返回了 false，由于第二次请求的时候 redis 的 key 已存在，所以无法 set 成功

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==) 由上图能够看到只有一次 set 成功，并 key 具有一个有效时间，此时已到达了分布式锁的条件。

## 如何删除锁

上面是创建锁，同样的具有有效时间，但是我们不能完全依赖这个有效时间，场景如：有效时间设置 1 分钟，本身用户 A 获取锁后，没遇到什么特殊情况正常生成了抢购订单后，此时其他用户应该能正常下单了才对，但是由于有个 1 分钟后锁才能自动释放，那其他用户在这 1 分钟无法正常下单（因为锁还是 A 用户的），因此我们需要 A 用户操作完后，主动去解锁：

```
public int delnx(String key, String val) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            if (jedis == null) {
                return 0;
            }

            //if redis.call('get','orderkey')=='1111' then return redis.call('del','orderkey') else return 0 end
            StringBuilder sbScript = new StringBuilder();
            sbScript.append("if redis.call('get','").append(key).append("')").append("=='").append(val).append("'").
                    append(" then ").
                    append("    return redis.call('del','").append(key).append("')").
                    append(" else ").
                    append("    return 0").
                    append(" end");

            return Integer.valueOf(jedis.eval(sbScript.toString()).toString());
        } catch (Exception ex) {
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return 0;
    }
```

这里也使用了 jedis 方式，直接执行 lua 脚本：根据 val 判断其是否存在，如果存在就 del；

其实个人认为通过 jedis 的 get 方式获取 val 后，然后再比较 value 是否是当前持有锁的用户，如果是那最后再删除，效果其实相当；只不过直接通过 eval 执行脚本，这样避免多一次操作了 redis 而已，缩短了原子操作的间隔。(如有不同见解请留言探讨)；同样这里创建个 get 方式的 api 来测试：

```
@GetMapping("/delnx/{key}/{val}")
public int delnx(@PathVariable String key, @PathVariable String val) {
   return jedisCom.delnx(key, val);
}
```

注意的是 delnx 时，需要传递创建锁时的 value，因为通过 et 的 value 与 delnx 的 value 来判断是否是持有锁的操作请求，只有 value 一样才允许 del；

## 模拟抢单动作 (10w 个人开抢)

有了上面对分布式锁的粗略基础，我们模拟下 10w 人抢单的场景，其实就是一个并发操作请求而已，由于环境有限，只能如此测试；如下初始化 10w 个用户，并初始化库存，商品等信息，如下代码：

```
//总库存
    private long nKuCuen = 0;
    //商品key名字
    private String shangpingKey = "computer_key";
    //获取锁的超时时间 秒
    private int timeout = 30 * 1000;

    @GetMapping("/qiangdan")
    public List<String> qiangdan() {

        //抢到商品的用户
        List<String> shopUsers = new ArrayList<>();

        //构造很多用户
        List<String> users = new ArrayList<>();
        IntStream.range(0, 100000).parallel().forEach(b -> {
            users.add("神牛-" + b);
        });

        //初始化库存
        nKuCuen = 10;

        //模拟开抢
        users.parallelStream().forEach(b -> {
            String shopUser = qiang(b);
            if (!StringUtils.isEmpty(shopUser)) {
                shopUsers.add(shopUser);
            }
        });

        return shopUsers;
    }
```

有了上面 10w 个不同用户，我们设定商品只有 10 个库存，然后通过并行流的方式来模拟抢购，如下抢购的实现：

```
/**
     * 模拟抢单动作
     *
     * @param b
     * @return
     */
    private String qiang(String b) {
        //用户开抢时间
        long startTime = System.currentTimeMillis();

        //未抢到的情况下，30秒内继续获取锁
        while ((startTime + timeout) >= System.currentTimeMillis()) {
            //商品是否剩余
            if (nKuCuen <= 0) {
                break;
            }
            if (jedisCom.setnx(shangpingKey, b)) {
                //用户b拿到锁
                logger.info("用户{}拿到锁...", b);
                try {
                    //商品是否剩余
                    if (nKuCuen <= 0) {
                        break;
                    }

                    //模拟生成订单耗时操作，方便查看：神牛-50 多次获取锁记录
                    try {
                        TimeUnit.SECONDS.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                    //抢购成功，商品递减，记录用户
                    nKuCuen -= 1;

                    //抢单成功跳出
                    logger.info("用户{}抢单成功跳出...所剩库存：{}", b, nKuCuen);

                    return b + "抢单成功，所剩库存：" + nKuCuen;
                } finally {
                    logger.info("用户{}释放锁...", b);
                    //释放锁
                    jedisCom.delnx(shangpingKey, b);
                }
            } else {
                //用户b没拿到锁，在超时范围内继续请求锁，不需要处理
//                if (b.equals("神牛-50") || b.equals("神牛-69")) {
//                    logger.info("用户{}等待获取锁...", b);
//                }
            }
        }
        return "";
    }
```

这里实现的逻辑是：

- parallelStream()：并行流模拟多用户抢购
- (startTime + timeout) >= System.currentTimeMillis()：判断未抢成功的用户，timeout 秒内继续获取锁
- 获取锁前和后都判断库存是否还足够
- jedisCom.setnx(shangpingKey, b)：用户获取抢购锁
- 获取锁后并下单成功，最后释放锁：jedisCom.delnx(shangpingKey, b)

再来看下记录的日志结果：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

最终返回抢购成功的用户：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

------

（完）

```
【推荐阅读】编写 if 时不带 else，你的代码会更好！
一文详解微服务架构
懵了！面试官问我：String 长度有限制吗？是多少？

```

好文章！点个在看！

阅读 3556

分享收藏

赞6在看7

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/RDPJnAXeokT3zoodxyr1ibfUheBukSQBORpbSmXHYlSOTr6oLOIvicb6x8dKKQ2MqH5HS7NQKMRJ6srldGPiaNgFMw6A3bUY5Zk/96)

  怅惘~

  2

  

  lua脚本删除和代码get后比较再删除还是有区别的,get和del拆除2个操作就非原子性了,在get后的一瞬间可能由于系统异常导致未执行del从而导致锁无法在正常业务结束后准时释放

- ![img](http://wx.qlogo.cn/mmopen/RDPJnAXeokQJXicUc5bHiaaUVBzqyeZFRibXpAcPjESJ3c0O4YtHgRa7mLRTsgFK4ictgEVgU0NTm1G9Ra6arK9KtaES5a9iaibwkP/96)

  雾散

  1

  

  会有死锁 释放错锁的坑 redission针不戳