## 使用 JWT 来保护你的 SpringBoot 应用

## 关键词

Spring Boot、OAuth 2.0、JWT、Spring Security、SSO、UAA

## 写在前面

这几天有人问我Spring Boot结合Spring Security实现OAuth认证的问题，写了个Demo，顺便分享下。Spring 2之后就没再用过Java，主要是xml太麻烦，就投入了Node.js的怀抱，现在Java倒是好过之前很多，无论是执行效率还是其他什么。感谢Pivotal团队在Spring boot上的努力，感谢Josh Long，一个有意思的攻城狮。

我又搞Java也是为了去折腾微服务，因为目前看国内就Java程序猿最好找，虽然水平好的难找，但是至少能找到，不像其他编程语言，找个会世界上最好的编程语言PHP的人真的不易。

## Spring Boot

有了Spring Boot这样的神器，可以很简单的使用强大的Spring框架。你需要关心的事儿只是创建应用，不必再配置了，“Just run!”，这可是Josh Long每次演讲必说的，他的另一句必须说的就是“make jar not war”，这意味着，不用太关心是Tomcat还是Jetty或者Undertow了。专心解决逻辑问题，这当然是个好事儿，部署简单了很多。

## 创建Spring Boot应用

有很多方法去创建Spring Boot项目，官方也推荐用：

> Spring Boot在线项目创建
> http://start.spring.io/
> CLI 工具
> https://docs.spring.io/spring-boot/docs/current/reference/html/cli-using-the-cli.html

start.spring.io可以方便选择你要用的组件，命令行工具当然也可以。目前Spring Boot已经到了1.53，我是懒得去更新依赖，继续用1.52版本。虽然阿里也有了中央库的国内版本不知道是否稳定。

如果你感兴趣，可以自己尝试下。你可以选Maven或者Gradle成为你项目的构建工具，Gradle优雅一些，使用了Groovy语言进行描述。

打开start.spring.io，创建的项目只需要一个Dependency，也就是Web，然后下载项目，用IntellJ IDEA打开。我的Java版本是1.8。

这里看下整个项目的pom.xml文件中的依赖部分:

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

所有Spring Boot相关的依赖都是以starter形式出现，这样你无需关心版本和相关的依赖，所以这样大大简化了开发过程。

当你在pom文件中集成了spring-boot-maven-plugin插件后你可以使用Maven相关的命令来run你的应用。例如mvn spring-boot:run，这样会启动一个嵌入式的Tomcat，并运行在8080端口，直接访问你当然会获得一个Whitelabel Error Page，这说明Tomcat已经启动了。

## 创建一个Web 应用

这还是一篇关于Web安全的文章，但是也得先有个简单的HTTP请求响应。我们先弄一个可以返回JSON的Controller。修改程序的入口文件：

```
@SpringBootApplication
@RestController
@EnableAutoConfiguration
public class DemoApplication {

    // main函数，Spring Boot程序入口
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    // 根目录映射 Get访问方式 直接返回一个字符串
    @RequestMapping("/")
    Map<String, String> hello() {
      // 返回map会变成JSON key value方式
      Map<String,String> map=new HashMap<String,String>();
      map.put("content", "hello freewolf~");
      return map;
    }
}
```

这里我尽量的写清楚，让不了解Spring Security的人通过这个例子可以了解这个东西，很多人都觉得它很复杂，而投向了Apache Shiro，其实这个并不难懂。知道主要的处理流程，和这个流程中哪些类都起了哪些作用就好了。

Spring Boot对于开发人员最大的好处在于可以对Spring应用进行自动配置。Spring Boot会根据应用中声明的第三方依赖来自动配置Spring框架，而不需要进行显式的声明。

**Spring Boot推荐采用基于Java注解的配置方式，而不是传统的XML。****只需要在主配置 Java 类上添加@EnableAutoConfiguration注解就可以启用自动配置。****Spring Boot的自动配置功能是没有侵入性的，只是作为一种基本的默认实现。**

