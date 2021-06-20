## 四张图，读懂 BIO、NIO、AIO、多路复用 IO 的区别！

点击关注 👉 [JAVA技术](javascript:void(0);) *4月9日*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/5Zvn1hEISlcI2jswIyukHKJ2bnW5TaXdQPbqpkr1N8Dst0PIobTCRSbTeibeHtNvLXgGwrH9jia2niaIz56fuOqbA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 学习之前，我们先来了解一下IO模型：

①同步阻塞IO（Blocking IO）：即传统的IO模型。

②同步非阻塞IO（Non-blocking IO）：默认创建的socket都是阻塞的，非阻塞IO要求socket被设置为NONBLOCK。注意这里所说的NIO并非Java的NIO（New IO）库。

③多路复用IO（IO Multiplexing）：即经典的Reactor设计模式，有时也称为异步阻塞IO，Java中的Selector和Linux中的epoll都是这种模型（Redis单线程为什么速度还那么快，就是因为用了多路复用IO和缓存操作的原因）

④异步IO（Asynchronous IO）：即经典的Proactor设计模式，也称为异步非阻塞IO。

## 图解：

![图片](https://mmbiz.qpic.cn/mmbiz_png/A0bYOQcma0MdARlQKMzo6zh1PC367DbHrEgSlnUNE9v2CXIZAchPUBWeF4x6eLkcDAiczE8h0IByicowgibDFg3kw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)img

![图片](https://mmbiz.qpic.cn/mmbiz_png/A0bYOQcma0MdARlQKMzo6zh1PC367DbH8T66eBiavjjzNjRkFOnj6DibjECdT9mb5s8sTNZDoPIFD0BLRYDOzxAQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)img

![图片](https://mmbiz.qpic.cn/mmbiz_png/A0bYOQcma0MdARlQKMzo6zh1PC367DbHKyMozSuCdN8L6pt7sL4TfWNTAHAxeyEWH4fqQp8F7ThW9y4FFKogJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)img

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)img

看了这些，你应该对这些IO有了新的认识了吧，那就给我个赞呗^_^

作者：扛麻袋的少年

来源：blog.csdn.net/lzb348110175/article/details/98941378

如果看到这里，说明你喜欢这篇文章，请 转发、点赞。同时 标星（置顶）本公众号可以第一时间接受到博文推送。

**推荐阅读：**（点击标题可跳转）

------



[一个程序员的水平能差到什么程度？](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247486207&idx=1&sn=20ace3e581c1e1456b3a9c466749838b&chksm=ebade16fdcda687901d1d32564f04b473c7b765259f15a15a73391f42d345d127b494dfc78e0&scene=21#wechat_redirect)

[SpringBoot开源在线考试系统](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247486200&idx=1&sn=8dee16fbc27acf7c22cc6b225d7fe422&chksm=ebade168dcda687e2195c4538f6109ba35c1d541e908b92ab219363dbb24e2a25d456e703d81&scene=21#wechat_redirect)

[Java异常最最最全面的面试及解答](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247486172&idx=1&sn=ffc994fafc601f53b6dc6e537cf68fbb&chksm=ebade14cdcda685aeb9e9162bc0429800de911f462a3e284cb1ab48cd46e658886533ec92f31&scene=21#wechat_redirect)

[基于SpringBoot+Vue在线考试系统【web端+小程序端】【附带源码】](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247486163&idx=1&sn=8dcc9296e7723b116f110dd863a0246c&chksm=ebade143dcda6855b487e74b42485d7702d57ad3a63a9e11640bbf61c3f96e4a105ecfadbe60&scene=21#wechat_redirect)

[推荐一款开源java版的视频管理系统](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247486156&idx=1&sn=ed06d6f3aeccc5bbab8fe988c3039598&chksm=ebade15cdcda684a68f92a613cd75845b3063b98d6dde10ac9ec40a3f41d7969d4eb47004492&scene=21#wechat_redirect)

[从零搭建SpringCloud服务（史上最详细）](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247486137&idx=1&sn=fe0685bef8a68fd2632b53736efcbb72&chksm=ebade129dcda683f6eb336eb08a816d5a1ffbd7d7a4bef178a0ba87a0fc0a240a5ff2ba89d64&scene=21#wechat_redirect)

[带工作流的springboot后台管理项目，一个企业级快速开发解决方案](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247486108&idx=1&sn=5c13089d4fedc48ced2100b24aaeaef4&chksm=ebade10cdcda681a9ffed809fc6814875de379fc2b9f8763b36ad4a35c6fe2c52045caae0c4c&scene=21#wechat_redirect)

[用Java实现每天给对象发情话](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247486033&idx=1&sn=93fdab26e1d6a9b5ddae6846ff9867e5&chksm=ebade1c1dcda68d7b391853c50e0a391759b3a885c35f0d2123b9f6120a7fb2aeb7927be5618&scene=21#wechat_redirect)

[一个女生不主动联系你还有机会吗？](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247485967&idx=1&sn=e35b8ee672221fd4b4ee6b544ad875a7&chksm=ebade19fdcda688975660c781dfc6cc223bef0e0248a9b2fd529d838539317c9aedc8cc08175&scene=21#wechat_redirect)

[一个女程序员的故事（程序员必看）](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247485696&idx=1&sn=81277e9da75454815977fba9099c7c81&chksm=ebade290dcda6b86126fcb06d36f5ffb25f4af865812d7e86496df4fc929e901afa6bb6fb2d1&scene=21#wechat_redirect)

[MySQL数据库面试题（2020最新、最全、最完整版本）附pdf版](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247485261&idx=1&sn=afc4bd2f59aa128f2defafc0852972a5&chksm=ebadecdddcda65cbf8c0866afb6343248e96bec4ca3da80660f9cd395a8f1129917e95cefb21&scene=21#wechat_redirect)

[【2020最新版】Java基础知识面试题.pdf](http://mp.weixin.qq.com/s?__biz=MzI4MTIxNDcxOQ==&mid=2247484933&idx=1&sn=f551bfadb3d8710e755574c5eeef29a9&chksm=ebaded95dcda648386ba0ec3c8c444f0454b9ced8c066772b35bd5fa545156d43290c14ce7ca&scene=21#wechat_redirect)



```
编程·技术·经验
欢迎扫码关注
```



阅读 844

分享收藏

赞4在看3