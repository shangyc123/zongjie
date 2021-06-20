## 用Socket编程？我还是选择了Netty

鸭血粉丝 [Java极客技术](javascript:void(0);) *4月23日*



每天早上七点三十，准时推送干货

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/tWOhQMr1wdCNesO2CZfm7icCXk2Z2DaAoVExrvFdBWfjtEricQUllkrd9dbmqallriaL5VSytkQeBbiaj4ahTlxibjw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



最近在写IO的这块的内容，于是就免不了去研究IO，NIO，AIO，在看NIO的时候，阿粉就发现了一个极其好的东西，那就是Netty，为什么说他好呢?大家就跟着阿粉来深度认识一下Netty吧。



#### 什么是Netty

我们先看看百度百科给我们的解释，什么是Netty

百度百科：

Netty是由JBOSS提供的一个java开源框架，现为Github上的独立项目。Netty提供异步的、事件驱动的网络应用程序框架和工具，用以快速开发高性能、高可靠性的网络服务器和客户端程序。

也就是说，Netty 是一个基于NIO的客户、服务器端的编程框架，使用Netty 可以确保你快速和简单的开发出一个网络应用，例如实现了某种协议的客户、服务端应用。Netty相当于简化和流线化了网络应用的编程开发过程，例如：基于TCP和UDP的socket服务开发。

上面是来自于百度百科给出的解释，我们也能清晰的看到，Netty是一个基于NIO的模型，使用Netty的地方很多就是socket服务开发，而关于NIO，相信大家肯定不陌生，因为之前阿粉出过一篇文章，也给NIO说的是明明白白。

文章连接奉上：

