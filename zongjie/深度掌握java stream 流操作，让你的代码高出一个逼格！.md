## 深度掌握java stream 流操作，让你的代码高出一个逼格！

原创 鸭血粉丝 [Java极客技术](javascript:void(0);) *4月15日*

每天早上七点三十，准时推送干货

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tWOhQMr1wdB2hfAtDnzgGbiat5DXpv7jG9vHvlBIpHrImvCvpLGvyemHeC95oevYfp6lCNzrnVgXRO5uMOUlib2Q/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##  

### 一、介绍

我们都知道，从 Java8 开始，jdk 新增加了一个 Stream 类，用来补充集合类，它的强大，相信用过它的朋友，能明显的感受到，不用使用for循环就能对集合作出很好的操作。

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。

这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。

元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

操作流程如下：

```
+--------------------+       +------+   +------+   +---+   +-------+
| stream of elements +-----> |filter+-> |sorted+-> |map+-> |collect|
+--------------------+       +------+   +------+   +---+   +-------+
```

采用 Stream API 可以极大提高 Java 程序员的生产力，让程序员写出高效率、干净、简洁的代码。

下面，我们就以实际的日常开发编程风格做对比，学习完 Stream 的编程风格之后，我敢保证，你会爱上它！

### 二、遍历操作

#### 2.1、遍历集合

日常开发中，我们经常需要需要遍历集合对象中的元素，例如，我们会采用如下方式进行遍历元素，然后过滤出某个字段的集合，jdk7 的操作：

```
/**
 * jdk7 从集合对象中获取用户ID集合
 * @param userList
 * @return
 */
public List<Long> getUserIds(List<User> userList){
    List<Long> userIds = new ArrayList<>();
    for (User user : userList) {
        userIds.add(user.getUserId());
    }
    return userIds;
}
```

当采用 Stream 编程之后，只需要通过一行代码，即可实现：

```
/**
 * jdk8 从集合对象中获取用户ID集合
 * @param userList
 * @return
 */
public List<Long> getUserIds(List<User> userList){
    List<Long> userIds = userList.stream().map(User::getUserId).collect(Collectors.toList());
    return userIds;
}
```

#### 2.2、筛选元素

筛选元素，是日常开发中经常会碰到，例如在 jdk7，我们会这么操作：

```
/**
 * jdk7 从集合对象中过滤出用户ID不为空的数据
 * @param userList
 * @return
 */
public List<Long> getUserIds7(List<User> userList){
    List<Long> userIds = new ArrayList<>();
    for (User user : userList) {
        if(user.getUserId() != null){
            userIds.add(user.getUserId());
        }
    }
    return userIds;
}
```

采用 Stream api，我们只需要通过`filter`方法来筛选出需要的数据，即可过滤出用户ID不为空的数据。

```
/**
 * jdk8 从集合对象中筛选出用户ID不为空的数据
 * @param userList
 * @return
 */
public List<Long> getUserIds8(List<User> userList){
    List<Long> userIds = userList.stream().filter(item -> item.getUserId() != null).map(User::getUserId).collect(Collectors.toList());
    return userIds;
}
```

#### 2.3、删除重复的内容

如果你想对返回的集合内容排除重复的数据，操作也很简单，在合并的时候使用`Collectors.toSet()`即可！

```
/**
 * jdk8 从集合对象中筛选出用户ID不为空的数据，并进行去重
 * @param userList
 * @return
 */
public Set<Long> getUserIds(List<User> userList){
    Set<Long> userIds = userList.stream().filter(item -> item.getUserId() != null).map(User::getUserId).collect(Collectors.toSet());
    return userIds;
}
```

#### 2.4、数据类型转换

在实际的开发过程中，经常会出现数据类型定义不一致的问题，例如有的系统，使用`String`接受，有的是用`Long`，对于这种场景，我们需要将其转换，操作也很简单

```
/**
 * jdk8 将Long类型数据转换成String类型
 * @param userIds
 * @return
 */
public List<String> getUserIds10(List<Long> userIds){
    List<String> userIdStrs = userIds.stream().map(x -> x.toString()).collect(Collectors.toList());
    return userIdStrs;
}
```

#### 2.5、数组转集合

我们还会碰到，前端传给我们的是一个数组，但是我们需要转成集合，采用 stream api 操作也很简单！

```
public static void main(String[] args) {
        //创建一个字符串数组
        String[] strArray = new String[]{"a","b","c"};
        //转换后的List 属于 java.util.ArrayList 能进行正常的增删查操作
        List<String> strList = Stream.of(strArray).collect(Collectors.toList());
}
```

### 三、集合转Map操作

在实际的开发过程中，还有一个使用最频繁的操作就是，将集合元素中某个主键字段作为`key`，元素作为`value`，来实现集合转map的需求，这种需求在数据组装方面使用的非常多，**尤其是在禁止连表 sql 查询操作的公司，视图数据的拼装只能在代码层面来实现**。

