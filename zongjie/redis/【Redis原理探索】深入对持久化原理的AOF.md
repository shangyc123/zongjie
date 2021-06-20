## 【Redis原理探索】深入对持久化原理的AOF

## AOF 的简介

- **Redis 的 AOF 是类似的 Log 的机制，每次写操作都会写到硬盘上，当系统崩溃时，可以通过 AOF 来恢复数据。**
- **每个带有写操作的命令被 Redis 服务器端收到运行时，该命令都会被记录到 AOF 文件上。由于只是一个 append 到文件操作 (采用的是存储 aof_buf 中，并且采用异步方法 fsync/fdatasync)，所以写到硬盘上的操作往往非常快**。

## AOF 的实现

Redis AOF 机制包括了两件事，AOF 和 Rewrite。

- **AOF 类似于普通数据库管理系统日志恢复点，当 AOF 文件随着写命令的运行膨胀时，当文件大小触碰到临界时，rewrite 会被运行**。

- **Rewrite 会像 replication 一样，fork 出一个子进程，创建一个临时文件，遍历数据库，将每个 key、value 对输出到临时文件**。

- - **输出格式就是 Redis 的命令，但是为了减小文件大小，会将多个 key、value 对集合起来用一条命令表达**。
  - **在 rewrite 期间的写操作会保存在内存的 aof_rewrite_buf_block 中，rewrite 成功后这些操作也会复制到临时文件中，在最后临时文件会代替 AOF 文件**。

> **以上在 AOF 打开的情况下，如果 AOF 是关闭的，那么 rewrite 操作可以通过 bgrewriteaof 命令来进行。**

## AOF 的流程

