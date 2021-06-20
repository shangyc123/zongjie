## 零散的MySQL基础总是记不住？看这一篇如何拯救你

[程序员闪充宝](javascript:void(0);) *4月3日*

![图片](https://mmbiz.qpic.cn/mmbiz_png/eukZ9J6BEiaeKZo1HxC99MeZJibns8KalXe20ibibaH585hIsxVMER52C0PWCVbibficBoZF67ianGgvdqlYuN6kuVQkA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 作者：Sicimike
>
> blog.csdn.net/Baisitao_/article/details/104714764

## 前言

在日常开发中，一些不常用且又比较基础的知识，过了一段时间之后，总是容易忘记或者变得有点模棱两可。本篇主要记录一些关于MySQL数据库比较基础的知识，以便日后快速查看。

## SQL命令

SQL命令分可以分为四组：**DDL**、**DML**、**DCL**和**TCL**。四组中包含的命令分别如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/eukZ9J6BEiafIoALjgZdtVWAEWA5Nw2mV3SPRAOXNOZ7rfpchBib4qHy6QElhVIpibibO32b5aOKhdSO3qiarNUK73Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### DDL

DDL是数据定义语言（Data Definition Language）的简称，它处理数据库schemas和描述数据应如何驻留在数据库中。

CREATE：创建数据库及其对象（如表，索引，视图，存储过程，函数和触发器） ALTER：改变现有数据库的结构 DROP：从数据库中删除对象 TRUNCATE：从表中删除所有记录，包括为记录分配的所有空间都将被删除 COMMENT：添加注释 RENAME：重命名对象

常用命令如下：

```
# 建表
CREATE TABLE sicimike  (
  id int(4) primary key auto_increment COMMENT '主键ID',
  name varchar(10) unique,
  age int(3) default 0,
  identity_card varchar(18)
  # PRIMARY KEY (id) // 也可以通过这种方式设置主键
  # UNIQUE KEY (name) // 也可以通过这种方式设置唯一键
  # key/index (identity_card, col1...) // 也可以通过这种方式创建索引
) ENGINE = InnoDB;

# 设置主键
alter table sicimike add primary key(id);

# 删除主键
alter table sicimike drop primary key;

# 设置唯一键
alter table sicimike add unique key(column_name);

# 删除唯一键
alter table sicimike drop index column_name;

# 创建索引
alter table sicimike add [unique/fulltext/spatial] index/key index_name (identity_card[(len)] [asc/desc])[using btree/hash]
create [unique/fulltext/spatial] index index_name on sicimike(identity_card[(len)] [asc/desc])[using btree/hash]
example：alter table sicimike add index idx_na(name, age);

# 删除索引
alter table sicimike drop key/index identity_card;
drop index index_name on sicimike;

# 查看索引
show index from sicimike;

# 查看列
desc sicimike;

# 新增列
alter table sicimike add column column_name varchar(30);

# 删除列
alter table sicimike drop column column_name;

# 修改列名
alter table sicimike change column_name new_name varchar(30);

# 修改列属性
alter table sicimike modify column_name varchar(22);

# 查看建表信息
show create table sicimike;

# 添加表注释
alter table sicimike comment '表注释';

# 添加字段注释
alter table sicimike modify column column_name varchar(10) comment '姓名';
————————————————
版权声明：本文为CSDN博主「Sicimike」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/Baisitao_/article/details/104714764
```

### DML

DML是**数据操纵语言**（Data Manipulation Language）的简称，包括最常见的SQL语句，例如`SELECT`，`INSERT`，`UPDATE`，`DELETE`等，它用于**存储**，**修改**，**检索**和**删除**数据库中的数据。

- 分页

  ```
  -- 查询从第11条数据开始的连续5条数据
  select * from sicimike limit 10, 5
  ```

- group by

  默认情况下，MySQL中的分组（group by）语句，不要求select返回的列，必须是分组的列或者是一个聚合函数。如果select查询的列不是分组的列，也不是聚合函数，则会返回该分组中第一条记录的数据。对比下面两条SQL语句，第二条SQL语句中，cname既不是分组的列，也不是以聚合函数的形式出现。所以在liming这个分组中，cname取的是第一条数据。

  ```
  mysql> select * from c;
  +-----+-------+----------+
  | CNO | CNAME | CTEACHER |
  +-----+-------+----------+
  |   1 | 数学  | liming   |
  |   2 | 语文  | liming   |
  |   3 | 历史  | xueyou   |
  |   4 | 物理  | guorong  |
  |   5 | 化学  | liming   |
  +-----+-------+----------+
  5 rows in set (0.00 sec)
  
  mysql> select cteacher, count(cteacher), cname from c group by cteacher;
  +----------+-----------------+-------+
  | cteacher | count(cteacher) | cname |
  +----------+-----------------+-------+
  | guorong  |               1 | 物理  |
  | liming   |               3 | 数学  |
  | xueyou   |               1 | 历史  |
  +----------+-----------------+-------+
  3 rows in set (0.00 sec)
  ————————————————
  ```

- having having关键字用于对分组后的数据进行筛选，功能相当于分组之前的where，不过要求更严格。过滤条件要么是一个聚合函数( ... having count(x) > 1)，要么是出现在select后面的列(select col1, col2 ... group by x having col1 > 1)

- 多表更新

  ```
  - update tableA a inner join tableB b on a.xxx = b.xxx set a.col1 = xxx, b.col1 = xxx where ...
  ```

- 多表删除

  ```
  - delete a, b from tableA a inner join tableB b on a.xxx = b.xxx where a.col1 = xxx and b.col1 = xxx 
  ```

### DCL

DCL是**数据控制语言**（Data Control Language）的简称，它包含诸如`GRANT`之类的命令，并且主要涉及数据库系统的权限，权限和其他控件。

- `GRANT` ：允许用户访问数据库的权限
- `REVOKE`：撤消用户使用GRANT命令赋予的访问权限

### TCL

TCL是**事务控制语言**（Transaction Control Language）的简称，用于处理数据库中的事务

- `COMMIT`：提交事务
- `ROLLBACK`：在发生任何错误的情况下回滚事务

### 范式

> 数据库规范化，又称正规化、标准化，是数据库设计的一系列原理和技术，以减少数据库中数据冗余，增进数据的一致性。关系模型的发明者埃德加·科德最早提出这一概念，并于1970年代初定义了第一范式、第二范式和第三范式的概念，还与Raymond F. Boyce于1974年共同定义了第三范式的改进范式——BC范式。除外还包括针对多值依赖的第四范式，连接依赖的第五范式、DK范式和第六范式。

现在数据库设计**最多满足3NF**，普遍认为范式过高，虽然具有对数据关系更好的约束性，但也导致数据关系表增加而令数据库IO更易繁忙，原来交由数据库处理的关系约束现更多在数据库使用程序中完成。

### 第一范式

定义：数据库中的所有**字段**（列）都是单一属性，**不可再分**的。这个单一属性由基本的数据类型所构成，如整型、浮点型、字符串等。第一范式是为了保证列的原子性。



上表不满足第一范式，其中的地址列是可以再拆分的，可以拆分成省、市、区等



### 第二范式

定义：数据库中的表不存在非关键字段对任一关键字字段的**部分函数**依赖**部分函数依赖**是指存在着**组合关键字**中的某一关键字决定非关键字的情况 第二范式在满足了第一范式的基础上，消除非主键列对**联合主键**的部分依赖



上面这张表中想要设置主键，只能是**商品名称**和**供应商名称**一起组成联合主键。但是**价格**和**分类**只依赖于**商品名称**，供应商电话只依赖于**供应商名称**，所以上面的表不满足第二范式，可以改成如下形式：商品信息表



供应商信息表



商品-供应商关联表



### 第三范式

定义：所有非主键属性都只和候选键有相关性，也就是说非主键属性之间应该是独立无关的。第三范式是在满足了第二范式的基础上，消除列与列之间的**传递依赖**。



在上面的表中，商品的**分类描述**依赖**分类**，而**分类**依赖**商品名称**，而不是**分类描述**直接依赖**商品名称**。这样就形成了**传递依赖**，所以不符合第三范式。可以改成如下形式

商品表



商品分类表



数据库设计时，遵循范式和反范式一直以来是一个颇受争议的问题。遵循范式对数据关系更好的约束性，并且减少数据冗余，可以更好地保证数据一致性。而反范式则是为了获得更好地性能。所以范式还是反范式并没有明确的标准，适合自己业务场景的才是最好的。

反范式设计时，需要考虑以下几个问题，分别是插入异常、更新异常和删除异常

- 插入异常：如果某个实体随着另一个实体的存在而存在，即缺少某个实体是无法表示这个实体，那么这个表就存在插入异常。
- 更新异常：如果更改表所对应的某个实体实例的单独属性时，需要将多行更新，那么就说明这个表存在更新异常
- 删除异常：如果删除表的某一行来表示某实体实例失效时，导致另一个不同实体实例信息丢失，那么这个表就存在删除异常

以违反第二范式的表为例



如果可乐第二制造厂这个供应商尚未开始供货，表中就不存在第二条记录，也就无法记录供应商的电话，这样就存在插入异常；如果需要把可乐的价格提高，需要更新表中的多条记录，这样就存在更新异常；如果删除可乐第二制造厂的供货信息，那么该供应商的电话也就丢失了，这样就存在删除异常。

一般存在插入异常的表，都会存在更新异常和删除异常。

## 横表纵表

SQL脚本

```
# 横表
CREATE TABLE `table_h2z` (
`name` varchar(32) DEFAULT NULL,
`chinese` int(11) DEFAULT NULL,
`math` int(11) DEFAULT NULL,
`english` int(11) DEFAULT NULL
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

/*Data for the table `table_h2z` */
insert  into `table_h2z`(`name`,`chinese`,`math`,`english`) values 
('mike',45,43,87),
('lily',53,64,88),
('lucy',57,75,75);

# 纵表
CREATE TABLE `table_z2h` (
  `name` varchar(32) DEFAULT NULL,
  `subject` varchar(8) NOT NULL DEFAULT '',
  `score` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Data for the table `table_z2h` */
insert  into `table_z2h`(`name`,`subject`,`score`) values 
('mike','chinese',45),
('lily','chinese',53),
('lucy','chinese',57),
('mike','math',43),
('lily','math',64),
('lucy','math',75),
('mike','english',87),
('lily','english',88),
('lucy','english',75);
```

### 横表转纵表

```
SELECT NAME, 'chinese' AS `subject`,  chinese AS `score` FROM table_h2z
UNION ALL
SELECT NAME, 'math' AS `subject`,  math AS `score` FROM table_h2z
UNION ALL
SELECT NAME, 'english' AS `subject`, english AS `score` FROM table_h2z
```

执行结果

```
+------+---------+-------+
| name | subject | score |
+------+---------+-------+
| mike | chinese |    45 |
| lily | chinese |    53 |
| lucy | chinese |    57 |
| mike | math    |    43 |
| lily | math    |    64 |
| lucy | math    |    75 |
| mike | english |    87 |
| lily | english |    88 |
| lucy | english |    75 |
+------+---------+-------+
9 rows in set (0.00 sec)
```

### 纵表转横表

```
SELECT NAME,
 SUM(CASE `subject` WHEN 'chinese' THEN score ELSE 0 END) AS chinese,
 SUM(CASE `subject` WHEN 'math' THEN score ELSE 0 END) AS math,
 SUM(CASE `subject` WHEN 'english' THEN score ELSE 0 END) AS english
FROM table_z2h
GROUP BY NAME
```

执行结果

```
+------+---------+------+---------+
| name | chinese | math | english |
+------+---------+------+---------+
| lily |      53 |   64 |      88 |
| lucy |      57 |   75 |      75 |
| mike |      45 |   43 |      87 |
+------+---------+------+---------+
3 rows in set (0.00 sec)
```

## 参考

- https://www.w3schools.in/mysql/ddl-dml-dcl/

```
华为面试官：为什么 HashMap 的加载因子是0.75？
趣头条面试题：ThreadLocal是什么？怎么用？为什么用它？有什么缺点
@Autowire和@Resource注解使用的正确姿势，这些年我一直用错了！！
强大：MyBatis 流式查询华为面试官：为什么 HashMap 的加载因子是0.75？
SpringBoot+OAuth2+JWT实现单点登录SSO完整教程，竟如此简单优雅！Springboot启动扩展点超详细总结，再也不怕面试官问了趣头条面试题：ThreadLocal是什么？怎么用？为什么用它？有什么缺点
点击阅读全文前往微服务电商教程
```

[阅读原文](https://mp.weixin.qq.com/s?__biz=MzIxMjU5NjEwMA==&mid=2247502470&idx=1&sn=2cf3334ccdafe9d17a13c9e02fc5eb38&key=bf0a475117e7a82e92818d54e26dcc428f352aac719865b6bdd7ee676f60485ab3980d314324ac1db28f3c2e80621670c826f6d5c6af5e7d83d675f76b8fe1d018b4fb38c4eb66a4bcd00d3c434917dc77a070335c6ad66407dc80dd4dcddec39137e92b98328652f02903b817ad67d9b7f0293092a3789aa29943864d20ceaa&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AUT6b1dLScdcDChghaip%2BEg%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2##)

阅读 2130

分享收藏

赞11在看15

写下你的留言