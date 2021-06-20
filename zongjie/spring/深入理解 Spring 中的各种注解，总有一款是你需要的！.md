## 深入理解 Spring 中的各种注解，总有一款是你需要的！

点击关注 👉 [JAVA技术](javascript:void(0);) *4月7日*

![图片](https://mmbiz.qpic.cn/mmbiz_png/5Zvn1hEISlfgtVxYKhFRNksWJ1lgibcbru653NgxHINn4XpYHyEicKaWhqcn4uqxpibZ7K1nJWDjOqvGtIYiceAVbA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





*作者：digdeep*

*来源：digdeep.cnblogs.com/digdeep/p/4525567.html*

Spring中的注解大概可以分为两大类：

1. spring的bean容器相关的注解，或者说bean工厂相关的注解；
2. springmvc相关的注解。

spring的bean容器相关的注解，先后有：@Required， @Autowired, @PostConstruct, @PreDestory

还有Spring3.0开始支持的JSR-330标准javax.inject.*中的注解(@Inject, @Named, @Qualifier, @Provider, @Scope, @Singleton).

springmvc相关的注解有：@Controller, @RequestMapping, @RequestParam， @ResponseBody等等。

要理解Spring中的注解，先要理解Java中的注解。

# 1. Java中的注解

Java中1.5中开始引入注解，我们最熟悉的应该是：@Override, 它的定义如下：

```
/**
 * Indicates that a method declaration is intended to override a
 * method declaration in a supertype. If a method is annotated with
 * this annotation type compilers are required to generate an error
 * message unless at least one of the following conditions hold:
 * The method does override or implement a method declared in a
 * supertype.
 * The method has a signature that is override-equivalent to that of
 * any public method declared in Object.
 *
 * @author  Peter von der Ahé
 * @author  Joshua Bloch
 * @jls 9.6.1.4 @Override
 * @since 1.5
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public@interface Override {
}
```

从注释，我们可以看出，@Override的作用是，提示编译器，使用了@Override注解的方法必须override父类或者java.lang.Object中的一个同名方法。

我们看到@Override的定义中使用到了 @Target, @Retention，它们就是所谓的“元注解”——就是定义注解的注解，或者说注解注解的注解(晕了...)。我们看下@Retention

```
/**
 * Indicates how long annotations with the annotated type are to
 * be retained.  If no Retention annotation is present on
 * an annotation type declaration, the retention policy defaults to
 * RetentionPolicy.CLASS.
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public@interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```

@Retention用于提示注解被保留多长时间，有三种取值：

```
publicenum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,
    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,
    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

RetentionPolicy.SOURCE 保留在源码级别，被编译器抛弃(@Override就是此类)；RetentionPolicy.CLASS被编译器保留在编译后的类文件级别，但是被虚拟机丢弃；

RetentionPolicy.RUNTIME保留至运行时，可以被反射读取。

再看 @Target:

```
package java.lang.annotation;

/**
 * Indicates the contexts in which an annotation type is applicable. The
 * declaration contexts and type contexts in which an annotation type may be
 * applicable are specified in JLS 9.6.4.1, and denoted in source code by enum
 * constants of java.lang.annotation.ElementType
 * @since 1.5
 * @jls 9.6.4.1 @Target
 * @jls 9.7.4 Where Annotations May Appear
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public@interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```

@Target用于提示该注解使用的地方，取值有：

```
publicenum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,
    /** Field declaration (includes enum constants) */
    FIELD,
    /** Method declaration */
    METHOD,
    /** Formal parameter declaration */
    PARAMETER,
    /** Constructor declaration */
    CONSTRUCTOR,
    /** Local variable declaration */
    LOCAL_VARIABLE,
    /** Annotation type declaration */
    ANNOTATION_TYPE,
    /** Package declaration */
    PACKAGE,
    /**
     * Type parameter declaration
     * @since 1.8
     */
    TYPE_PARAMETER,
    /**
     * Use of a type
     * @since 1.8
     */
    TYPE_USE
}
```

分别表示该注解可以被使用的地方：

1. 类,接口，注解，enum;
2. 属性域；
3. 方法；
4. 参数；
5. 构造函数；
6. 局部变量；
7. 注解类型；
8. 包

所以：

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public@interface Override {
}
```

表示 @Override 只能使用在方法上，保留在源码级别，被编译器处理，然后抛弃掉。

还有一个经常使用的元注解 @Documented ：

```
/**
 * Indicates that annotations with a type are to be documented by javadoc
 * and similar tools by default.  This type should be used to annotate the
 * declarations of types whose annotations affect the use of annotated
 * elements by their clients.  If a type declaration is annotated with
 * Documented, its annotations become part of the public API
 * of the annotated elements.
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public@interface Documented {
}
```

表示注解是否能被 javadoc 处理并保留在文档中。

# 2. 使用 元注解 来自定义注解 和 处理自定义注解

有了元注解，那么我就可以使用它来自定义我们需要的注解。结合自定义注解和AOP或者过滤器，是一种十分强大的武器。

比如可以使用注解来实现权限的细粒度的控制——在类或者方法上使用权限注解，然后在AOP或者过滤器中进行拦截处理。

下面是一个关于登录的权限的注解的实现：

```
/**
 * 不需要登录注解
 */
@Target({ ElementType.METHOD, ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public@interface NoLogin {
}
```