![图片](https://mmbiz.qpic.cn/mmbiz_png/5Xv0xlEBe9ibPdJiacBrTEricUANhgvj2VhMFReibhuINW0RSUYv51JpwyVgiaPW7bcyibXzgAWGFGeLoSHS6PA2GJhA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- Redis Server 启动，如果 AOF 机制打开那么初始化 AOF 状态，并且如果存在 AOF 文件，读取 AOF 文件。随着 Redis 不断接受命令，每个写命令都被添加到 AOF 文件，AOF 文件膨胀到重写阈值需要重写时又或者接收到客户端的 **bgrewriteaof** 命令。
- fork 出一个子进程进行 rewrite，而父进程继续接受操作命令，现在的写操作命令都会被额外添加到一个 **aof_rewrite_buf_blocks** 缓冲中。
- 当子进程 rewrite 结束后，父进程收到子进程退出信号，**把 aof_rewrite_buf_blocks 的缓冲添加到 rewrite 后的 AOF 文件中，然后切换 AOF 的文件 fd，采用新文件覆盖旧文件。rewrite 任务完成，继续第二个步骤。**

## 关键点

- **由于写操作通常是有缓冲的，有可能 AOF 操作并没有写到硬盘中，可以通过 fsync()/fdatasync() 来强制输出到硬盘中**。

- **而 fsync() 的频率可以通过配置文件中的 flush 策略来指定，可以选择每次事件循环写操作都强制 fsync 或者每秒 fsync 至少运行一次**。

- ** 当 rewrite 子进程开始后，父进程接受到的命令会添加到 aof_rewrite_buf_blocks 中，使得 rewrite 成功后，将这些命令添加到新文件中。

- 在 rewrite 过程中，原来的 AOF 也可以选择是否继续添加，由于存在性能上的问题，在 rewrite 过程中，如果 fsync() 继续执行，会导致 IO 性能受损影响 Redis 性能。**所以一般情况下 rewrite 期间禁止 fsync() 到旧 AOF 文件**。**这策略可以在配置文件中修改**。（就是在 rewrite 过程中 aof 操作的同步机制，可以会暂时停止不会 save 到 aof 文件中）。

- 在 rewrite 结束后，在将新 rewrite 文件重命名为配置中指定的文件时，如果旧 AOF 存在，那么会 **unlink** 掉**旧文件并删除**。

- - 这是就存在一个问题，处理 rewrite 文件迁移的是主线程，rename(oldpath, newpath) 过程会覆盖旧文件，**这是 rename 会 unlink(oldfd)，而 unlink 操作会导致 block 主线程**。
  - 这时，** 我们就需要类似 libeio(software.schmorp.de/pkg/libeio.…

### 自动的 bgrewriteaof

- **避免 aof 文件过大，我们会周期性的做 bgrewriteaof 来重整 aof 文件。以前我们会额外的配置 crontab 在业务低峰期执行这个命令，这额外的增加一个 workaroud 的脚本任务在大集群里是很糟糕的，不易检查，出错无法即时发现**。
- **于是这个自动 bgrewriteaof 功能被直接加到 redis 的内部**。首先对于 aof 文件，**server 对象添加一个字段来记录 aof 文件的大小 server.appendonly_current_size**，**每次 aof 发生变化都会维护这个字段**。

#### aof.c

```
nwritten = write(server.appendfd，server.aofbuf，sdslen(server.aofbuf));
.....
server.appendonly_current_size += nwritten;

复制代码
```

> bgrewriteaof 完毕或者实例启动载入 aof 数据后也会调用 aofUpdateCurrentSize 这个函数维护这个字段，同时会记录下此时的 aof 文件的大小 **server.auto_aofrewrite_base_size** 作为基准值，用于接下来判断 aof 增长率。

#### aof.c

```
aofUpdateCurrentSize();
server.auto_aofrewrite_base_size = server.appendonly_current_size;

复制代码
```

> 有了当前值和基准值我们就可以判断 aof 文件的增长情况。另外还需要配置两个参数来判断是否需要自动触发 bgrewriteaof。

#### redis.h

```
int auto_aofrewrite_perc; 
off_t auto_aofrewrite_min_size; 

复制代码
```

- auto_aofrewrite_perc: AOF 文件的大小超过基准百分之多少后触发 **bgrewriteaof**。默认这个值设置为 100，**意味着当前 AOF 是基准大小的两倍的时候触发 bgrewriteaof**。**把它设置为 0 可以禁用自动触发的功能**。
- auto_aofrewrite_min_size: **当前 AOF 文件大于多少字节后才触发。避免在 aof 较小的时候无谓行为。默认大小为 64mb。**

两个参数都是可以在 conf 里静态配置，或者通过 config set 来动态修改的。

```
redis 127.0.0.1:6379> config get auto-aof-rewrite-percentage
1) "auto-aof-rewrite-percentage"
2) "100"
redis 127.0.0.1:6379> config get auto-aof-rewrite-min-size
1) "auto-aof-rewrite-min-size"
2) "1048576"
redis 127.0.0.1:6379> config get auto-aof-rewrite-min-size
1) "auto-aof-rewrite-min-size"
2) "1048576"
redis 127.0.0.1:6379> config set auto-aof-rewrite-percentage 200
OK
redis 127.0.0.1:6379> config set auto-aof-rewrite-min-size 10485760
OK
复制代码
```

> **然后就是触发检查的主逻辑，serverCron 时间事件中每次都会检查现有状态和参数来判断是否需要启动 bgrewriteaof**。

#### redis.c

```
          if (server.bgsavechildpid == -1 &&
              server.bgrewritechildpid == -1 &&
              server.auto_aofrewrite_perc &&
              server.appendonly_current_size > server.auto_aofrewrite_min_size)
          {
             long long base = server.auto_aofrewrite_base_size ?
                             server.auto_aofrewrite_base_size : 1;
             long long growth = (server.appendonly_current_size*100/base) - 100;
             if (growth >= server.auto_aofrewrite_perc) {
                 redisLog(REDIS_NOTICE，"Starting automatic rewriting of AOF on %lld%% growth"，growth);
                 rewriteAppendOnlyFileBackground();
             }
         }
复制代码
```

> **以上代码显示，如果 aof 文件增长百分率 growth 大于 auto_aofrewrite_perc，则自动的触发后一个 bgrewriteaof**。

### 延迟 bgrewriteaof

> **这是个小的改进，手动触发的 bgrewriteaof 的时候如果同时存在 bgsave 在备份，会推迟这次操走的事件，设置 server.aofrewrite_scheduled=1，待到 bgsave 结束后的下一次 serverCron 里才会触发**。

**设置 aofrewrite_scheduled=1**

#### aof.c

```
 void bgrewriteaofCommand(redisClient *c) {
     if (server.bgrewritechildpid != -1) {
         addReplyError(c，"Background append only file rewriting already in 
         progress");
     } else if (server.bgsavechildpid != -1) {
         server.aofrewrite_scheduled = 1;
         addReplyStatus(c，"Background append only file rewriting 
         scheduled");
     } else if (rewriteAppendOnlyFileBackground() == REDIS_OK) {
         addReplyStatus(c，"Background append only file rewriting 
         started");
     } else {
         addReply(c，shared.err);
     }
 }

复制代码
```

### 触发 bgrewriteaof

#### redis.c

```
    if (server.bgsavechildpid == -1 && server.bgrewritechildpid == -1 &&
        server.aofrewrite_scheduled){
         rewriteAppendOnlyFileBackground();
    }
复制代码
```

