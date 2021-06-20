## 【真实案例】盘点分库分表中件间Mycat中的坑

原创 鸭血粉丝 [Java极客技术](javascript:void(0);) *5月20日*

```
每天早上七点三十，准时推送干货
```

### 一、介绍

公司最近在搞服务分离，数据切分的工作，因为订单和订单项表的数据量实在过大，而且每天都是以50万的数据量在增长，基于现状，项目组决定采用分库的方式来解决当前遇到的问题。

那具体怎么切分呢？

分库的策略其实还比较简单，主要是要确定**分片的字段和策略**。

最开始是想通过主键ID的奇、偶数来分两个库，`order_1`库主要用于存储奇数的ID，`order_2`库主要用于存储偶数的ID。

但是这种切分，局限性非常大，因为最多只能分两个库，如果随着数据量的增大，后面就没很难在分了。

之后又想到了另一个分片字段：`城市ID`，因为订单表上有`城市ID`的属性，我们可以基于此进行分库，但是全国有几百个城市，不可能分几百个库或者表，最后的讨论结果是：

- 城市ID的生成固定大小，默认三位数，100～999
- 将订单表分成三个库，`order_1`、`order_2`、`order_3`
- 当城市ID 在`100~399`区间，就存储到`order_1`库
- 当城市ID 在`400~699`区间，就存储到`order_2`库
- 当城市ID 在`700～999`区间，就存储到`order_3`库

通过`城市ID`进行分片，如果后期订单数据量进一步过大，也可以进一步的分库！

基于`Mysql`数据库，使用最广、最成熟的分布式中间件当属于`Mycat`。

但是，自从采用`Mycat`中间件进行分库之后，发现了非常多的坑，下面我们就一起来看看这些坑点！

### 二、细数Mycat中的坑点

#### 2.1、分页查询会出现全表扫描

当我们把功能上线之后，测试人员在页面上从末尾页不停的往前分页查询订单数据的时候，运维平台突然报监控到很多慢 SQL 报警。

以下是运维平台监控到的慢sql语句。

```
SELECT id FROM order
WHERE OrderCreateTime BETWEEN '2021-05-01 00:00:00' AND '2021-06-01 00:00:00'
ORDER BY id DESC
LIMIT 0, 151400
```

于是，运维同学开始找到我们，说我们程序有问题，并在群里开始吐槽我们开发写的啥玩意，但是我们开发坚信程序没有问题，通过查询日志，我们排查到代码的查询语句是长这样的。

```
SELECT id FROM order
WHERE OrderCreateTime BETWEEN '2021-05-01 00:00:00' AND '2021-06-01 00:00:00'
ORDER BY id DESC
LIMIT 151300, 100
```

与实际运维给的慢sql语句中的`LIMIT 0, 151400`完全不符合。

包括我们自己也 review 了代码，把 sql日志也截了图，找技术总监说理去。

之后，当测试人员再次点击分页查询的时候，运维又监控到了`LIMIT 0, 151400`这种怪异的`SQL`，我们花了好几个小时排查，在本地跑测试，还是没发现什么问题，真的感觉到了要怀疑人生了！

当多次测试的时候，这个问题每次都能复现，让我想起了一个问题，是不是 Mycat 分页的时候，对全表扫描了。

后来经过查阅资料，才发现真有这个坑！

在分库分表的情况下，宕 limit 的开始位置特别大的时候，例如大于某表的总行数时，mycat 将查询各个分表的结果集返，然后在mycat中进行合并和排序，再返回结果。

例如，当你原始的 sql 语句是这样的：

```
SELECT * FROM table_name WHERE type='xxx' ORDER BY create_time LIMIT 10000,1000
```

通过 mycat 执行的结果，会是这样的：

```
SELECT * FROM table_name WHERE type='xxx' ORDER BY create_time LIMIT 0,11000
```

**结果集特别大的情况会导致查询很慢，严重的情况会直接导致 mycat OOM**！

因此，在分库分表的情况，不要用 mycat 进行大批量的数据分页查询，通过条件过滤，减小分页的数据量大小！

#### 2.2、子查询结果偶尔不完整

当通过某些条件，筛选订单项数据时，测试人员反馈某些数据偶尔出现不完整。

具体SQL操作如下：

```
select id,productName
from orderItem
where orderId in (
 select id from order where userName = '张三'
)
```

预期的查询结果时：

```
1,"巧克力"
2,"可乐"
3,"果冻"
4,"苹果手机"
```

但是实际查询的时候，有时候的结果如下：