例如下面这段代码，角色表里面关联角色组ID信息，当查询角色信息的时候，需要把角色组名称也展示处理，采用`map`方式来匹配，效率会非常高。

实际代码案例分享

```
//角色组ID集合
Set<Long> roleGroupIds = new HashSet<>();
//查询所有的角色信息
List<RoleInfo> dbList = roleInfoMapper.findByPage(request);
for (RoleInfo source : dbList) {
    roleGroupIds.add(source.getRoleGroupId());
    RoleInfoDto result = new RoleInfoDto();
    BeanUtils.copyProperties(source, result);
    resultList.add(result);
}
//查询角色组信息
if (CollectionUtils.isNotEmpty(roleGroupIds)) {
    List<RoleGroupInfo> roleGroupInfoList = roleGroupInfoMapper.selectByIds(new ArrayList<>(roleGroupIds));
    if (CollectionUtils.isNotEmpty(roleGroupInfoList)) {
     //将List转换成Map，其中id主键作为key，对象作为value
        Map<Long, RoleGroupInfo> sourceMap = new HashMap<>();
        for (RoleGroupInfo roleGroupInfo : roleGroupInfoList) {
            sourceMap.put(roleGroupInfo.getId(), roleGroupInfo);
        }
  //封装角色组名称
        for (RoleInfoDto result : resultList) {
            if (sourceMap.containsKey(result.getRoleGroupId())) {
                result.setRoleGroupName(sourceMap.get(result.getRoleGroupId()).getName());
            }
        }
    }
}
```

#### 3.1、集合转 map（不分组）

在 jdk7 中，将集合中的元素转 map，我们通常会采用如下方式。

```
/**
 * jdk7 将集合转换成Map，其中用户ID作为主键key
 * @param userList
 * @return
 */
public Map<Long, User> getMap(List<User> userList){
    Map<Long, User> userMap = new HashMap<>();
    for (User user : userList) {
        userMap.put(user.getUserId(), user);
    }
    return userMap;
}
```

在 jdk8 中，采用 stream api的方式，我们只需要一行代码即可实现

```
/**
 * jdk8 将集合转换成Map，其中用户ID作为主键key,如果集合对象有重复的key,以第一个匹配到的为主
 * @param userList
 * @return
 */
public Map<Long, User> getMap(List<User> userList){
    Map<Long, User> userMap = userList.stream().collect(Collectors.toMap(User::getUserId, v -> v, (k1,k2) -> k1));
    return userMap;
}
```

打开`Collectors.toMap`方法源码，一起来看看到底是啥。

```
public static <T, K, U>
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper,
                                BinaryOperator<U> mergeFunction) {
    return toMap(keyMapper, valueMapper, mergeFunction, HashMap::new);
}
```

从参数表可以看出：

- 第一个参数：表示 key
- 第二个参数：表示 value
- 第三个参数：表示某种规则

上文中的`Collectors.toMap(User::getUserId, v -> v, (k1,k2) -> k1)`，表达的意思就是将`userId`的内容作为`key`，`v -> v`是表示将元素`user`作为`value`，其中`(k1,k2) -> k1`表示如果存在相同的`key`，将第一个匹配的元素作为内容，第二个舍弃！

#### 3.2、集合转map（分组）

在实际的操作中，有一些场景需要我们将相同的key，加入到一个集合，而不是覆盖，哪改如何做呢？

如果是采用 jdk7，我们大概会这么做。

```
/**
 * jdk7 将集合转换成Map，将相同的key，加入到一个集合中，实现分组
 * @param userList
 * @return
 */
public Map<Long, List<User>> getMapGroup(List<User> userList){
    Map<Long, List<User>> userListMap = new HashMap<>();
    for (User user : userList) {
        if(userListMap.containsKey(user.getUserId())){
            userListMap.get(user.getUserId()).add(user);
        } else {
            List<User> users = new ArrayList<>();
            users.add(user);
            userListMap.put(user.getUserId(), users);
        }
    }
    return userListMap;
}
```

而在 jdk8 中，采用 stream api的方式，我们只需要一行代码即可实现

```
/**
 * jdk8 将集合转换成Map，将相同的key，加入到一个集合中，实现分组
 * @param userList
 * @return
 */
public Map<Long, List<User>> getMapGroup(List<User> userList){
    Map<Long, List<User>> userMap = userList.stream().collect(Collectors.groupingBy(User::getUserId));
    return userMap;
}
```

### 四、分页操作

stream api 的强大之处还不仅仅是对集合进行各种组合操作，还支持分页操作。

例如，将如下的数组从小到大进行排序，排序完成之后，从第1行开始，查询10条数据出来，操作如下：

```
//需要查询的数据
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5,10, 6, 20, 30, 40, 50, 60, 100);
List<Integer> dataList= numbers.stream().sorted((x, y) -> x.compareTo(y)).skip(0).limit(10).collect(Collectors.toList());
System.out.println(dataList.toString());
```