[如果有人再问你 Java IO，把这篇文章砸他头上](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491115&idx=1&sn=14f1712cc787befd6c78d64612a00f95&chksm=c28571eaf5f2f8fca45ea7caf3ae7b5dee57c89e2cf7c0d115d0fa5211cca976e3462ea5cb74&token=950876690&lang=zh_CN&scene=21#wechat_redirect)

#### 对比Netty和传统的Socket

我们既然要说Netty，那么我们肯定要对Netty还有Socket不同的代码进行一个分析，分析的透彻了，你才能真的选择使用Netty，而不再进行Socket的开发了，相信到时候，大家肯定会做出最正确的选择。

##### 传统Socket编程服务端

```
package com.example.demo.socket;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @ClassName SocketDemo
 * @Author yld
 * @Date 2021/4/19 10:33
 * @Description SocketDemo
 */
public class SocketServerDemo {

    public static void main(String[] args) {
        ServerSocket server=null;
        try {
            server=new ServerSocket(18080);
            System.out.println("时间服务已经启动--端口号为：18080...");
            while (true){
                Socket client = server.accept();
                //每次接收到一个新的客户端连接，启动一个新的线程来处理
                new Thread(new TimeServerHandler(client)).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                server.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
package com.example.demo.socket;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;
import java.util.Calendar;

/**
 * @ClassName TimeServerHandler
 * @Author yld
 * @Date 2021/4/19 10:35
 * @Description TimeServerHandler
 */
public class TimeServerHandler implements Runnable {
    private Socket clientProxxy;

    public TimeServerHandler(Socket clientProxxy) {
        this.clientProxxy = clientProxxy;
    }

    @Override
    public void run() {
        BufferedReader reader = null;
        PrintWriter writer = null;
        try {
            reader = new BufferedReader(new InputStreamReader(clientProxxy.getInputStream()));
            writer =new PrintWriter(clientProxxy.getOutputStream()) ;
            while (true) {//因为一个client可以发送多次请求，这里的每一次循环，相当于接收处理一次请求
                String request = reader.readLine();
                if (!"GET CURRENT TIME".equals(request)) {
                    writer.println("BAD_REQUEST");
                } else {
                    writer.println(Calendar.getInstance().getTime().toLocaleString());
                }
                writer.flush();
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            try {
                writer.close();
                reader.close();
                clientProxxy.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

##### 传统Socket编程客户端

```
package com.example.demo.socket;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

/**
 * @ClassName SocketClientDemo
 * @Author yld
 * @Date 2021/4/19 10:42
 * @Description SocketClientDemo
 */
public class SocketClientDemo {
    public static void main(String[] args)  {
        BufferedReader reader = null;
        PrintWriter writer = null;
        Socket client=null;
        try {
            client=new Socket("127.0.0.1",18080);
            writer = new PrintWriter(client.getOutputStream());
            reader = new BufferedReader(new InputStreamReader(client.getInputStream()));

            while (true){//每隔5秒发送一次请求
                writer.println("GET CURRENT TIME");
                writer.flush();
                String response = reader.readLine();
                System.out.println("Current Time:"+response);
                Thread.sleep(5000);
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                writer.close();
                reader.close();
                client.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
```

老生常谈，我们来执行一下才能知道效果，

首先运行服务端：

```
TimeServer Started on 18080...
```

接着启动客户端

```
Current Time:2021-4-19 10:48:21
Current Time:2021-4-19 10:48:26
Current Time:2021-4-19 10:48:31
Current Time:2021-4-19 10:48:36
Current Time:2021-4-19 10:48:41
Current Time:2021-4-19 10:48:46
Current Time:2021-4-19 10:48:51
```

大家看一下，这是不是就是相当于一个Socket的客户端和服务端之间进行通信的过程，在client端可以发送请求指令"GET CURRENT TIME"给server端，每隔5秒钟发送一次，每次server端都返回当前时间。

而这也是传统的BIO的做法，每一个client都需要去对应一个线程去进行处理，client越多，那么你要开启的线程也就会越多，也就是说，如果采用BIO通信模型的服务端，通常由一个独立的Acceptor线程负责监听客户端的连接，当接收到客户端的连接请求后，会为每一个客户端请求创建新的线程进行请求的处理，处理完成后通过输出流返回信息给客户端，响应完成后销毁线程。

模型图如下

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



这时候就有大佬说，你不会用线程池么？但是阿粉之前曾经在面试的时候说过，使用线程池的话，它实际上并没有解决任何实际性的问题，他实际上就是对BIO做了一个优化，属于伪异步IO通信。

伪异步IO通信模型图

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



异步IO通信确实能缓解一部分的压力，但是这种模型也是有缺陷的，当有大量客户端请求的时候，随着并发访问量的增长，伪异步IO就会造成线程池阻塞。

这时候就取决于你们这个是想选择，系统发生线程堆栈溢出、创建新线程失败等问题呢，还是选择大量客户端请求，造成线程池阻塞。

都说，技术是为了解决问题而出现的，那么接下来就有了解决这个问题的技术出现了，Netty，我们来看看Netty吧。

#### Netty环境搭建

在这里我们使用的依旧是Springboot来整合Netty的环境，然后在后续过程中，使用Netty实现服务端程序和客户端程序，最后阿粉还会继续给大家再说一下这个BIO和NIO的模型，虽然Netty并没有实现传说中的AIO，但是已经算是吧这个NIO的模型，实现到了极致了。

来，既然使用Springboot，那我们就把项目搭建一下吧，顺便把我们的Netty依赖都加入进来。

(1).我们先创建出来一个项目

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



(2).加入我们的pom的依赖

```
 <!--Netty-->
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.31.Final</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.22</version>
        </dependency>

        <!-- logger -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
        
```

##### Netty服务端程序

```
package com.example.demo.netty;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;

/**
 * @ClassName NettyServerDemo
 * @Author yld
 * @Date 2021/4/19 11:11
 * @Description NettyServerDemo
 */
public class NettyServerDemo {

    private int port=18081;
    public void run() throws Exception {
        EventLoopGroup bossGroup = new NioEventLoopGroup(); 
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap b = new ServerBootstrap(); 
            b.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class) 
                    .childHandler(new ChannelInitializer<SocketChannel>() { 
                        @Override
                        public void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new TimeServerHandler());
                        }
                    });

            ChannelFuture f = b.bind(port).sync(); 
            System.out.println("TimeServer Started on 18081...");
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
            bossGroup.shutdownGracefully();
        }
    }
    public static void main(String[] args) throws Exception {
        new NettyServerDemo().run();
    }
}
package com.example.demo.netty;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.util.Date;

/**
 * @ClassName TimeServerHandler
 * @Author yld
 * @Date 2021/4/19 11:19
 * @Description TimeServerHandler
 */
public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String request = (String) msg;
        String response = null;
        if ("QUERY TIME ORDER".equals(request)) {
            response = new Date(System.currentTimeMillis()).toString();
        } else {
            response = "BAD REQUEST";
        }
        response = response + System.getProperty("line.separator");
         ByteBuf resp = Unpooled.copiedBuffer(response.getBytes());
        ctx.writeAndFlush(resp);
    }
}
```

##### Netty客户端程序

```
package com.example.demo.netty;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;

/**
 * @ClassName NettyClientDemo
 * @Author yld
 * @Date 2021/4/19 11:21
 * @Description NettyClientDemo
 */
public class NettyClientDemo {
    public static void main(String[] args) throws Exception {
        String host = "localhost";
        int port = 18081;
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(workerGroup);
            b.channel(NioSocketChannel.class);
            b.handler(new ChannelInitializer<SocketChannel>() {
                @Override
                public void initChannel(SocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new LineBasedFrameDecoder(1024));
                    ch.pipeline().addLast(new StringDecoder());
                    ch.pipeline().addLast(new TimeClientHandler());
                }
            });
            // 开启客户端.
            ChannelFuture f = b.connect(host, port).sync();
            // 等到连接关闭.
            f.channel().closeFuture().sync();
        } finally {
            workerGroup.shutdownGracefully();
        }
    }
}
package com.example.demo.netty;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * @ClassName TimeClientHandler
 * @Author yld
 * @Date 2021/4/19 11:22
 * @Description TimeClientHandler
 */
