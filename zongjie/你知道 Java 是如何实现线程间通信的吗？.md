## 你知道 Java 是如何实现线程间通信的吗？

点击关注 👉 [JAVA技术](javascript:void(0);) *4月4日*

![图片](https://mmbiz.qpic.cn/mmbiz_png/5Zvn1hEISlcUf8T2924P4N74I8W0BdZ9u7fTrib55zFehxFperSyChU3vbnmV5MV9EibOwc5lTGQRgxCKoRibdSRg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

*来源**：http://wingjay.com*

正常情况下，每个子线程完成各自的任务就可以结束了。不过有的时候，我们希望多个线程协同工作来完成某个任务，这时就涉及到了线程间通信了。

本文涉及到的知识点：

1. thread.join(),
2. object.wait(),
3. object.notify(),
4. CountdownLatch,
5. CyclicBarrier,
6. FutureTask,
7. Callable 。

> 本文涉及代码：
> https://github.com/wingjay/HelloJava/blob/master/multi-thread/src/ForArticle.java

下面我从几个例子作为切入点来讲解下 Java 里有哪些方法来实现线程间通信。

1. 如何让两个线程依次执行？
2. 那如何让 两个线程按照指定方式有序交叉运行呢？
3. 四个线程 A B C D，其中 D 要等到 A B C 全执行完毕后才执行，而且 A B C 是同步运行的
4. 三个运动员各自准备，等到三个人都准备好后，再一起跑
5. 子线程完成某件任务后，把得到的结果回传给主线程

### 如何让两个线程依次执行？

假设有两个线程，一个是线程 A，另一个是线程 B，两个线程分别依次打印 1-3 三个数字即可。我们来看下代码：

```
private static void demo1() {  
    Thread A = new Thread(new Runnable() {  
        @Override  
        public void run() {  
            printNumber("A");  
        }  
    });  
    Thread B = new Thread(new Runnable() {  
        @Override  
        public void run() {  
            printNumber("B");  
        }  
    });  
    A.start();  
    B.start();  
}  
```

其中的 printNumber(String) 实现如下，用来依次打印 1, 2, 3 三个数字：

```
private static void printNumber(String threadName) {  
    int i=0;  
    while (i++ < 3) {  
        try {  
            Thread.sleep(100);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println(threadName + " print: " + i);  
    }  
}  
```

这时我们得到的结果是：

> B print: 1
> A print: 1
> B print: 2
> A print: 2
> B print: 3
> A print: 3

可以看到 A 和 B 是同时打印的。

那么，如果我们希望 B 在 A 全部打印 完后再开始打印呢？我们可以利用 thread.join() 方法，代码如下:

```
private static void demo2() {  
    Thread A = new Thread(new Runnable() {  
        @Override  
        public void run() {  
            printNumber("A");  
        }  
    });  
    Thread B = new Thread(new Runnable() {  
        @Override  
        public void run() {  
            System.out.println("B 开始等待 A");  
            try {  
                A.join();  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            printNumber("B");  
        }  
    });  
    B.start();  
    A.start();  
}  
```

得到的结果如下：

> B 开始等待 A
> A print: 1
> A print: 2
> A print: 3
>
> B print: 1
> B print: 2
> B print: 3

所以我们能看到 A.join() 方法会让 B 一直等待直到 A 运行完毕。

### 那如何让 两个线程按照指定方式有序交叉运行呢？

还是上面那个例子，我现在希望 A 在打印完 1 后，再让 B 打印 1, 2, 3，最后再回到 A 继续打印 2, 3。这种需求下，显然 Thread.join() 已经不能满足了。我们需要更细粒度的锁来控制执行顺序。推荐：[Java面试练题宝典](https://mp.weixin.qq.com/s?__biz=MzIyNDU2ODA4OQ==&mid=2247486846&idx=1&sn=75bd4dcdbaad73191a1e4dbf691c118a&scene=21#wechat_redirect)

这里，我们可以利用 object.wait() 和 object.notify() 两个方法来实现。代码如下：

```
/**  
 * A 1, B 1, B 2, B 3, A 2, A 3  
 */  
private static void demo3() {  
    Object lock = new Object();  
    Thread A = new Thread(new Runnable() {  
        @Override  
        public void run() {  
            synchronized (lock) {  
                System.out.println("A 1");  
                try {  
                    lock.wait();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
                System.out.println("A 2");  
                System.out.println("A 3");  
            }  
        }  
    });  
    Thread B = new Thread(new Runnable() {  
        @Override  
        public void run() {  
            synchronized (lock) {  
                System.out.println("B 1");  
                System.out.println("B 2");  
                System.out.println("B 3");  
                lock.notify();  
            }  
        }  
    });  
    A.start();  
    B.start();  
}  
```

打印结果如下：

> A 1
> A waiting…
>
> B 1
> B 2
> B 3
> A 2
> A 3

正是我们要的结果。

那么，这个过程发生了什么呢？

1. 首先创建一个 A 和 B 共享的对象锁 lock = new Object();
2. 当 A 得到锁后，先打印 1，然后调用 lock.wait() 方法，交出锁的控制权，进入 wait 状态；
3. 对 B 而言，由于 A 最开始得到了锁，导致 B 无法执行；直到 A 调用 lock.wait() 释放控制权后， B 才得到了锁；
4. B 在得到锁后打印 1， 2， 3；然后调用 lock.notify() 方法，唤醒正在 wait 的 A;
5. A 被唤醒后，继续打印剩下的 2，3。

为了更好理解，我在上面的代码里加上 log 方便读者查看。

```
private static void demo3() {  
    Object lock = new Object();  
    Thread A = new Thread(new Runnable() {  
        @Override  
        public void run() {  
            System.out.println("INFO: A 等待锁 ");  
            synchronized (lock) {  
                System.out.println("INFO: A 得到了锁 lock");  
                System.out.println("A 1");  
                try {  
                    System.out.println("INFO: A 准备进入等待状态，放弃锁 lock 的控制权 ");  
                    lock.wait();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
                System.out.println("INFO: 有人唤醒了 A, A 重新获得锁 lock");  
                System.out.println("A 2");  
                System.out.println("A 3");  
            }  
        }  
    });  
    Thread B = new Thread(new Runnable() {  
        @Override  
        public void run() {  
            System.out.println("INFO: B 等待锁 ");  
            synchronized (lock) {  
                System.out.println("INFO: B 得到了锁 lock");  
                System.out.println("B 1");  
                System.out.println("B 2");  
                System.out.println("B 3");  
                System.out.println("INFO: B 打印完毕，调用 notify 方法 ");  
                lock.notify();  
            }  
        }  
    });  
    A.start();  
    B.start();  
}  
```

打印结果如下:

> INFO: A 等待锁
> INFO: A 得到了锁 lock
> A 1
> INFO: A 准备进入等待状态，调用 lock.wait() 放弃锁 lock 的控制权
> INFO: B 等待锁
> INFO: B 得到了锁 lock
> B 1
> B 2
> B 3
> INFO: B 打印完毕，调用 lock.notify() 方法
> INFO: 有人唤醒了 A, A 重新获得锁 lock
> A 2
> A 3

##### 四个线程 A B C D，其中 D 要等到 A B C 全执行完毕后才执行，而且 A B C 是同步运行的

最开始我们介绍了 thread.join()，可以让一个线程等另一个线程运行完毕后再继续执行，那我们可以在 D 线程里依次 join A B C，不过这也就使得 A B C 必须依次执行，而我们要的是这三者能同步运行。

或者说，我们希望达到的目的是：A B C 三个线程同时运行，各自独立运行完后通知 D；对 D 而言，只要 A B C 都运行完了，D 再开始运行。针对这种情况，我们可以利用 CountdownLatch 来实现这类通信方式。它的基本用法是：

1. 创建一个计数器，设置初始值，CountdownLatch countDownLatch = new CountDownLatch(2);
2. 在 等待线程 里调用 countDownLatch.await() 方法，进入等待状态，直到计数值变成 0；
3. 在 其他线程 里，调用 countDownLatch.countDown() 方法，该方法会将计数值减小 1；
4. 当 其他线程 的 countDown() 方法把计数值变成 0 时，等待线程 里的 countDownLatch.await() 立即退出，继续执行下面的代码。

实现代码如下：

```
private static void runDAfterABC() {  
    int worker = 3;  
    CountDownLatch countDownLatch = new CountDownLatch(worker);  
    new Thread(new Runnable() {  
        @Override  
        public void run() {  
            System.out.println("D is waiting for other three threads");  
            try {  
                countDownLatch.await();  
                System.out.println("All done, D starts working");  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }).start();  
    for (char threadName='A'; threadName <= 'C'; threadName++) {  
        final String tN = String.valueOf(threadName);  
        new Thread(new Runnable() {  
            @Override  
            public void run() {  
                System.out.println(tN + " is working");  
                try {  
                    Thread.sleep(100);  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
                System.out.println(tN + " finished");  
                countDownLatch.countDown();  
            }  
        }).start();  
    }  
}  
```

下面是运行结果：

> D is waiting for other three threads
> A is working
> B is working
> C is working
>
> A finished
> C finished
> B finished
> All done, D starts working

其实简单点来说，CountDownLatch 就是一个倒计数器，我们把初始计数值设置为3，当 D 运行时，先调用 countDownLatch.await() 检查计数器值是否为 0，若不为 0 则保持等待状态；当A B C 各自运行完后都会利用countDownLatch.countDown()，将倒计数器减 1，当三个都运行完后，计数器被减至 0；此时立即触发 D 的 await() 运行结束，继续向下执行。

因此，CountDownLatch 适用于一个线程去等待多个线程的情况。

##### 三个运动员各自准备，等到三个人都准备好后，再一起跑

上面是一个形象的比喻，针对 线程 A B C 各自开始准备，直到三者都准备完毕，然后再同时运行 。也就是要实现一种 线程之间互相等待 的效果，那应该怎么来实现呢？

上面的 CountDownLatch 可以用来倒计数，但当计数完毕，只有一个线程的 await() 会得到响应，无法让多个线程同时触发。

为了实现线程间互相等待这种需求，我们可以利用 CyclicBarrier 数据结构，它的基本用法是：

1. 先创建一个公共 CyclicBarrier 对象，设置 同时等待 的线程数，CyclicBarrier cyclicBarrier = new CyclicBarrier(3);
2. 这些线程同时开始自己做准备，自身准备完毕后，需要等待别人准备完毕，这时调用 cyclicBarrier.await(); 即可开始等待别人；
3. 当指定的 同时等待 的线程数都调用了 cyclicBarrier.await();时，意味着这些线程都准备完毕好，然后这些线程才 同时继续执行。

实现代码如下，设想有三个跑步运动员，各自准备好后等待其他人，全部准备好后才开始跑：

```
private static void runABCWhenAllReady() {  
    int runner = 3;  
    CyclicBarrier cyclicBarrier = new CyclicBarrier(runner);  
    final Random random = new Random();  
    for (char runnerName='A'; runnerName <= 'C'; runnerName++) {  
        final String rN = String.valueOf(runnerName);  
        new Thread(new Runnable() {  
            @Override  
            public void run() {  
                long prepareTime = random.nextInt(10000) + 100;  
                System.out.println(rN + " is preparing for time: " + prepareTime);  
                try {  
                    Thread.sleep(prepareTime);  
                } catch (Exception e) {  
                    e.printStackTrace();  
                }  
                try {  
                    System.out.println(rN + " is prepared, waiting for others");  
                    cyclicBarrier.await(); // 当前运动员准备完毕，等待别人准备好  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                } catch (BrokenBarrierException e) {  
                    e.printStackTrace();  
                }  
                System.out.println(rN + " starts running"); // 所有运动员都准备好了，一起开始跑  
            }  
        }).start();  
    }  
}  
```

打印的结果如下：

> A is preparing for time: 4131
> B is preparing for time: 6349
> C is preparing for time: 8206
> A is prepared, waiting for others
> B is prepared, waiting for others
> C is prepared, waiting for others
> C starts running
> A starts running
> B starts running

子线程完成某件任务后，把得到的结果回传给主线程

实际的开发中，我们经常要创建子线程来做一些耗时任务，然后把任务执行结果回传给主线程使用，这种情况在 Java 里要如何实现呢？

回顾线程的创建，我们一般会把 Runnable 对象传给 Thread 去执行。Runnable定义如下：

```
public interface Runnable {  
    public abstract void run();  
}  
```

可以看到 run() 在执行完后不会返回任何结果。那如果希望返回结果呢？这里可以利用另一个类似的接口类 Callable：

```
@FunctionalInterface  
public interface Callable<V> {  
    /**  
     * Computes a result, or throws an exception if unable to do so.  
     *  
     * @return computed result  
     * @throws Exception if unable to compute a result  
     */  
    V call() throws Exception;  
}  
```

可以看出 Callable 最大区别就是返回范型 V 结果。

那么下一个问题就是，如何把子线程的结果回传回来呢？在 Java 里，有一个类是配合 Callable 使用的：FutureTask，不过注意，它获取结果的 get 方法会阻塞主线程。

举例，我们想让子线程去计算从 1 加到 100，并把算出的结果返回到主线程。

```
private static void doTaskWithResultInWorker() {  
    Callable<Integer> callable = new Callable<Integer>() {  
        @Override  
        public Integer call() throws Exception {  
            System.out.println("Task starts");  
            Thread.sleep(1000);  
            int result = 0;  
            for (int i=0; i<=100; i++) {  
                result += i;  
            }  
            System.out.println("Task finished and return result");  
            return result;  
        }  
    };  
    FutureTask<Integer> futureTask = new FutureTask<>(callable);  
    new Thread(futureTask).start();  
    try {  
        System.out.println("Before futureTask.get()");  
        System.out.println("Result: " + futureTask.get());  
        System.out.println("After futureTask.get()");  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    } catch (ExecutionException e) {  
        e.printStackTrace();  
    }  
}  
```

打印结果如下：

> Before futureTask.get()
> Task starts
> Task finished and return result
> Result: 5050
> After futureTask.get()

可以看到，主线程调用 futureTask.get() 方法时阻塞主线程；然后 Callable 内部开始执行，并返回运算结果；此时 futureTask.get() 得到结果，主线程恢复运行。

这里我们可以学到，通过 FutureTask 和 Callable 可以直接在主线程获得子线程的运算结果，只不过需要阻塞主线程。当然，如果不希望阻塞主线程，可以考虑利用 ExecutorService，把 FutureTask 放到线程池去管理执行。

### 小结

多线程是现代语言的共同特性，而线程间通信、线程同步、线程安全是很重要的话题。本文针对 Java 的线程间通信进行了大致的讲解，后续还会对线程同步、线程安全进行讲解。

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

阅读 1076

分享收藏

赞8在看5