这个入口类我们添加@RestController和@EnableAutoConfiguration两个注解。@RestController注解相当于@ResponseBody和@Controller合在一起的作用。

run整个项目。访问http://localhost:8080/就能看到这个JSON的输出。使用Chrome浏览器可以装JSON Formatter这个插件，显示更PL一些。

```
{
  "content": "hello freewolf~"
}
```

为了显示统一的JSON返回，这里建立一个JSONResult类进行，简单的处理。首先修改pom.xml，加入org.json相关依赖。

```
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
</dependency>
```

然后在我们的代码中加入一个新的类，里面只有一个结果集处理方法，因为只是个Demo，所有这里都放在一个文件中。这个类只是让返回的JSON结果变为三部分：

- **status** - 返回状态码 0 代表正常返回，其他都是错误
- **message** - 一般显示错误信息
- **result** - 结果集



```
class JSONResult{
    public static String fillResultString(Integer status, String message, Object result){
        JSONObject jsonObject = new JSONObject(){{
            put("status", status);
            put("message", message);
            put("result", result);
        }};
        return jsonObject.toString();
    }
}
```

然后我们引入一个新的@RestController并返回一些简单的结果，后面我们将对这些内容进行访问控制，这里用到了上面的结果集处理类。这里多放两个方法，后面我们来测试权限和角色的验证用。

```
@RestController
class UserController {

    // 路由映射到/users
    @RequestMapping(value = "/users", produces="application/json;charset=UTF-8")
    public String usersList() {

        ArrayList<String> users =  new ArrayList<String>(){{
            add("freewolf");
            add("tom");
            add("jerry");
        }};

        return JSONResult.fillResultString(0, "", users);
    }

    @RequestMapping(value = "/hello", produces="application/json;charset=UTF-8")
    public String hello() {
        ArrayList<String> users =  new ArrayList<String>(){{ add("hello"); }};
        return JSONResult.fillResultString(0, "", users);
    }

    @RequestMapping(value = "/world", produces="application/json;charset=UTF-8")
    public String world() {
        ArrayList<String> users =  new ArrayList<String>(){{ add("world"); }};
        return JSONResult.fillResultString(0, "", users);
    }
}
```

重新run这个文件，访问http://localhost:8080/users就看到了下面的结果:

```
{
  "result": [
    "freewolf",
    "tom",
    "jerry"
  ],
  "message": "",
  "status": 0
}
```

如果你细心，你会发现这里的JSON返回时，Chrome的格式化插件好像并没有识别？这是为什么呢？我们借助curl分别看一下我们写的两个方法的Header信息.

```
curl -I http://127.0.0.1:8080/
curl -I http://127.0.0.1:8080/users
```

可以看到第一个方法hello，由于返回值是Map，Spring已经有相关的机制自动处理成JSON:

```
Content-Type: application/json;charset=UTF-8
```

第二个方法usersList由于返回时String，由于是@RestControler已经含有了@ResponseBody也就是直接返回内容，并不模板。

所以就是：

```
Content-Type: text/plain;charset=UTF-8
```

那怎么才能让它变成JSON呢，其实也很简单只需要补充一下相关注解：

```
@RequestMapping(value = "/users", produces="application/json;charset=UTF-8")
```

这样就好了。

## 使用JWT保护你的Spring Boot应用

终于我们开始介绍正题，这里我们会对/users进行访问控制，先通过申请一个JWT(JSON Web Token读jot)，然后通过这个访问/users，才能拿到数据。

关于JWT,出门奔向以下内容，这些不在本文讨论范围内:

> https://tools.ietf.org/html/rfc7519
> https://jwt.io/

JWT很大程度上还是个新技术，通过使用HMAC(Hash-based Message Authentication Code)计算信息摘要，也可以用RSA公私钥中的私钥进行签名。这个根据业务场景进行选择。

## 添加Spring Security

