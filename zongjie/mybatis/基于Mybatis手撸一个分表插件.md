## 基于Mybatis手撸一个分表插件

[Java极客技术](javascript:void(0);) *5月21日*

编者荐语：

分表插件使用的频率还是很高的，那你有自己写过吗？

以下文章来源于程序猿阿星 ，作者点击关注 👉

[![程序猿阿星](http://wx.qlogo.cn/mmhead/Q3auHgzwzM5LvT4kweKOAZicXXqiaqpWNPiaS7mgNDMmCACsCNJ4p9Qrw/0)**程序猿阿星**一起成长进阶！专注技术原理、源码，通过图解方式输出技术，这里将会分享操作系统、计算机网络、Java、分布式、数据库等精品原创文章](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495851&idx=1&sn=33856bb05dd6737cd729ec2eb60ae6fa&key=2b0dcaaa74ed1decf390faace1da9a973e41820a2e133e4cd9f8488b417908ad29225dd6d04ab43d7dc67514a1b48207ffce994945864c5375a39ab088a9fc8ec2dcb16d800d479be1ad226f2dce39f58aa6c3ae5782b527bf51b1abb4cb1c484a18aab6722920244fab60816cdbd95b3108859d72a70e0a27199d995364a988&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=ASsph3Cq4MMN7%2F%2BfRJfWKwc%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2#)

大家好，我是摸鱼失败的阿星

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/23OQmC1ia8nxwNlx9sQOrrSIumMFRe8BnvRrZtdNd9OdkvHVjDSiaL22ib2xYeDd6HySiau7SZjEjBsseh2L0Micu7g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# 背景

事情是酱紫的，阿星的上级`leader`负责记录信息的业务，每日预估数据量是`15`万左右，所以引入`sharding-jdbc`做分表。

上级`leader`完成业务的开发后，走了一波自测，`git push`后，就忙其他的事情去了。

项目的框架是`SpringBoot+Mybaits`

# 出问题了

阿星负责的业务也开发完了，熟练的`git pull`，准备自测，单元测试`run`一下，上个厕所回来收工，就是这么自信。

![图片](https://mmbiz.qpic.cn/mmbiz_gif/23OQmC1ia8nxwNlx9sQOrrSIumMFRe8BnjkyawdA7nB9IWswZ4fWbmhyWkUOkX8q781snMRAMAWniaZEudv7EnrQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

回来后，看下控制台，人都傻了，一片红，内心不禁感叹“**如果这是股票基金该多好**”。

出了问题就要解决，随着排查深入，我的眉头一皱发现事情并不简单，怎么以前的一些代码都报错了？

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/23OQmC1ia8nxwNlx9sQOrrSIumMFRe8BncmuZMiaAHvDbq5EVCjSPHAibESfExw3ticGvGlGvEKVJWtBVXa1kNlbfw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

随着排查深入，最后跟到了`Mybatis`源码，发现罪魁祸首是`sharding-jdbc`引起的，因为数据源是`sharding-jdbc`的，导致后续执行`sql`的是`ShardingPreparedStatement`。

这就意味着，`sharding-jdbc`影响项目的所有业务表，因为最终数据库交互都由`ShardingPreparedStatement`去做了，历史的一些`sql`语句因为`sql`函数或者其他写法，使得`ShardingPreparedStatement`无法处理而出现异常。

关键代码如下

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nxwNlx9sQOrrSIumMFRe8Bnh9TWQmHq6ic9fibYl9Sbld4jfMf9icDkMFe7KkU2L2XU61tOYp7JibO4hA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

发现问题后，阿星马上就反馈给`leader`了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/23OQmC1ia8nxwNlx9sQOrrSIumMFRe8Bn8nNtGSLQcrQI6r4YgibIz0SkA9Z1gRxcGtyMLcgAlKQ03ibc2kYBR89w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

唉，本来还想摸鱼的，看来摸鱼的时间是没了，还多了一项任务。

# 分析

竟然交给阿星来做了，就撸起袖子开干吧，先看看分表功能的需求

- 支持自定义分表策略
- 能控制影响范围
- 通用性

分表会提前建立好，所以不需要考虑表不存在的问题，核心逻辑实现，通过分表策略得到分表名，再把分表名动态替换到`sql`。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 分表策略

为了支持分表策略，我们需要先定义分表策略抽象接口，定义如下

```
/**
 * @Author 程序猿阿星
 * @Description 分表策略接口
 * @Date 2021/5/9
 */
public interface ITableShardStrategy {


    /**
     * @author: 程序猿阿星
     * @description: 生成分表名
     * @param tableNamePrefix 表前缀名
     * @param value 值
     * @date: 2021/5/9
     * @return: java.lang.String
     */
    String generateTableName(String tableNamePrefix,Object value);

    /**
     * 验证tableNamePrefix
     */
    default void verificationTableNamePrefix(String tableNamePrefix){
        if (StrUtil.isBlank(tableNamePrefix)) {
            throw new RuntimeException("tableNamePrefix is null");
        }
    }
}
```

`generateTableName`函数的任务就是生成分表名，入参有`tableNamePrefix、value`，`tableNamePrefix`为分表前缀，`value`作为生成分表名的逻辑参数。

`verificationTableNamePrefix`函数验证`tableNamePrefix`必填，提供给实现类使用。

为了方便理解，下面是`id`取模策略代码，取模两张表

```
/**
 * @Author 程序猿阿星
 * @Description 分表策略id
 * @Date 2021/5/9
 */
@Component
public class TableShardStrategyId implements ITableShardStrategy {
    @Override
    public String generateTableName(String tableNamePrefix, Object value) {
        verificationTableNamePrefix(tableNamePrefix);
        if (value == null || StrUtil.isBlank(value.toString())) {
            throw new RuntimeException("value is null");
        }
        long id = Long.parseLong(value.toString());
        //此处可以缓存优化
        return tableNamePrefix + "_" + (id % 2);
    }
}
```

传入进来的`value`是`id`值，用`tableNamePrefix`拼接`id`取模后的值，得到分表名返回。

## 控制影响范围

分表策略已经抽象出来，下面要考虑控制影响范围，我们都知道`Mybatis`规范中每个`Mapper`类对应一张业务主体表，`Mapper`类的函数对应业务主体表的相关`sql`。

阿星想着，可以给`Mapper`类打上注解，代表该`Mpaaer`类对应的业务主体表有分表需求，从规范来说`Mapper`类的每个函数对应的主体表都是正确的，但是有些同学可能不会按规范来写。

假设`Mpaaer`类对应的是`B`表，`Mpaaer`类的某个函数写着`A`表的`sql`，甚至是历史遗留问题，所以注解不仅仅可以打在`Mapper`类上，同时还可以打在`Mapper`类的任意一个函数上，并且保证小粒度覆盖粗粒度。

阿星这里自定义分表注解，代码如下

```
/**
 * @Author 程序猿阿星
 * @Description 分表注解
 * @Date 2021/5/9
 */
@Target(value = {ElementType.TYPE,ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface TableShard {

    // 表前缀名
    String tableNamePrefix();

    //值
    String value() default "";

    //是否是字段名，如果是需要解析请求参数改字段名的值（默认否）
    boolean fieldFlag() default false;

    // 对应的分表策略类
    Class<? extends ITableShardStrategy> shardStrategy();


}
```



注解的作用范围是类、接口、函数，运行时生效。

`tableNamePrefix`与`shardStrategy`属性都好理解，表前缀名和分表策略，剩下的`value`与`fieldFlag`要怎么理解，分表策略分两类，第一类依赖表中某个字段值，第二类则不依赖。

根据企业`id`取模，属于第一类，此处的`value`设置企业`id`入参字段名，`fieldFlag`为`true`，意味着，会去解析获取企业`id`字段名对应的值。

根据日期分表，属于第二类，直接在分表策略实现类里面写就行了，不依赖表字段值，`value`与`fieldFlag`无需填写，当然你`value`也可以设置时间格式，具体看分表策略实现类的逻辑。

## 通用性

抽象分表策略与分表注解都搞定了，最后一步就是根据分表注解信息，去执行分表策略得到分表名，再把分表名动态替换到`sql`中，同时具有通用性。

`Mybatis`框架中，有拦截器机制做扩展，我们只需要拦截`StatementHandler#prepare`函数，即`StatementHandle`创建`Statement`之前，先把`sql`里面的表名动态替换成分表名。

`Mybatis`分表拦截器流程图如下

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

`Mybatis`分表拦截器代码如下，有点长哈，主流程看`intercept`函数就好了。

```
/**
 * @Author 程序员阿星
 * @Description 分表拦截器
 * @Date 2021/5/9
 */
@Intercepts({
        @Signature(
                type = StatementHandler.class,
                method = "prepare",
                args = {Connection.class, Integer.class}
        )
})
public class TableShardInterceptor implements Interceptor {

    private static final ReflectorFactory defaultReflectorFactory = new DefaultReflectorFactory();

    @Override
    public Object intercept(Invocation invocation) throws Throwable {

        // MetaObject是mybatis里面提供的一个工具类，类似反射的效果
        MetaObject metaObject = getMetaObject(invocation);
        BoundSql boundSql = (BoundSql) metaObject.getValue("delegate.boundSql");
        MappedStatement mappedStatement = (MappedStatement)
                metaObject.getValue("delegate.mappedStatement");

        //获取Mapper执行方法
        Method method = invocation.getMethod();

        //获取分表注解
        TableShard tableShard = getTableShard(method,mappedStatement);

        // 如果method与class都没有TableShard注解或执行方法不存在，执行下一个插件逻辑
        if (tableShard == null) {
            return invocation.proceed();
        }

        //获取值
        String value = tableShard.value();
        //value是否字段名，如果是，需要解析请求参数字段名的值
        boolean fieldFlag = tableShard.fieldFlag();

        if (fieldFlag) {
            //获取请求参数
            Object parameterObject = boundSql.getParameterObject();

            if (parameterObject instanceof MapperMethod.ParamMap) { //ParamMap类型逻辑处理

                MapperMethod.ParamMap parameterMap = (MapperMethod.ParamMap) parameterObject;
                //根据字段名获取参数值
                Object valueObject = parameterMap.get(value);
                if (valueObject == null) {
                    throw new RuntimeException(String.format("入参字段%s无匹配", value));
                }
                //替换sql
                replaceSql(tableShard, valueObject, metaObject, boundSql);

            } else { //单参数逻辑

                //如果是基础类型抛出异常
                if (isBaseType(parameterObject)) {
                    throw new RuntimeException("单参数非法，请使用@Param注解");
                }

                if (parameterObject instanceof Map){
                    Map<String,Object>  parameterMap =  (Map<String,Object>)parameterObject;
                    Object valueObject = parameterMap.get(value);
                    //替换sql
                    replaceSql(tableShard, valueObject, metaObject, boundSql);
                } else {
                    //非基础类型对象
                    Class<?> parameterObjectClass = parameterObject.getClass();
                    Field declaredField = parameterObjectClass.getDeclaredField(value);
                    declaredField.setAccessible(true);
                    Object valueObject = declaredField.get(parameterObject);
                    //替换sql
                    replaceSql(tableShard, valueObject, metaObject, boundSql);
                }
            }

        } else {//无需处理parameterField
            //替换sql
            replaceSql(tableShard, value, metaObject, boundSql);
        }
        //执行下一个插件逻辑
        return invocation.proceed();
    }


    @Override
    public Object plugin(Object target) {
        // 当目标类是StatementHandler类型时，才包装目标类，否者直接返回目标本身, 减少目标被代理的次数
        if (target instanceof StatementHandler) {
            return Plugin.wrap(target, this);
        } else {
            return target;
        }
    }


    /**
     * @param object
     * @methodName: isBaseType
     * @author: 程序员阿星
     * @description: 基本数据类型验证，true是，false否
     * @date: 2021/5/9
     * @return: boolean
     */
    private boolean isBaseType(Object object) {
        if (object.getClass().isPrimitive()
                || object instanceof String
                || object instanceof Integer
                || object instanceof Double
                || object instanceof Float
                || object instanceof Long
                || object instanceof Boolean
                || object instanceof Byte
                || object instanceof Short) {
            return true;
        } else {
            return false;
        }
    }

    /**
     * @param tableShard 分表注解
     * @param value      值
     * @param metaObject mybatis反射对象
     * @param boundSql   sql信息对象
     * @author: 程序猿阿星
     * @description: 替换sql
     * @date: 2021/5/9
     * @return: void
     */
    private void replaceSql(TableShard tableShard, Object value, MetaObject metaObject, BoundSql boundSql) {
        String tableNamePrefix = tableShard.tableNamePrefix();
        //获取策略class
        Class<? extends ITableShardStrategy> strategyClazz = tableShard.shardStrategy();
        //从spring ioc容器获取策略类

        ITableShardStrategy tableShardStrategy = SpringUtil.getBean(strategyClazz);
        //生成分表名
        String shardTableName = tableShardStrategy.generateTableName(tableNamePrefix, value);
        // 获取sql
        String sql = boundSql.getSql();
        // 完成表名替换
        metaObject.setValue("delegate.boundSql.sql", sql.replaceAll(tableNamePrefix, shardTableName));
    }

    /**
     * @param invocation
     * @author: 程序猿阿星
     * @description: 获取MetaObject对象-mybatis里面提供的一个工具类，类似反射的效果
     * @date: 2021/5/9
     * @return: org.apache.ibatis.reflection.MetaObject
     */
    private MetaObject getMetaObject(Invocation invocation) {
        StatementHandler statementHandler = (StatementHandler) invocation.getTarget();
        // MetaObject是mybatis里面提供的一个工具类，类似反射的效果
        MetaObject metaObject = MetaObject.forObject(statementHandler,
                SystemMetaObject.DEFAULT_OBJECT_FACTORY,
                SystemMetaObject.DEFAULT_OBJECT_WRAPPER_FACTORY,
                defaultReflectorFactory
        );

        return metaObject;
    }

    /**
     * @author: 程序猿阿星
     * @description: 获取分表注解
     * @param method
     * @param mappedStatement
     * @date: 2021/5/9
     * @return: com.xing.shard.interceptor.TableShard
     */
    private TableShard getTableShard(Method method, MappedStatement mappedStatement) throws ClassNotFoundException {
        String id = mappedStatement.getId();
        //获取Class
        final String className = id.substring(0, id.lastIndexOf("."));
        //分表注解
        TableShard tableShard = null;
        //获取Mapper执行方法的TableShard注解
        tableShard = method.getAnnotation(TableShard.class);
        //如果方法没有设置注解，从Mapper接口上面获取TableShard注解
        if (tableShard == null) {
            // 获取TableShard注解
            tableShard = Class.forName(className).getAnnotation(TableShard.class);
        }
        return tableShard;
    }

}
```

到了这里，其实分表功能就已经完成了，我们只需要把**分表策略抽象接口、分表注解、分表拦截器**抽成一个通用`jar`包，需要使用的项目引入这个`jar`，然后注册分表拦截器，自己根据业务需求实现分表策略，在给对应的`Mpaaer`加上分表注解就好了。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

# 实践跑起来

这里阿星单独写了一套`demo`，场景是有两个分表策略，表也提前建立好了

- 根据`id`分表

- - `tb_log_id_0`
  - `tb_log_id_1`

- 根据日期分表

- - `tb_log_date_202105`
  - `tb_log_date_202106`

**预警：后面都是代码实操环节，请各位读者大大耐心看完（非Java开发除外）**。

## TableShardStrategy定义

```
/**
 * @Author wx
 * @Description 分表策略日期
 * @Date 2021/5/9
 */
@Component
public class TableShardStrategyDate implements ITableShardStrategy {

    private static final String DATE_PATTERN = "yyyyMM";

    @Override
    public String generateTableName(String tableNamePrefix, Object value) {
        verificationTableNamePrefix(tableNamePrefix);
        if (value == null || StrUtil.isBlank(value.toString())) {
            return tableNamePrefix + "_" +DateUtil.format(new Date(), DATE_PATTERN);
        } else {
            return tableNamePrefix + "_" +DateUtil.format(new Date(), value.toString());
        }
    }
}



**
 * @Author 程序猿阿星
 * @Description 分表策略id
 * @Date 2021/5/9
 */
@Component
public class TableShardStrategyId implements ITableShardStrategy {
    @Override
    public String generateTableName(String tableNamePrefix, Object value) {
        verificationTableNamePrefix(tableNamePrefix);
        if (value == null || StrUtil.isBlank(value.toString())) {
            throw new RuntimeException("value is null");
        }
        long id = Long.parseLong(value.toString());
        //可以加入本地缓存优化
        return tableNamePrefix + "_" + (id % 2);
    }
}
```

## Mapper定义

Mapper接口

```
/**
 * @Author 程序猿阿星
 * @Description
 * @Date 2021/5/8
 */
@TableShard(tableNamePrefix = "tb_log_date",shardStrategy = TableShardStrategyDate.class)
public interface LogDateMapper {

    /**
     * 查询列表-根据日期分表
     */
    List<LogDate> queryList();

    /**
     * 单插入-根据日期分表
     */
    void  save(LogDate logDate);

}


-------------------------------------------------------------------------------------------------


/**
 * @Author 程序猿阿星
 * @Description
 * @Date 2021/5/8
 */
@TableShard(tableNamePrefix = "tb_log_id",value = "id",fieldFlag = true,shardStrategy = TableShardStrategyId.class)
public interface LogIdMapper {

    /**
     * 根据id查询-根据id分片
     */
    LogId queryOne(@Param("id") long id);

    /**
     * 单插入-根据id分片
     */
    void save(LogId logId);


}
```

------

Mapper.xml

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xing.shard.mapper.LogDateMapper">
    
    //对应LogDateMapper#queryList函数
    <select id="queryList" resultType="com.xing.shard.entity.LogDate">
        select
        id as id,
        comment as comment,
        create_date as createDate
        from
        tb_log_date
    </select>
    
    //对应LogDateMapper#save函数
    <insert id="save" >
        insert into tb_log_date(id, comment,create_date)
        values (#{id}, #{comment},#{createDate})
    </insert>
</mapper>

-------------------------------------------------------------------------------------------------

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xing.shard.mapper.LogIdMapper">
    
    //对应LogIdMapper#queryOne函数
    <select id="queryOne" resultType="com.xing.shard.entity.LogId">
        select
        id as id,
        comment as comment,
        create_date as createDate
        from
        tb_log_id
        where
        id = #{id}
    </select>
    
    //对应save函数
    <insert id="save" >
        insert into tb_log_id(id, comment,create_date)
        values (#{id}, #{comment},#{createDate})
    </insert>

</mapper>
```

## 执行下单元测试

日期分表单元测试执行

```
    @Test
    void test() {
        LogDate logDate = new LogDate();
        logDate.setId(snowflake.nextId());
        logDate.setComment("测试内容");
        logDate.setCreateDate(new Date());
        //插入
        logDateMapper.save(logDate);
        //查询
        List<LogDate> logDates = logDateMapper.queryList();
        System.out.println(JSONUtil.toJsonPrettyStr(logDates));
    }
```

输出结果

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

------

`id`分表单元测试执行

```
    @Test
    void test() {
        LogId logId = new LogId();
        long id = snowflake.nextId();
        logId.setId(id);
        logId.setComment("测试");
        logId.setCreateDate(new Date());
        //插入
        logIdMapper.save(logId);
        //查询
        LogId logIdObject = logIdMapper.queryOne(id);
        System.out.println(JSONUtil.toJsonPrettyStr(logIdObject));
    }
```

输出结果

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

# 小结一下

本文可以当做对`Mybatis`进阶的使用教程，通过`Mybatis`拦截器实现分表的功能，满足基本的业务需求，虽然比较简陋，但是`Mybatis`这种扩展机制与设计值得学习思考。

阅读 1468

分享收藏

赞10在看3

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/PiajxSqBRaELfkq0pEyjQ6a1jia2cln52yXSYMAWWEVozgXaasXoUMqicknspa4VR4tKSR908Hx0tYOtibCwFaRfTQ/96)

  灿若千阳

  

  日期分页查询会如何执行呢

- ![img](http://wx.qlogo.cn/mmopen/Q3auHgzwzM5ia6bibaaH86CaQ0Qv4gJScKxbCHx3T68pekTJNlIMOhY3eSLkoO8OoGqVmXOicK7sSF5o2Liaubia44w/96)

  阿星

  

  谢谢大佬转载