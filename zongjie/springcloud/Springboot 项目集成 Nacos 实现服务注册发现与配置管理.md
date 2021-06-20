## Springboot 项目集成 Nacos 实现服务注册发现与配置管理

原创 鸭血粉丝 [Java极客技术](javascript:void(0);) *4月18日*

每天早上七点三十，准时推送干货



![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tWOhQMr1wdCHHNCc2MMicMIezMX6OgRiaEJxtN6BKxvCTuaYnEs3uBubusM1CHdFyyWyoggU7HtMjwaoicbgkuqbA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Hello 大家好，我是阿粉，前面的文章给大家介绍了一下如何在本地搭建微服务环境下的服务注册中心和配置管理中心 `Nacos`，今天通过

我们通过使用 `SpringBoot` 项目集成 `Nacos` 来给大家演示一下是如何使用 `Nacos` 来实现服务发现和配置管理的。

## 启动 Nacos 服务

启动完本地搭建的 `Nacos` 服务后，我们可以看到，目前的服务管理下面的服务列表里面在三个命名空间下都没有服务，这是正常的，因为目前我们还没有服务接入`Nacos`。

![图片](https://mmbiz.qpic.cn/mmbiz_png/tWOhQMr1wdDA1jHpj7cBVoOCuVw0GavSubbV4pDNsIbLuYicdWTOTtQ2D7Zg1UG2aCPNjh6nkExsoiaf5HzKTzhg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Nacos 服务启动成功后，我们再创建两个 `SpringBoot` 项目，一个用于接入 `Nacos` 服务注册与发现和配置中心作为服务提供者 `Producer`，另一个只接入 `Nacos`的服务注册与发现，调用 `Producer` 获取配置中心的参数，我们叫做`Consumer`。

## 服务提供者 Producer

1. 我们首先创建一个 `SpringBoot` 的项目，`bootstrap.properties` 文件内容如下：

   ```
   spring.application.name=producer
   
   #######################配置中心配置#################################
   # 指定的命名空间，只会在对应的命名空间下查找对应的配置文件
   spring.cloud.nacos.config.namespace=caeser-adsys-naming
   spring.cloud.nacos.config.file-extension=properties
   # 配置的分组名称
   spring.cloud.nacos.config.group=TEST1
   # 配置文件，数组形式，可以多个，依次递增
   spring.cloud.nacos.config.ext-config[0].data-id=com.example.properties
   spring.cloud.nacos.config.ext-config[0].group=TEST1
   # 配置中心的地址
   spring.cloud.nacos.config.server-addr=127.0.0.1:8848
   #启用自动刷新对应的配置文件
   spring.cloud.nacos.config.ext-config[0].refresh=true
   ######################服务注册发现配置##################################
   
   # 服务集群名称
   spring.cloud.nacos.discovery.cluster-name=TEST1_GROUP
   # 服务注册中心的地址
   spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   # 服务的命名空间
   spring.cloud.nacos.discovery.namespace=caeser-adsys-naming
   ```

2. `application.properties` 的文件内容如下，主要就是一个端口，其他配置根据情况自行添加或删除就好：

   ```
   # 服务启动的端口
   server.port=8080
   spring.main.allow-bean-definition-overriding=true
   # tomcat 配置
   server.tomcat.max-threads=500
   spring.mvc.servlet.load-on-startup=1
   spring.servlet.multipart.max-file-size=40MB
   spring.servlet.multipart.max-request-size=100MB
   # 日志配置
   logging.level.root=info
   logging.level.com.alibaba=error
   logging.pattern.console=%clr{[%level]}{green} [%d{yyyy-MM-dd HH:mm:ss}] %clr{[${PID:-}]}{faint} %clr{[%thread]}{magenta} %clr{[%-40.40logger{80}:%line]}{cyan} %msg%n
   ```

3. 在启动类上面增加如下注解

   ```
   package com.ziyou.nacos.demo.producer;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cache.annotation.EnableCaching;
   import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
   
   @SpringBootApplication(scanBasePackages = "com.ziyou.nacos")
   @EnableDiscoveryClient
   @EnableCaching
   public class ProducerApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ProducerApplication.class, args);
       }
   }
   ```

4. `pom.xml` 文件内容如下：

   ```
   <?xml version="1.0" encoding="UTF-8"?>
   
   <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
     <modelVersion>4.0.0</modelVersion>
   
     <parent>
       <groupId>org.example</groupId>
       <artifactId>nacos-demo</artifactId>
       <version>1.0-SNAPSHOT</version>
     </parent>
   
     <artifactId>producer</artifactId>
     <version>1.0-SNAPSHOT</version>
   
     <name>producer Maven Webapp</name>
     <!-- FIXME change it to the project's website -->
     <url>http://www.example.com</url>
   
     <properties>
       <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <maven.compiler.source>1.7</maven.compiler.source>
       <maven.compiler.target>1.7</maven.compiler.target>
       <spring.maven.plugin.version>2.2.2.RELEASE</spring.maven.plugin.version>
     </properties>
   
     <dependencies>
       <!-- Spring Boot -->
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter</artifactId>
         <exclusions>
           <exclusion>
             <groupId>org.springframework.boot</groupId>
             <artifactId>spring-boot-starter-logging</artifactId>
           </exclusion>
         </exclusions>
       </dependency>
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-log4j2</artifactId>
       </dependency>
       <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
   
       <!-- nacos 配置中心 -->
       <dependency>
         <groupId>com.alibaba.cloud</groupId>
         <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
       </dependency>
       <!-- nacos 注册发现 -->
       <dependency>
         <groupId>com.alibaba.cloud</groupId>
         <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       </dependency>
     </dependencies>
   
     <build>
       <!--指定下面的目录为资源文件-->
       <resources>
         <!--设置自动替换-->
         <resource>
           <directory>src/main/resources</directory>
           <filtering>true</filtering>
           <includes>
             <include>**/**</include>
           </includes>
         </resource>
       </resources>
       <finalName>producer</finalName>
       <plugins>
         <plugin>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-maven-plugin</artifactId>
           <version>${spring.maven.plugin.version}</version>
           <executions>
             <execution>
               <goals>
                 <goal>repackage</goal>
               </goals>
             </execution>
           </executions>
         </plugin>
       </plugins>
     </build>
   </project>
   ```

5. 在 `Producer` 侧提供一个获取配置里面内容的接口，代码如下：

   ```
   package com.ziyou.nacos.demo.producer.controller;
   
   import com.ziyou.nacos.demo.producer.config.UserConfig;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   /**
    * <br>
    * <b>Function：</b><br>
    * <b>Author：</b>@author ziyou<br>
    * <b>Date：</b>2021-04-11 19:59<br>
    * <b>Desc：</b>无<br>
    */
   @RestController
   @RequestMapping(value = "producer")
   public class ProducerController {
   
       private UserConfig userConfig;
   
       @GetMapping("/getUsername")
       private String getUsername() {
           String result = userConfig.getUsername() + "-" + userConfig.getPassword();
           System.out.println(result);
           return result;
       }
   
       @Autowired
       public void setUserConfig(UserConfig userConfig) {
           this.userConfig = userConfig;
       }
   }
   ```

   ```
   package com.ziyou.nacos.demo.producer.config;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.cloud.context.config.annotation.RefreshScope;
   import org.springframework.stereotype.Component;
   
   /**
    * <br>
    * <b>Function：</b><br>
    * <b>Author：</b>@author ziyou<br>
    * <b>Date：</b>2021-04-11 20:39<br>
    * <b>Desc：</b>无<br>
    */
   @RefreshScope
   @Component
   public class UserConfig {
       @Value("${username}")
       private String username;
       @Value("${password}")
       private String password;
   
       public String getUsername() {
           return username;
       }
   
       public void setUsername(String username) {
           this.username = username;
       }
   
       public String getPassword() {
           return password;
       }
   
       public void setPassword(String password) {
           this.password = password;
       }
   }
   ```

6. 启动 `Producer`，并且手动调用接口，启动 `Producer` 过后，我们在 `Nacos` 的服务注册列表可以看如下所示的内容，在 `test1` 的命名空间下，已经有了我们创建的 `Producer` 服务。

   

7. ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

8. 通过手动调用 `Producer` 的接口 `http://127.0.0.1:8080/producer/getUsername` 显示如下内容![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)并且我们看下此时 `Nacos` 的配置中心里面配置文件`com.example.properties` 里面的内容正是这个，这个时候我们手动把配置里面`password` 参数的值改成`JavaGeek666`，再次访问接口，我们会发现接口的输出也自动改变了。

   ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

   修改配置内容如下：

   ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

   再次访问结果如下：

   ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 服务调用者 Consumer

前面我们已经完成了`Producer` 的服务注册与配置动态生效的功能，这个时候基本已经可以使用了，不过我们还需要更进一步通过 `Nacos` 来实现服务发现，接下来我们创建 `Consumer` 的 `SpringBoot` 的项目，配置文件和`pom.xml` 文件基本一致，只要修改端口以及对应地方，下面贴一下不一样的地方

1. `boostrap.properties` 内容如下，因为这里我们只调用`Producer` 的接口，不需要接入 `Nacos` 的配置中心，所以这里只配置发服务注册与发现

   ```
   spring.application.name=consumer
   
   ######################服务注册发现配置##################################
   spring.cloud.nacos.discovery.cluster-name=TEST1_GROUP
   spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
   spring.cloud.nacos.discovery.namespace=caeser-adsys-naming
   ```

2. 启动类，配置上 `feignClient` 需要扫描的包路径

   ```
   package com.ziyou.nacos.demo.consumer;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cache.annotation.EnableCaching;
   import org.springframework.cloud.openfeign.EnableFeignClients;
   
   /**
    * <br>
    * <b>Function：</b><br>
    * <b>Author：</b>@author ziyou<br>
    * <b>Date：</b>2021-04-11 17:07<br>
    * <b>Desc：</b>无<br>
    */
   @SpringBootApplication(scanBasePackages = "com.ziyou.nacos")
   @EnableFeignClients(basePackages = {"com.ziyou.nacos.demo.consumer.rpc"})
   @EnableCaching
   public class ConsumerApplication {
       public static void main(String[] args) {
           SpringApplication.run(ConsumerApplication.class, args);
       }
   }
   ```

3. 编写调用 `Producer` 的接口，`FeignClient` 里面的 `value` 就是 `Producer` 的应用名称

   ```
   package com.ziyou.nacos.demo.consumer.rpc;
   
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.stereotype.Component;
   import org.springframework.web.bind.annotation.GetMapping;
   
   /**
    * <br>
    * <b>Function：</b><br>
    * <b>Author：</b>@author ziyou<br>
    * <b>Date：</b>2021-04-11 20:01<br>
    * <b>Desc：</b>无<br>
    */
   @FeignClient(value = "producer")
   @Component
   public interface IProducerFeign {
       /**
        * 获取生产者名称接口
        *
        * @return
        */
       @GetMapping("/producer/getUsername")
       String getUsername();
   
   }
   ```

   ```
   package com.ziyou.nacos.demo.consumer.controller;
   
   import com.ziyou.nacos.demo.consumer.rpc.IProducerFeign;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   /**
    * <br>
    * <b>Function：</b><br>
    * <b>Author：</b>@author ziyou<br>
    * <b>Date：</b>2021-04-11 19:59<br>
    * <b>Desc：</b>无<br>
    */
   @RestController
   @RequestMapping(value = "consumer")
   public class TestNacosController {
   
       private IProducerFeign iProducerFeign;
   
       @GetMapping("/testNacos")
       private String testNacos() {
           return iProducerFeign.getUsername();
       }
   
       @Autowired
       public void setiProducerFeign(IProducerFeign iProducerFeign) {
           this.iProducerFeign = iProducerFeign;
       }
   }
   ```

4. 启动`Consumer`，我们可以看到在 `Nacos` 如下图所示![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

5. 调用 `Consumer` 的接口`consumer/testNacos`，结果如下图所示，同样的如果此时更改了 `Nacos` 配置文件中的内容，`Consumer` 这边也是可以实时更新的，感兴趣的小伙伴可以自己试试。

   ![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

今天主要给大家介绍了一下如何通过 `SpringBoot` 项目来接入 Naocs 实现服务注册与发现，以及配置管理和动态刷新，相关的代码已经上传到 GitHub 了，公众号回复【源码】获取地址。



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



![img](https://mmbiz.qlogo.cn/mmbiz_jpg/fPADUR9p6acau4uticGBhaiaeH6q8T7g9Zw7BHRDpx9B7BdKKgWZvWOyJRWL0fTZyfJwiaAH4pFKtlb7Cv8vovcCA/0?wx_fmt=jpeg)

鸭血粉丝

原创不易，众筹一碗鸭血粉丝汤

![赞赏二维码](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495078&idx=1&sn=c7732d38027ee5b6e9a04b2e3041e2b9&key=bf0a475117e7a82e627ae4ef8f86752eb5d7171a45b37cbfc490e8693daca75671e7728cb0935bde49f80a6f4a812237007138763db15bba013fdda94705d5880c9fd93e7d81cbffd533a139031515eb62b45d481b1cc5aec9f613888b5f2d88468cdf5765050341238e3344588f8798d549114895e9f1063f7769b3a36fd1d2&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AVc1D01BO4RINR%2Fjs4nm%2B%2F4%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2)[喜欢作者](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247495078&idx=1&sn=c7732d38027ee5b6e9a04b2e3041e2b9&key=bf0a475117e7a82e627ae4ef8f86752eb5d7171a45b37cbfc490e8693daca75671e7728cb0935bde49f80a6f4a812237007138763db15bba013fdda94705d5880c9fd93e7d81cbffd533a139031515eb62b45d481b1cc5aec9f613888b5f2d88468cdf5765050341238e3344588f8798d549114895e9f1063f7769b3a36fd1d2&ascene=0&uin=MjYzNjIyMzg4MQ%3D%3D&devicetype=Windows+10+x64&version=6302019c&lang=zh_CN&exportkey=AVc1D01BO4RINR%2Fjs4nm%2B%2F4%3D&pass_ticket=OWtm6r0zC3z9GzIgc9ZkUHdrBKVkEvdzEoV8ZueHfTh%2FnpO1f5kBXoFLutqVP7vF&wx_header=0&fontgear=2##)

阅读 1520

分享收藏

赞5在看13

写下你的留言