根据上文我们说过我们要对/users进行访问控制，让用户在/login进行登录并获得Token。这里我们需要将spring-boot-starter-security加入pom.xml。加入后，我们的Spring Boot项目将需要提供身份验证，相关的pom.xml如下:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.7.0</version>
</dependency>
```

至此我们之前所有的路由都需要身份验证。我们将引入一个安全设置类WebSecurityConfig，这个类需要从WebSecurityConfigurerAdapter类继承。

```
@Configuration
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    // 设置 HTTP 验证规则
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 关闭csrf验证
        http.csrf().disable()
                // 对请求进行认证
                .authorizeRequests()
                // 所有 / 的所有请求 都放行
                .antMatchers("/").permitAll()
                // 所有 /login 的POST请求 都放行
                .antMatchers(HttpMethod.POST, "/login").permitAll()
                // 权限检查
                .antMatchers("/hello").hasAuthority("AUTH_WRITE")
                // 角色检查
                .antMatchers("/world").hasRole("ADMIN")
                // 所有请求需要身份认证
                .anyRequest().authenticated()
            .and()
                // 添加一个过滤器 所有访问 /login 的请求交给 JWTLoginFilter 来处理 这个类处理所有的JWT相关内容
                .addFilterBefore(new JWTLoginFilter("/login", authenticationManager()),
                        UsernamePasswordAuthenticationFilter.class)
                // 添加一个过滤器验证其他请求的Token是否合法
                .addFilterBefore(new JWTAuthenticationFilter(),
                        UsernamePasswordAuthenticationFilter.class);
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        // 使用自定义身份验证组件
        auth.authenticationProvider(new CustomAuthenticationProvider());

    }
}
```

先放两个基本类，一个负责存储用户名密码，另一个是一个权限类型，负责存储权限和角色。

```
class AccountCredentials {

    private String username;
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

class GrantedAuthorityImpl implements GrantedAuthority{
    private String authority;

    public GrantedAuthorityImpl(String authority) {
        this.authority = authority;
    }

    public void setAuthority(String authority) {
        this.authority = authority;
    }

    @Override
    public String getAuthority() {
        return this.authority;
    }
}
```

在上面的安全设置类中，我们设置所有人都能访问/和POST方式访问/login，其他的任何路由都需要进行认证。然后将所有访问/login的请求，都交给JWTLoginFilter过滤器来处理。

稍后我们会创建这个过滤器和其他这里需要的JWTAuthenticationFilter和CustomAuthenticationProvider两个类。

先建立一个JWT生成，和验签的类

```
class TokenAuthenticationService {
    static final long EXPIRATIONTIME = 432_000_000;     // 5天
    static final String SECRET = "P@ssw02d";            // JWT密码
    static final String TOKEN_PREFIX = "Bearer";        // Token前缀
    static final String HEADER_STRING = "Authorization";// 存放Token的Header Key