public class TimeClientHandler extends ChannelInboundHandlerAdapter {

    private byte[] req=("QUERY TIME ORDER" + System.getProperty("line.separator")).getBytes();
    @Override
    public void channelActive(ChannelHandlerContext ctx) {//1
        ByteBuf message = Unpooled.buffer(req.length);
        message.writeBytes(req);
        ctx.writeAndFlush(message);
    }
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.println("Now is:" + body);
    }
}
```

首先启动服务端，控制台输出：

```
TimeServer Started on 18081...
```

接着启动客户端，控制要输出：

```
Now is:Mon Apr 19 11:34:21 CST 2021
```

既然代码写了，那我们是不是就得来分析一下这个Netty在中间都干了什么东西，他的类是什么样子的，都有哪些方法。

大家先从代码的源码上开始看起，因为我们在代码中分别使用到了好几个类，而这些类的父类，或者是接口定义者追根到底，也就是这个样子的，我们从IDEA中打开他的类图可以清晰的看到。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



而在源码中，最重要的就是这个Channel,接下来，我们就来分析一波吧。

**Channel**

相信大家对看源码这块绝对是觉得非常难受，阿粉也是，但是还是需要给大家进行分析源码，不然怎么知道源码创作者是什么意思呢？

All I/O operations are asynchronous.一句话点出核心所有的IO操作都是异步的，这意味着任何I/O调用都将立即返回，但不保证请求的I/O操作已完成。这是在源码的注释上面给出的解释。

**Channel分类**

- 服务端: NioServerSocketChannel
- 客户端: NioSocketChannel

看到这个，大家肯定也都不陌生，因为Channel即可以在JDK的Socket中充当管道出现，同时，也在Netty的服务端和客户端进行IO数据交互，充当一个媒介的存在，那么他的区别在哪？

Netty对Jdk原生的ServerSocketChannel进行了封装和增强封装成了NioXXXChannel, 相对于原生的JdkChannel,Netty的Channel增加了如下的组件。

- id 标识唯一身份信息
- 可能存在的parent Channel
- 管道 pepiline
- 用于数据读写的unsafe内部类
- 关联上相伴终生的NioEventLoop

如果大家想对这个这个类的API有更多的了解，官网给大家送上。

Package io.netty.channel

今天这篇文章，我们先分析各种源代码，然后在接下来的文章，阿粉会给大家介绍关于Netty的线程模型，他使用的是什么线程模型来实现NIO的网络模型。

而关于Channel，其实换成大家容易理解的话的话，那就是**由它负责同对端进行网络通信、注册和数据操作等功能**

```
A Channel can have a parent depending on how it was created. For instance, a SocketChannel, that was accepted by ServerSocketChannel, will return the ServerSocketChannel as its parent on parent().

The semantics of the hierarchical structure depends on the transport implementation where the Channel belongs to. For example, you could write a new Channel implementation that creates the sub-channels that share one socket connection, as BEEP and SSH do.
```

一个Channel可以有一个父Channel，这取决于它是如何创建的。例如，被ServerSocketChannel接受的SocketChannel将返回ServerSocketChannel作为其parent（）上的父对象。层次结构的语义取决于通道所属的传输实现。

Channel的抽象类AbstractChannel中有一个受保护的构造方法，而AbstractChannel内部有一个pipeline属性，Netty在对Channel进行初始化的时候将该属性初始化为DefaultChannelPipeline的实例。

#### 为什么选择Netty

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



其实在上面的图中，已经能看出来了，不同的I/O模型，效率，使用难度，吞吐量都是非常重要的，所以选择的时候，肯定要慎重选择，而我们为什么不使用Java原生的呢？

实际上很简单，1.复杂，2.不好用

对于Java的NIO的类库和API繁杂使用麻烦，你需要熟练掌握Selectol,ServerSocketChannel,SocketChannel,ByteBuffer 等

JDK NIO的BUG，比如epoll bug，这个BUG会在linux上导致cpu 100%，使得nio server/client不可用，而且在1.7中都没有解决完这个bug,只不过发生频率比较低。

而Netty是一个高性能、异步事件驱动的NIO框架，它提供了对TCP、UDP和文件传输的支持，作为一个异步NIO框架，Netty的所有IO操作都是异步非阻塞的，通过Future-Listener机制，用户可以方便的主动获取或者通过通知机制获得IO操作结果。

所以，综上考虑，我们还是选择使用了Netty，而不使用Socket，作为开发者的你,会选择什么样的I/O模型呢？

##### 文章参考

《Netty实战》

《田守枝的Java技术博客》



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

阅读 2592

分享收藏

赞16在看10

写下你的留言