```
1,"巧克力"
2,"可乐"
4,"苹果手机"
```

在网上查询了相关的问题，在分库分表的情况下，子查询出了偶尔查询不到完整数据外，还会出现 mycat 内部死锁，因此尽量在代码中不要使用子查询，而是采用主键ID或是索引字段进行单表查询，这样效率会大大提升！

#### 2.3、跨分片join问题

由于历史代码的缘故，订单服务里面存在很多各种连表操作，例如：

```
select a.*,b.accountName,c.address
from order a
left join account b on a.accountId = b.id
left join account_address c on  b.id = c.accountId
where a.orderId = 11110011
```

但是在走 mycat 查询之后，直接报错！

原因是：mycat 目前只支持两张分片表的 Join，如果要支持多张表需要自己改造程序代码或者改造Mycat的源代码。

#### 2.4、部分SQL语法不支持

在实际使用的时候，发现还有部分sql语句是不支持的。

复制插入(不支持)

```
insert into......select.....
```

复杂更新(不支持)

```
update a, b set a.remark='备注' where a.id=b.id;
```

复杂删除(不支持)

```
delete a from a join b on a.id=b.id;
```

还有就是不支持跨库连表操作！

#### 2.5、不支持存储过程创建和调用

有一点，需要大家注意的，在走 mycat 中间件的方式与数据库连接的时候，如果代码中写了存储过程等语句，是 mycat 是不支持调用的，因此尽量不要使用！

### 三、小结

虽然上面介绍了 mycat 有一些坑，但是这些坑，通过一些优化手段还是可以避免的。

实际上，mycat 作为分库分表的中间件，也有许多的优势，例如下面官网的介绍。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

据了解，mycat 是目前最成熟、使用最广的中间件，因此大家在使用的时候，不需要带有啥顾虑，对于以上的坑点，尽可能的避免。

在后面的文章，小编也会给大家详细的介绍 mycat 实践操作，本位如果有理解不对的地方，欢迎指出！



![img](https://mmbiz.qlogo.cn/mmbiz_jpg/fPADUR9p6acau4uticGBhaiaeH6q8T7g9Zw7BHRDpx9B7BdKKgWZvWOyJRWL0fTZyfJwiaAH4pFKtlb7Cv8vovcCA/0?wx_fmt=jpeg)

鸭血粉丝

原创不易，众筹一碗鸭血粉丝汤

![赞赏二维码](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495849&idx=2&sn=2f465b3e3a0527a6b726c83098804057&key=b15bee76b5b937f975ad20c50d5eae48189cf58045b8525656b6e456e3703f41c68450de5aa8cb6733dfbfc0a544a666c8ce2048f6dc414fbf8c2a64228158d2fc6270c9077bb85aeea72d7b1b3508a4e7632cceac4ba10bf777bf69828dc675c893deabb37d8c9acc8d4512e3a78dc11e2c8bf03a57c27c38aa53318a6753ab&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AZUCuGUSDvVttSqjE1Kwems%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2)[喜欢作者](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495849&idx=2&sn=2f465b3e3a0527a6b726c83098804057&key=b15bee76b5b937f975ad20c50d5eae48189cf58045b8525656b6e456e3703f41c68450de5aa8cb6733dfbfc0a544a666c8ce2048f6dc414fbf8c2a64228158d2fc6270c9077bb85aeea72d7b1b3508a4e7632cceac4ba10bf777bf69828dc675c893deabb37d8c9acc8d4512e3a78dc11e2c8bf03a57c27c38aa53318a6753ab&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AZUCuGUSDvVttSqjE1Kwems%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2##)

阅读 786

分享收藏

赞3在看4

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/3Lqm1xHojtaqCUNQL6icnI9ffib291oWAwPTQW4dPULISaKAeVN4xyPx4PiaqPhUk3mFQ0S7vhMAGqqnuR7cGcPNwAJ2vhCQAdb/96)

  张自强

  2

  

  其实还有一个坑，就是在建表的时候，如果你的表的前缀是01-10这样是没有问题的，但是如果你的表的前缀是1-10就会出现String.format 相关的报错

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)

  1

  

  顶一个

- ![img](http://wx.qlogo.cn/mmopen/FHiaziacrdwPl3Qeek6pSLW8LlueiczemBS065eAzEqk0OXKP6tIFJoJJAAOmBBbrqeHLms5QAahT26y2oLpE6armz0B1QHT0mR/96)

  艺超

  

  第一个有排序加分页的情况，都得是将数据全部获取，再一起排序。和limit大小无关，只是越大每个库需要查询的数据就会越多。