  // JWT生成方法
    static void addAuthentication(HttpServletResponse response, String username) {

    // 生成JWT
        String JWT = Jwts.builder()
                // 保存权限（角色）
                .claim("authorities", "ROLE_ADMIN,AUTH_WRITE")
                // 用户名写入标题
                .setSubject(username)
                // 有效期设置
                        .setExpiration(new Date(System.currentTimeMillis() + EXPIRATIONTIME))
                // 签名设置
                        .signWith(SignatureAlgorithm.HS512, SECRET)
                        .compact();

        // 将 JWT 写入 body
        try {
            response.setContentType("application/json");
            response.setStatus(HttpServletResponse.SC_OK);
            response.getOutputStream().println(JSONResult.fillResultString(0, "", JWT));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

  // JWT验证方法
    static Authentication getAuthentication(HttpServletRequest request) {
        // 从Header中拿到token
        String token = request.getHeader(HEADER_STRING);

        if (token != null) {
            // 解析 Token
            Claims claims = Jwts.parser()
                    // 验签
                    .setSigningKey(SECRET)
                    // 去掉 Bearer
                    .parseClaimsJws(token.replace(TOKEN_PREFIX, ""))
                    .getBody();

            // 拿用户名
            String user = claims.getSubject();

            // 得到 权限（角色）
            List<GrantedAuthority> authorities =  AuthorityUtils.commaSeparatedStringToAuthorityList((String) claims.get("authorities"));

            // 返回验证令牌
            return user != null ?
                    new UsernamePasswordAuthenticationToken(user, null, authorities) :
                    null;
        }
        return null;
    }
}
```

这个类就两个static方法，一个负责生成JWT，一个负责认证JWT最后生成验证令牌。注释已经写得很清楚了，这里不多说了。

下面来看自定义验证组件，这里简单写了，这个类就是提供密码验证功能，在实际使用时换成自己相应的验证逻辑，从数据库中取出、比对、赋予用户相应权限。

```
// 自定义身份认证验证组件
class CustomAuthenticationProvider implements AuthenticationProvider {

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        // 获取认证的用户名 & 密码
        String name = authentication.getName();
        String password = authentication.getCredentials().toString();

        // 认证逻辑
        if (name.equals("admin") && password.equals("123456")) {

            // 这里设置权限和角色
            ArrayList<GrantedAuthority> authorities = new ArrayList<>();
            authorities.add( new GrantedAuthorityImpl("ROLE_ADMIN") );
            authorities.add( new GrantedAuthorityImpl("AUTH_WRITE") );
            // 生成令牌
            Authentication auth = new UsernamePasswordAuthenticationToken(name, password, authorities);
            return auth;
        }else {
            throw new BadCredentialsException("密码错误~");
        }
    }

    // 是否可以提供输入类型的认证服务
    @Override
    public boolean supports(Class<?> authentication) {
        return authentication.equals(UsernamePasswordAuthenticationToken.class);
    }
}
```

下面实现JWTLoginFilter 这个Filter比较简单，除了构造函数需要重写三个方法。

- **attemptAuthentication** - 登录时需要验证时候调用

- **successfulAuthentication** - 验证成功后调用

- **unsuccessfulAuthentication** - 验证失败后调用，这里直接灌入500错误返回，由于同一JSON返回，HTTP就都返回200了

  

```
class JWTLoginFilter extends AbstractAuthenticationProcessingFilter {

    public JWTLoginFilter(String url, AuthenticationManager authManager) {
        super(new AntPathRequestMatcher(url));
        setAuthenticationManager(authManager);
    }

    @Override
    public Authentication attemptAuthentication(
            HttpServletRequest req, HttpServletResponse res)
            throws AuthenticationException, IOException, ServletException {

        // JSON反序列化成 AccountCredentials
        AccountCredentials creds = new ObjectMapper().readValue(req.getInputStream(), AccountCredentials.class);

        // 返回一个验证令牌
        return getAuthenticationManager().authenticate(
                new UsernamePasswordAuthenticationToken(
                        creds.getUsername(),
                        creds.getPassword()
                )
        );
    }

    @Override
    protected void successfulAuthentication(
            HttpServletRequest req,
            HttpServletResponse res, FilterChain chain,
            Authentication auth) throws IOException, ServletException {

        TokenAuthenticationService.addAuthentication(res, auth.getName());
    }


    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {

        response.setContentType("application/json");
        response.setStatus(HttpServletResponse.SC_OK);
        response.getOutputStream().println(JSONResult.fillResultString(500, "Internal Server Error!!!", JSONObject.NULL));
    }
}
```

再完成最后一个类JWTAuthenticationFilter，这也是个拦截器，它拦截所有需要JWT的请求，然后调用TokenAuthenticationService类的静态方法去做JWT验证。

```
class JWTAuthenticationFilter extends GenericFilterBean {

