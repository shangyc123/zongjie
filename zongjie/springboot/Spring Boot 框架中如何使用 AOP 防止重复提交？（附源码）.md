## Spring Boot 框架中如何使用 AOP 防止重复提交？（附源码）

在传统的web项目中，防止重复提交，通常做法是：后端生成一个唯一的提交令牌（uuid），并存储在服务端。页面提交请求携带这个提交令牌，后端验证并在第一次验证后删除该令牌，保证提交请求的唯一性。

上述的思路其实没有问题的，但是需要前后端都稍加改动，如果在业务开发完在加这个的话，改动量未免有些大了，本节的实现方案无需前端配合，纯后端处理。

**思路**

1、自定义注解 @NoRepeatSubmit 标记所有Controller中的提交请求

2、通过AOP 对所有标记了 @NoRepeatSubmit 的方法拦截

3、在业务方法执行前，获取当前用户的 token（或者JSessionId）+ 当前请求地址，作为一个唯一 KEY，去获取 Redis 分布式锁（如果此时并发获取，只有一个线程会成功获取锁）

4、业务方法执行后，释放锁

**关于Redis 分布式锁**

1）不了解的同学戳这里 ==> Redis分布式锁的正确实现方式。

2）使用Redis 是为了在负载均衡部署，如果是单机的部署的项目可以使用一个线程安全的本地Cache 替代 Redis。

**Code**

这里只贴出 AOP 类和测试类，完整代码：Github，Gitee



```
@Aspect
@Component
public class RepeatSubmitAspect {

    private final static Logger LOGGER = LoggerFactory.getLogger(RepeatSubmitAspect.class);

    @Autowired
    private RedisLock redisLock;

    @Pointcut("@annotation(noRepeatSubmit)")
    public void pointCut(NoRepeatSubmit noRepeatSubmit) {
    }

    @Around("pointCut(noRepeatSubmit)")
    public Object around(ProceedingJoinPoint pjp, NoRepeatSubmit noRepeatSubmit) throws Throwable {
        int lockSeconds = noRepeatSubmit.lockTime();

        HttpServletRequest request = RequestUtils.getRequest();
        Assert.notNull(request, "request can not null");

        // 此处可以用token或者JSessionId
        String token = request.getHeader("Authorization");
        String path = request.getServletPath();
        String key = getKey(token, path);
        String clientId = getClientId();

        boolean isSuccess = redisLock.tryLock(key, clientId, lockSeconds);

        if (isSuccess) {
            LOGGER.info("tryLock success, key = [{}], clientId = [{}]", key, clientId);
            // 获取锁成功, 执行进程
            Object result;
            try {
                result = pjp.proceed();

            } finally {
                // 解锁
                redisLock.releaseLock(key, clientId);
                LOGGER.info("releaseLock success, key = [{}], clientId = [{}]", key, clientId);

            }

            return result;

        } else {
            // 获取锁失败，认为是重复提交的请求
            LOGGER.info("tryLock fail, key = [{}]", key);
            return new ResultBean(ResultBean.FAIL, "重复请求，请稍后再试", null);
        }

    }

    private String getKey(String token, String path) {
        return token + path;
    }

    private String getClientId() {
        return UUID.randomUUID().toString();
    }

}
```



**多线程测试**

测试代码如下，模拟十个请求并发同时提交



```
@Component
public class RunTest implements ApplicationRunner {

    private static final Logger LOGGER = LoggerFactory.getLogger(RunTest.class);

    @Autowired
    private RestTemplate restTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("执行多线程测试");
        String url="http://localhost:8000/submit";
        CountDownLatch countDownLatch = new CountDownLatch(1);
        ExecutorService executorService = Executors.newFixedThreadPool(10);

        for(int i=0; i<10; i++){
            String userId = "userId" + i;
            HttpEntity request = buildRequest(userId);
            executorService.submit(() -> {
                try {
                    countDownLatch.await();
                    System.out.println("Thread:"+Thread.currentThread().getName()+", time:"+System.currentTimeMillis());
                    ResponseEntity<String> response = restTemplate.postForEntity(url, request, String.class);
                    System.out.println("Thread:"+Thread.currentThread().getName() + "," + response.getBody());

                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }

        countDownLatch.countDown();
    }

    private HttpEntity buildRequest(String userId) {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        headers.set("Authorization", "yourToken");
        Map<String, Object> body = new HashMap<>();
        body.put("userId", userId);
        return new HttpEntity<>(body, headers);
    }

}
```



成功防止重复提交，控制台日志如下，可以看到十个线程的启动时间几乎同时发起，只有一个请求提交成功了。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



> 项目源码：
> gitee.com/yintianwen7/taven-springboot-learning/tree/master/repeat-submit-intercept

> 作者：殷天文
>
> jianshu.com/p/09860b74658e