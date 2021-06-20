## 使用uuid作为数据库主键，被技术总监怼了一顿！

原创 鸭血粉丝 [Java极客技术](javascript:void(0);) *4月12日*

每天早上七点三十，准时推送干货



![图片](https://mmbiz.qpic.cn/mmbiz_jpg/laEmibHFxFw554pf0WN6FeiaB2rHpR4nrjpsfmuXsiaKC8O64oOHQMict7nk9icic6wE9O0lfvrlmz8MokrO6euO105w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



看完本文，你一定会有所收获

### 一、摘要

在日常开发中，数据库中主键id的生成方案，主要有三种

- 数据库自增ID
- 采用随机数生成不重复的ID
- 采用jdk提供的uuid

对于这三种方案，我发现在数据量少的情况下，没有特别的差异，但是当单表的数据量达到百万级以上时候，他们的性能有着显著的区别，光说理论不行，还得看实际程序测试，今天小编就带着大家一探究竟！

### 二、程序实例

首先，我们在本地数据库中创建三张单表`tb_uuid_1`、`tb_uuid_2`、`tb_uuid_3`，同时设置`tb_uuid_1`表的主键为自增长模式，脚本如下：

```
CREATE TABLE `tb_uuid_1` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='主键ID自增长';
CREATE TABLE `tb_uuid_2` (
  `id` bigint(20) unsigned NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='主键ID随机数生成';
CREATE TABLE `tb_uuid_3` (
  `id` varchar(50)  NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='主键采用uuid生成';
```

下面，我们采用`Springboot + mybatis`来实现插入测试。

#### 2.1、数据库自增

以数据库自增为例，首先编写好各种实体、数据持久层操作，方便后续进行测试

```
/**
 * 表实体
 */
public class UUID1 implements Serializable {

    private Long id;

    private String name;
  
  //省略set、get
}
/**
 * 数据持久层操作
 */
public interface UUID1Mapper {

    /**
     * 自增长插入
     * @param uuid1
     */
    @Insert("INSERT INTO tb_uuid_1(name) VALUES(#{name})")
    void insert(UUID1 uuid1);
}
/**
 * 自增ID，单元测试
 */
@Test
public void testInsert1(){
    long start = System.currentTimeMillis();
    for (int i = 0; i < 1000000; i++) {
        uuid1Mapper.insert(new UUID1().setName("张三"));
    }
    long end = System.currentTimeMillis();
    System.out.println("花费时间：" +  (end - start));
}
```

#### 2.2、采用随机数生成ID

这里，我们**采用`twitter`的雪花算法来实现随机数ID的生成**，工具类如下：

```
public class SnowflakeIdWorker {

    private static SnowflakeIdWorker instance = new SnowflakeIdWorker(0,0);

    /**
     * 开始时间截 (2015-01-01)
     */
    private final long twepoch = 1420041600000L;
    /**
     * 机器id所占的位数
     */
    private final long workerIdBits = 5L;
    /**
     * 数据标识id所占的位数
     */
    private final long datacenterIdBits = 5L;
    /**
     * 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
     */
    private final long maxWorkerId = -1L ^ (-1L << workerIdBits);
    /**
     * 支持的最大数据标识id，结果是31
     */
    private final long maxDatacenterId = -1L ^ (-1L << datacenterIdBits);
    /**
     * 序列在id中占的位数
     */
    private final long sequenceBits = 12L;
    /**
     * 机器ID向左移12位
     */
    private final long workerIdShift = sequenceBits;
    /**
     * 数据标识id向左移17位(12+5)
     */
    private final long datacenterIdShift = sequenceBits + workerIdBits;
    /**
     * 时间截向左移22位(5+5+12)
     */
    private final long timestampLeftShift = sequenceBits + workerIdBits + datacenterIdBits;
    /**
     * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
     */
    private final long sequenceMask = -1L ^ (-1L << sequenceBits);
    /**
     * 工作机器ID(0~31)
     */
    private long workerId;
    /**
     * 数据中心ID(0~31)
     */
    private long datacenterId;
    /**
     * 毫秒内序列(0~4095)
     */
    private long sequence = 0L;
    /**
     * 上次生成ID的时间截
     */
    private long lastTimestamp = -1L;
    /**
     * 构造函数
     * @param workerId     工作ID (0~31)
     * @param datacenterId 数据中心ID (0~31)
     */
    public SnowflakeIdWorker(long workerId, long datacenterId) {
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        if (datacenterId > maxDatacenterId || datacenterId < 0) {
            throw new IllegalArgumentException(String.format("datacenter Id can't be greater than %d or less than 0", maxDatacenterId));
        }
        this.workerId = workerId;
        this.datacenterId = datacenterId;
    }
    /**
     * 获得下一个ID (该方法是线程安全的)
     * @return SnowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();
        // 如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(
                    String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }
        // 如果是同一时间生成的，则进行毫秒内序列
        if (lastTimestamp == timestamp) {
            sequence = (sequence + 1) & sequenceMask;
            // 毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = tilNextMillis(lastTimestamp);
            }
        }
        // 时间戳改变，毫秒内序列重置
        else {
            sequence = 0L;
        }
        // 上次生成ID的时间截
        lastTimestamp = timestamp;
        // 移位并通过或运算拼到一起组成64位的ID
        return ((timestamp - twepoch) << timestampLeftShift) //
                | (datacenterId << datacenterIdShift) //
                | (workerId << workerIdShift) //
                | sequence;
    }
    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     * @param lastTimestamp 上次生成ID的时间截
     * @return 当前时间戳
     */
    protected long tilNextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = timeGen();
        }
        return timestamp;
    }
    /**
     * 返回以毫秒为单位的当前时间
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }

    public static SnowflakeIdWorker getInstance(){
        return instance;
    }


    public static void main(String[] args) throws InterruptedException {
        SnowflakeIdWorker idWorker = SnowflakeIdWorker.getInstance();
        for (int i = 0; i < 10; i++) {
            long id = idWorker.nextId();
            Thread.sleep(1);
            System.out.println(id);
        }
    }
}
```

其他的操作，与上面类似。

#### 2.3、uuid

同样的，uuid的生成，我们事先也可以将工具类编写好：

```
public class UUIDGenerator {

    /**
     * 获取uuid
     * @return
     */
    public static String getUUID(){
        return UUID.randomUUID().toString();
    }
}
```

最后的单元测试，代码如下：

```
@RunWith(SpringRunner.class)
@SpringBootTest()
public class UUID1Test {

    private static final Integer MAX_COUNT = 1000000;

    @Autowired
    private UUID1Mapper uuid1Mapper;

    @Autowired
    private UUID2Mapper uuid2Mapper;

    @Autowired
    private UUID3Mapper uuid3Mapper;

    /**
     * 测试自增ID耗时
     */
    @Test
    public void testInsert1(){
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
            uuid1Mapper.insert(new UUID1().setName("张三"));
        }
        long end = System.currentTimeMillis();
        System.out.println("自增ID，花费时间：" +  (end - start));
    }

    /**
     * 测试采用雪花算法生产的随机数ID耗时
     */
    @Test
    public void testInsert2(){
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
            long id = SnowflakeIdWorker.getInstance().nextId();
            uuid2Mapper.insert(new UUID2().setId(id).setName("张三"));
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间：" +  (end - start));
    }

    /**
     * 测试采用UUID生成的ID耗时
     */
    @Test
    public void testInsert3(){
        long start = System.currentTimeMillis();
        for (int i = 0; i < MAX_COUNT; i++) {
            String id = UUIDGenerator.getUUID();
            uuid3Mapper.insert(new UUID3().setId(id).setName("张三"));
        }
        long end = System.currentTimeMillis();
        System.out.println("花费时间：" +  (end - start));
    }
}
```

### 三、性能测试

程序环境搭建完成之后，啥也不说了，直接撸起袖子，将单元测试跑起来！

首先测试一下，插入100万数据的情况下，三者直接的耗时结果如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在原有的数据量上，我们继续插入30万条数据，三者耗时结果如下：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可以看出在数据量 100W 左右的时候，uuid的插入效率垫底，随着插入的数据量增长，uuid 生成的ID插入呈直线下降！

时间占用量总体效率排名为：**自增ID > 雪花算法生成的ID >> uuid生成的ID**。

**在数据量较大的情况下，为什么uuid生成的ID远不如自增ID呢**？

关于这点，我们可以从 mysql 主键存储的内部结构来进行分析。

#### 3.1、自增ID内部结构

自增的主键的值是顺序的，所以 Innodb 把每一条记录都存储在一条记录的后面。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

当达到页面的最大填充因子时候(innodb默认的最大填充因子是页大小的15/16,会留出1/16的空间留作以后的修改)，会进行如下操作：

- 下一条记录就会写入新的页中，一旦数据按照这种顺序的方式加载，主键页就会近乎于顺序的记录填满，提升了页面的最大填充率，不会有页的浪费
- 新插入的行一定会在原有的最大数据行下一行，mysql定位和寻址很快，不会为计算新行的位置而做出额外的消耗

#### 3.2、使用uuid的索引内部结构

uuid相对顺序的自增id来说是毫无规律可言的，新行的值不一定要比之前的主键的值要大，所以innodb无法做到总是把新行插入到索引的最后，而是需要为新行寻找新的合适的位置从而来分配新的空间。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这个过程需要做很多额外的操作，数据的毫无顺序会导致数据分布散乱，将会导致以下的问题：

- 写入的目标页很可能已经刷新到磁盘上并且从缓存上移除，或者还没有被加载到缓存中，innodb在插入之前不得不先找到并从磁盘读取目标页到内存中，这将导致大量的随机IO
- 因为写入是乱序的,innodb不得不频繁的做页分裂操作,以便为新的行分配空间,页分裂导致移动大量的数据，一次插入最少需要修改三个页以上
- 由于频繁的页分裂，页会变得稀疏并被不规则的填充，最终会导致数据会有碎片

在把值载入到聚簇索引(innodb默认的索引类型)以后，有时候会需要做一次`OPTIMEIZE TABLE`来重建表并优化页的填充，这将又需要一定的时间消耗。

因此，在选择主键ID生成方案的时候，尽可能别采用uuid的方式来生成主键ID，随着数据量越大，插入性能会越低！

### 四、总结

在实际使用过程中，推荐使用主键自增ID和雪花算法生成的随机ID。

但是使用自增ID也有缺点：

1、别人一旦爬取你的数据库，就可以根据数据库的自增id获取到你的业务增长信息，很容易进行数据窃取。2、其次，对于高并发的负载，innodb在按主键进行插入的时候会造成明显的锁争用，主键的上界会成为争抢的热点，因为所有的插入都发生在这里，并发插入会导致间隙锁竞争。

总结起来，如果业务量小，推荐采用自增ID，如果业务量大，推荐采用雪花算法生成的随机ID。

本篇文章主要从实际程序实例出发，讨论了三种主键ID生成方案的性能差异， 鉴于笔者才疏学浅，可能也有理解不到位的地方，欢迎网友们批评指出！

### 五、参考

1、[方志明 - 使用雪花id或uuid作为Mysql主键，被老板怼了一顿！](https://mp.weixin.qq.com/s?__biz=MzAxNjk4ODE4OQ==&mid=2247504126&idx=2&sn=d03614253398de4f72430070a91aa2ee&scene=21#wechat_redirect)



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



![img](https://mmbiz.qlogo.cn/mmbiz_jpg/fPADUR9p6acau4uticGBhaiaeH6q8T7g9Zw7BHRDpx9B7BdKKgWZvWOyJRWL0fTZyfJwiaAH4pFKtlb7Cv8vovcCA/0?wx_fmt=jpeg)

鸭血粉丝

原创不易，众筹一碗鸭血粉丝汤

![赞赏二维码](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247494971&idx=1&sn=365eaf74ef16878af390883606f5fadc&key=bf0a475117e7a82e8b6959477776808665cff441ffe780a190e485283e14781063afde0d8c3e1bdc8412fb5555c8e38efa3edd09b80452b01170bc1ac8f638ecaa3e9664e9c20be6405a05e856287dcbefea8b50f4db0e47c94bfd2d29eefbc78e534793d8bf591f25bbf3e291f743d0e63a561eb90130d32c985038530889fd&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AQc3z2G05B3MKVJH6zV8%2Bk4%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2)[稀罕作者](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247494971&idx=1&sn=365eaf74ef16878af390883606f5fadc&key=bf0a475117e7a82e8b6959477776808665cff441ffe780a190e485283e14781063afde0d8c3e1bdc8412fb5555c8e38efa3edd09b80452b01170bc1ac8f638ecaa3e9664e9c20be6405a05e856287dcbefea8b50f4db0e47c94bfd2d29eefbc78e534793d8bf591f25bbf3e291f743d0e63a561eb90130d32c985038530889fd&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AQc3z2G05B3MKVJH6zV8%2Bk4%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2##)

[1](javascript:;) 人喜欢

![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM50icmo8BicWIuoGTicibcR6UxNHoIXtkJ6uVOOxGicN8MqH0Q/132)

阅读 3912

分享收藏

赞26在看16

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/0jPKUo2uNZCNlNHvFNu1MXhVy5mc8RT4Ic3NQz2fH9mRV0koh4pD8l12waYJWtd9d9mHJbWD5CIsf7iaHW4icD6ezLvBsf6uFc/96)

  zf

  2

  

  雪花算法生成的主键是因为时间戳的原因所以也能顺序插入吗？

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)

  1

  

  对的

  ![img](http://wx.qlogo.cn/mmopen/0jPKUo2uNZCNlNHvFNu1MXhVy5mc8RT4Ic3NQz2fH9mRV0koh4pD8l12waYJWtd9d9mHJbWD5CIsf7iaHW4icD6ezLvBsf6uFc/96)

  zf

  1

- ![img](http://wx.qlogo.cn/mmopen/HsUHg9qicbJVgKSZoCzlfuShTiblTNOqGgFpPw7tSc9qJGEsib16QGCV8YLrV6Pk2IJzC91KWQGpiaqia6JCaCgBPmYbwTRJDsoKz/96)

  散淡样子

  1

  

  几个问题请教一下： 1.使用自增id达到上限怎么办？ 2.雪花id会不会像自增id一样达到上限？ 3.雪花id必须要自己生成出来去赋值到实体中吗？可以通过注解等方式自动生成吗？ 4.如果必须自己赋值，我可以直接复制你的雪花算法代码使用吗？

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)

  1

  

  自增ID，如果类型是long，那么就有上百亿数据，单表达到亿级，这种情况直接分库分表了，分库分表下一般采用雪花算法ID来作为主键；雪花算法是根据机器码+时间戳+随机数来生成的，不存在上限；直接通过工具类来获取ID就可以了，通过注解方式来生成相反会把功能搞的更加复杂；

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)

  1

  

  可以直接复制雪花算法代码，但是需要修改workerid和dataid，表示机器ID+数据中心id，根据自己集群来配置

  ![img](http://wx.qlogo.cn/mmopen/HsUHg9qicbJVgKSZoCzlfuShTiblTNOqGgFpPw7tSc9qJGEsib16QGCV8YLrV6Pk2IJzC91KWQGpiaqia6JCaCgBPmYbwTRJDsoKz/96)

  散淡样子

  

  学到了，学到了，感谢

- ![img](http://wx.qlogo.cn/mmopen/ajNVdqHZLLDxdlXkmKS5VnQibr8OOknq9ibOHVJnSxZr9hNmKYRvuM7gmyPeh4ab3VVQIhABkY5NdqXibscXia3zwA/96)

  123

  1

  

  请问那数据库会像处理自增id一样处理雪花算法的id么 还是很进行比较

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)

  1

  

  跟雪花算法ID一样，不同点地方是，雪花算法id存储空间更大些