    @Override
    public void doFilter(ServletRequest request,
                         ServletResponse response,
                         FilterChain filterChain)
            throws IOException, ServletException {
        Authentication authentication = TokenAuthenticationService
                .getAuthentication((HttpServletRequest)request);

        SecurityContextHolder.getContext()
                .setAuthentication(authentication);
        filterChain.doFilter(request,response);
    }
}
```

现在代码就写完了，整个Spring Security结合JWT基本就差不多了，下面我们来测试下，并说下整体流程。

开始测试，先运行整个项目，这里介绍下过程：

- **先程序启动** - main函数
- **注册验证组件** - WebSecurityConfig 类 configure(AuthenticationManagerBuilder auth)方法，这里我们注册了自定义验证组件
- **设置验证规则** - WebSecurityConfig 类 configure(HttpSecurity http)方法，这里设置了各种路由访问规则
- **初始化过滤组件** - JWTLoginFilter 和 JWTAuthenticationFilter 类会初始化

首先测试获取Token，这里使用CURL命令行工具来测试。

```
curl -H "Content-Type: application/json" -X POST -d '{"username":"admin","password":"123456"}'  http://127.0.0.1:8080/login
```

结果：

```
{
  "result": "eyJhbGciOiJIUzUxMiJ9.eyJhdXRob3JpdGllcyI6IlJPTEVfQURNSU4sQVVUSF9XUklURSIsInN1YiI6ImFkbWluIiwiZXhwIjoxNDkzNzgyMjQwfQ.HNfV1CU2CdAnBTH682C5-KOfr2P71xr9PYLaLpDVhOw8KWWSJ0lBo0BCq4LoNwsK_Y3-W3avgbJb0jW9FNYDRQ",
  "message": "",
  "status": 0
}
```

这里我们得到了相关的JWT，反Base64之后,就是下面的内容，标准JWT。

```
{"alg":"HS512"}{"authorities":"ROLE_ADMIN,AUTH_WRITE","sub":"admin","exp":1493782240}ͽ]BS`pS6~hCVH%
ܬ)֝ଖoE5р
```

整个过程如下：

- 拿到传入JSON，解析用户名密码 - JWTLoginFilter 类 attemptAuthentication 方法
- 自定义身份认证验证组件，进行身份认证 - CustomAuthenticationProvider 类 authenticate 方法
- 盐城成功 - JWTLoginFilter 类 successfulAuthentication 方法
- 生成JWT - TokenAuthenticationService 类 addAuthentication方法

再测试一个访问资源的：

```
curl -H "Content-Type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJhdXRob3JpdGllcyI6IlJPTEVfQURNSU4sQVVUSF9XUklURSIsInN1YiI6ImFkbWluIiwiZXhwIjoxNDkzNzgyMjQwfQ.HNfV1CU2CdAnBTH682C5-KOfr2P71xr9PYLaLpDVhOw8KWWSJ0lBo0BCq4LoNwsK_Y3-W3avgbJb0jW9FNYDRQ"  http://127.0.0.1:8080/users
```

结果：

```
{
  "result":["freewolf","tom","jerry"],
  "message":"",
  "status":0
}
```

说明我们的Token生效可以正常访问。其他的结果您可以自己去测试。再回到处理流程：

- **接到请求进行拦截** - JWTAuthenticationFilter 中的方法
- **验证JWT** - TokenAuthenticationService 类 getAuthentication 方法
- **访问Controller**

这样本文的主要流程就结束了，本文主要介绍了，如何用Spring Security结合JWT保护你的Spring Boot应用。

如何使用Role和Authority，这里多说一句其实在Spring Security中，对于GrantedAuthority接口实现类来说是不区分是Role还是Authority，二者区别就是如果是hasAuthority判断，就是判断整个字符串，判断hasRole时，系统自动加上ROLE_到判断的Role字符串上，也就是说hasRole("CREATE")和hasAuthority('ROLE_CREATE')是相同的。利用这些可以搭建完整的RBAC体系。

本文到此，你已经会用了本文介绍的知识点。

## 代码

> https://github.com/freew01f/securing-spring-boot-with-jwts