其中`skip`参数表示第几行，`limit`表示查询的数量，类似页容量！

### 五、查找与匹配操作

stream api 还支持对集合进行查找，同时还支持正则匹配模式。

- allMatch（检查是否匹配所有元素）

```
List<Integer> list = Arrays.asList(10, 5, 7, 3);
boolean allMatch = list.stream()//
        .allMatch(x -> x > 2);//是否全部元素都大于2
System.out.println(allMatch);
```

- findFirst（返回第一个元素）

```
List<Integer> list = Arrays.asList(10, 5, 7, 3);
Optional<Integer> first = list.stream()//
        .findFirst();
Integer val = first.get();
System.out.println(val);//输出10
```

- reduce（可以将流中元素反复结合起来，得到一个值）

```
List<Integer> list = Arrays.asList(10, 5, 7, 3);
Integer result = list.stream()//
    .reduce(2, Integer::sum);
System.out.println(result);//输出27，其实相当于2+10+5+7+3，就是一个累加
```

stream api 支持的操作方法非常多，这里只列举了几种类型，具体在使用的时候，可以参考官网接口文档说明！

### 六、并行操作

**所谓并行，指的是多个任务在同一时间点发生，并由不同的cpu进行处理，不互相抢占资源；而并发，指的是多个任务在同一时间点内同时发生了，但由同一个cpu进行处理，互相抢占资源**。

这点上大家一定要区分清楚，别弄混了！

stream api 的并行操作和串行操作，只有一个方法区别，其他都一样，例如下面我们使用`parallelStream`来输出空字符串的数量：

```
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 采用并行计算方法，获取空字符串的数量
long count = strings.parallelStream().filter(string -> string.isEmpty()).count();
```

在实际使用的时候，`并行操作`不一定比`串行操作`快！对于简单操作，数量非常大，同时服务器是多核的话，建议使用Stream并行！反之，采用串行操作更可靠！

### 七、小结

本文主要，围绕 jdk stream api 操作，结合实际的日常开发需求，做了简单总结和分享，鉴于笔者才疏学浅，可能也有理解不到位的地方，欢迎网友们批评指出！

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

![赞赏二维码](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495026&idx=1&sn=186d83aab739d5a694c8350db717c490&key=c20744b47a3230e1704c95ea621a5b8a4bd0128e1ab6ec414b511e119a8d37df06e9e1dea018c3fef7e6e5b7d216b7643d1d007de781cbe04c2717fa2f947a79206abb5cf23cd395962711cc9af625f5d97dfc4240c457a360a3fdbab8abc279e05c29bbda23b6d605d5be48c37f45024a99b45f3bdedfa6455e9fe3e2af996d&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AbP%2F0Mi6LjV1sRReYZ9lg54%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2)[喜欢作者](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495026&idx=1&sn=186d83aab739d5a694c8350db717c490&key=c20744b47a3230e1704c95ea621a5b8a4bd0128e1ab6ec414b511e119a8d37df06e9e1dea018c3fef7e6e5b7d216b7643d1d007de781cbe04c2717fa2f947a79206abb5cf23cd395962711cc9af625f5d97dfc4240c457a360a3fdbab8abc279e05c29bbda23b6d605d5be48c37f45024a99b45f3bdedfa6455e9fe3e2af996d&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AbP%2F0Mi6LjV1sRReYZ9lg54%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2##)

阅读 2977

分享收藏

赞30在看23

写下你的留言

**精选留言**

- ![img](http://wx.qlogo.cn/mmopen/GUO7SFXUWWN3iamibbhl0OPml1f1Tm6EH6r5Rx8CKTDb2vOwlPfT1jPN7WoibvtoV753Z8Yic1ENeHLt49Blf0zbcyBEa9gG1CXM/96)

  多喝热水

  

  jdk13有啥新特性呢

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)

  

  文章正在准备中

- ![img](http://wx.qlogo.cn/mmopen/Q3auHgzwzM6BaYet78PM25nOiabD4Lbyc9icbYnEIueZwg3jxJPyoCfvV45rgJKwRtnZA28WkicTl5npT9QDGFoUBs7I64hgzy0RO2ffcEB870/96)

  南蛮虫

  

  一行代码那么长，可读性太差

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)

  

  可以换行

- ![img](http://wx.qlogo.cn/mmopen/3Lqm1xHojtaJxFia7PfTAwI6D90H90SWM8lAqficrc5AiabVeXYYo5ctbo1rbI5zBb5aCRpAmuxZ9143icq32s0iaoA/96)

  KK

  

  看怎么用了，如果一层套一层就是灾难

  ![img](http://wx.qlogo.cn/mmhead/Q3auHgzwzM4Pr0tDibJericMHyYBTKCWXrmF788JSJ3Y1ibTJ5iaibCibc5g/0)

  Java极客技术

  (作者)