我们自定义了一个注解 @NoLogin, 可以被用于 方法 和 类 上，注解一直保留到运行期，可以被反射读取到。

该注解的含义是：被 @NoLogin 注解的类或者方法，即使用户没有登录，也是可以访问的。下面就是对注解进行处理了：

```
/**
 * 检查登录拦截器
 * 如不需要检查登录可在方法或者controller上加上@NoLogin
 */
publicclass CheckLoginInterceptor implements HandlerInterceptor {
    privatestaticfinal Logger logger = Logger.getLogger(CheckLoginInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        if (!(handler instanceof HandlerMethod)) {
            logger.warn("当前操作handler不为HandlerMethod=" + handler.getClass().getName() + ",req="
                        + request.getQueryString());
            returntrue;
        }
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        String methodName = handlerMethod.getMethod().getName();
        // 判断是否需要检查登录
        NoLogin noLogin = handlerMethod.getMethod().getAnnotation(NoLogin.class);
        if (null != noLogin) {
            if (logger.isDebugEnabled()) {
                logger.debug("当前操作methodName=" + methodName + "不需要检查登录情况");
            }
            returntrue;
        }
        noLogin = handlerMethod.getMethod().getDeclaringClass().getAnnotation(NoLogin.class);
        if (null != noLogin) {
            if (logger.isDebugEnabled()) {
                logger.debug("当前操作methodName=" + methodName + "不需要检查登录情况");
            }
            returntrue;
        }
        if (null == request.getSession().getAttribute(CommonConstants.SESSION_KEY_USER)) {
            logger.warn("当前操作" + methodName + "用户未登录,ip=" + request.getRemoteAddr());
            response.getWriter().write(JsonConvertor.convertFailResult(ErrorCodeEnum.NOT_LOGIN).toString()); // 返回错误信息
            returnfalse;
        }
        returntrue;
    }
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) throws Exception {
    }
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {
    }
}
```

上面我们定义了一个登录拦截器，首先使用反射来判断方法上是否被 @NoLogin 注解：

NoLoginnoLogin=handlerMethod.getMethod().getAnnotation(NoLogin.class);

然后判断类是否被 @NoLogin 注解：

noLogin=handlerMethod.getMethod().getDeclaringClass().getAnnotation(NoLogin.class);

如果被注解了，就返回 true，如果没有被注解，就判断是否已经登录，没有登录则返回错误信息给前台和false.

这是一个简单的使用注解和过滤器来进行权限处理的例子。

扩展开来，那么我们就可以使用注解，来表示某方法或者类，只能被具有某种角色，或者具有某种权限的用户所访问，然后在过滤器中进行判断处理。

# 3. spring的bean容器相关的注解

1）@Autowired 是我们使用得最多的注解，其实就是 autowire=byType 就是根据类型的自动注入依赖（基于注解的依赖注入）

可以使用在属性域，方法，构造函数上。

2）@Qualifier 就是 autowire=byName, @Autowired注解判断多个bean类型相同时，就需要使用 @Qualifier("xxBean") 来指定依赖的bean的id：

```
@Controller
@RequestMapping("/user")
publicclass HelloController {
    @Autowired
    @Qualifier("userService")
    private UserService userService;
```

3）@Resource 属于JSR250标准，用于属性域额和方法上。也是 byName 类型的依赖注入。

使用方式：@Resource(name="xxBean"). 不带参数的 @Resource 默认值类名首字母小写。

4）JSR-330标准javax.inject.*中的注解(@Inject, @Named, @Qualifier, @Provider, @Scope, @Singleton)。

@Inject就相当于@Autowired, @Named 就相当于 @Qualifier, 另外 @Named 用在类上还有 @Component的功能。

5）@Component， @Controller, @Service, @Repository, 这几个注解不同于上面的注解

上面的注解都是将被依赖的bean注入进入，而这几个注解的作用都是生产bean,

这些注解都是注解在类上，将类注解成spring的bean工厂中一个一个的bean。

@Controller, @Service, @Repository基本就是语义更加细化的@Component。

6）@PostConstruct 和 @PreDestroy 不是用于依赖注入，而是bean 的生命周期。

类似于 init-method(InitializeingBean) destory-method(DisposableBean)

# 4. spring中注解的处理

spring中注解的处理基本都是通过实现接口 BeanPostProcessor 来进行的：

```
publicinterface BeanPostProcessor {
    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```

相关的处理类有：

AutowiredAnnotationBeanPostProcessor，CommonAnnotationBeanPostProcessor，PersistenceAnnotationBeanPostProcessor，RequiredAnnotationBeanPostProcessor

这些处理类，可以通过 context:annotation-config/ 配置隐式的配置进spring容器。

这些都是依赖注入的处理，还有生产bean的注解(@Component， @Controller, @Service, @Repository)的处理：

这些都是通过指定扫描的基包路径来进行的，将他们扫描进spring的bean容器。

注意context:component-scan也会默认将 AutowiredAnnotationBeanPostProcessor，CommonAnnotationBeanPostProcessor 配置进来。所以context:annotation-config/是可以省略的。

另外context:component-scan也可以扫描@Aspect风格的AOP注解，但是需要在配置文件中加入 aop:aspectj-autoproxy/ 进行配合。

# 5. Spring注解和JSR-330标准注解的区别：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



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

阅读 1016

分享收藏

赞5在看3