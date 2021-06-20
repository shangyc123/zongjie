## 实战来了！Spring Boot+Redis 分布式锁模拟抢单！

点击关注 👉 [写代码的渣渣鹏](javascript:void(0);) *3月16日*



![图片](https://mmbiz.qpic.cn/mmbiz_jpg/iccLKyfqr2YJgIoTcV6kd36RtPQcb0B4DEuklF2GHJVuBKkRMaicSO8QXlGZeATZN5CyqNjw4KQvoP2A0nico5gTA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

作者：神牛003

来源：cnblogs.com/wangrudong003/p/10627539.html

本篇内容主要讲解的是redis分布式锁，这个在各大厂面试几乎都是必备的，下面结合模拟抢单的场景来使用她；本篇不涉及到的redis环境搭建，快速搭建个人测试环境，这里建议使用docker；本篇内容节点如下：

## jedis的nx生成锁

- 如何删除锁
- 模拟抢单动作(10w个人开抢)
- jedis的nx生成锁

对于java中想操作redis，好的方式是使用jedis，首先pom中引入依赖：

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

对于分布式锁的生成通常需要注意如下几个方面：

- **创建锁的策略** redis的普通key一般都允许覆盖，A用户set某个key后，B在set相同的key时同样能成功，如果是锁场景，那就无法知道到底是哪个用户set成功的；这里jedis的setnx方式为我们解决了这个问题，简单原理是：当A用户先set成功了，那B用户set的时候就返回失败，满足了某个时间点只允许一个用户拿到锁。

  

- **锁过期时间** 某个抢购场景时候，如果没有过期的概念，当A用户生成了锁，但是后面的流程被阻塞了一直无法释放锁，那其他用户此时获取锁就会一直失败，无法完成抢购的活动；当然正常情况一般都不会阻塞，A用户流程会正常释放锁；过期时间只是为了更有保障。

下面来上段setnx操作的代码：

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

这里注意点在于jedis的set方法，其参数的说明如：

- NX：是否存在key，存在就不set成功
- PX：key过期时间单位设置为毫秒（EX：单位秒）

setnx如果失败直接封装返回false即可，下面我们通过一个get方式的api来调用下这个setnx方法：

```
@GetMapping("/setnx/{key}/{val}")
public boolean setnx(@PathVariable String key, @PathVariable String val) {
     return jedisCom.setnx(key, val);
}
```

访问如下测试url，正常来说第一次返回了true，第二次返回了false，由于第二次请求的时候redis的key已存在，所以无法set成功

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)img

由上图能够看到只有一次set成功，并key具有一个有效时间，此时已到达了分布式锁的条件。

## 如何删除锁

上面是创建锁，同样的具有有效时间，但是我们不能完全依赖这个有效时间，场景如：有效时间设置1分钟，本身用户A获取锁后，没遇到什么特殊情况正常生成了抢购订单后，此时其他用户应该能正常下单了才对，但是由于有个1分钟后锁才能自动释放，那其他用户在这1分钟无法正常下单（因为锁还是A用户的），因此我们需要A用户操作完后，主动去解锁：

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

这里也使用了jedis方式，直接执行lua脚本：根据val判断其是否存在，如果存在就del；

其实个人认为通过jedis的get方式获取val后，然后再比较value是否是当前持有锁的用户，如果是那最后再删除，效果其实相当；只不过直接通过eval执行脚本，这样避免多一次操作了redis而已，缩短了原子操作的间隔。(如有不同见解请留言探讨)；同样这里创建个get方式的api来测试：

```
@GetMapping("/delnx/{key}/{val}")
public int delnx(@PathVariable String key, @PathVariable String val) {
   return jedisCom.delnx(key, val);
}
```

注意的是delnx时，需要传递创建锁时的value，因为通过et的value与delnx的value来判断是否是持有锁的操作请求，只有value一样才允许del；

## 模拟抢单动作(10w个人开抢)

有了上面对分布式锁的粗略基础，我们模拟下10w人抢单的场景，其实就是一个并发操作请求而已，由于环境有限，只能如此测试；如下初始化10w个用户，并初始化库存，商品等信息，如下代码：

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

有了上面10w个不同用户，我们设定商品只有10个库存，然后通过并行流的方式来模拟抢购，如下抢购的实现：

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
- (startTime + timeout) >= System.currentTimeMillis()：判断未抢成功的用户，timeout秒内继续获取锁
- 获取锁前和后都判断库存是否还足够
- jedisCom.setnx(shangpingKey, b)：用户获取抢购锁
- 获取锁后并下单成功，最后释放锁：jedisCom.delnx(shangpingKey, b)

再来看下记录的日志结果：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)img

最终返回抢购成功的用户：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)img



阅读 649

分享收藏

赞5在看6

写下你的留言