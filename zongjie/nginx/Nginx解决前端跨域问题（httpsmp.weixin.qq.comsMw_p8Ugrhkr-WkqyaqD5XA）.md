## Nginx解决前端跨域问题（https://mp.weixin.qq.com/s/Mw_p8Ugrhkr-WkqyaqD5XA）

[写代码的渣渣鹏](javascript:void(0);) *5月21日*





  最近连续两个朋友问我跨域相关问题，我猜想可能不少朋友也遇到类似问题，我打算写个博客聊一下我实际使用的配置，

先说明一下，我并不太了解这配置，没精力去了解太多，但我觉得其中有一些关键的小注意点，可能有些初学者不太注意到，导致配置有问题，本文章可能只对新手有点帮助，如果你有好配置，欢迎评论回复，让大家学习！

Nginx的CORS配置，网上太多这配置了，但大家更多的复制粘贴、转发，几乎都是类似下面这三两行：





- 
- 
- 

```
add_header Access-Control-Allow-Origin *;add_header Access-Control-Allow-Headers X-Requested-With;add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
```





这样有用么？有用，我以前这样使用也正常过，但后来还是遇到问题了，发现有些项目请求就不成功，也遇到有些浏览器成功，有些浏览器不成功；

我也在网上查找各种资料和调整写法，最后我调整好的写法，基本的使用没问题，我在项目中也一直使用着！



下面发一段我实际项目中的部分配置：





- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
location /xxx-web {  add_header 'Access-Control-Allow-Origin' $http_origin;  add_header 'Access-Control-Allow-Credentials' 'true';  add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';  add_header 'Access-Control-Allow-Headers' 'DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';  add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';  if ($request_method = 'OPTIONS') {    add_header 'Access-Control-Max-Age' 1728000;    add_header 'Content-Type' 'text/plain; charset=utf-8';    add_header 'Content-Length' 0;    return 204;  }  root   html;  index  index.html index.htm;  proxy_pass http://127.0.0.1:8080;  proxy_set_header Host $host;  proxy_set_header X-Real-IP $remote_addr;  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  proxy_set_header X-Forwarded-Proto $scheme;  proxy_connect_timeout 5;}
```



跨域相关的配置，主要是下面这部分：



- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
  add_header 'Access-Control-Allow-Origin' $http_origin;  add_header 'Access-Control-Allow-Credentials' 'true';  add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';  add_header 'Access-Control-Allow-Headers' 'DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';  add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';  if ($request_method = 'OPTIONS') {    add_header 'Access-Control-Max-Age' 1728000;    add_header 'Content-Type' 'text/plain; charset=utf-8';    add_header 'Content-Length' 0;    return 204;  }
```



下面简单讲解一下，以便大家配置成功！

**1、Access-Control-Allow-Origin**，这里使用变量 $http_origin取得当前来源域，大家说用“*”代表允许所有，我实际使用并不成功，原因未知；



**2、Access-Control-Allow-Credentials**，为 true 的时候指请求时可带上Cookie，自己按情况配置吧；



**3、Access-Control-Allow-Methods**，OPTIONS一定要有的，另外一般也就GET和POST，如果你有其它的也可加进去；



**4、Access-Control-Allow-Headers，**这个要注意，里面一定要包含自定义的http头字段（就是说前端请求接口时，如果在http头里加了自定义的字段，这里配置一定要写上相应的字段），从上面可看到我写的比较长，我在网上搜索一些常用的写进去了，里面有“web-token”和“app-token”，这个是我项目里前端请求时设置的，所以我在这里要写上；



**5、Access-Control-Expose-Headers，**可不设置，看网上大致意思是默认只能获返回头的6个基本字段，要获取其它额外的，先在这设置才能获取它；



**6、语句“ if ($request_method = 'OPTIONS') { ”，**因为浏览器判断是否允许跨域时会先往后端发一个 options 请求，然后根据返回的结果判断是否允许跨域请求，所以这里单独判断这个请求，然后直接返回；



好了，按我上面配置基本都可使用（有些朋友确定只百度复制了两行，然后说配置好了，跟前端朋友互扯），



下面发一个实际配置供参考，我做了少量更改，如下：





- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
server {  listen       80;  server_name  xxx.com;   location /xxx-web/papi {    add_header 'Access-Control-Allow-Origin' $http_origin;    add_header 'Access-Control-Allow-Credentials' 'true';    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';    add_header 'Access-Control-Allow-Headers' 'DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';    add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';    if ($request_method = 'OPTIONS') {      add_header 'Access-Control-Max-Age' 1728000;      add_header 'Content-Type' 'text/plain; charset=utf-8';      add_header 'Content-Length' 0;      return 204;    }    root   html;    index  index.html index.htm;    proxy_pass http://127.0.0.1:7071;    proxy_set_header Host $host;    proxy_set_header X-Real-IP $remote_addr;    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;    proxy_set_header X-Forwarded-Proto $scheme;    proxy_connect_timeout 5;  }   location /xxx-web {    add_header 'Access-Control-Allow-Origin' $http_origin;    add_header 'Access-Control-Allow-Credentials' 'true';    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';    add_header 'Access-Control-Allow-Headers' 'DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';    add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';    if ($request_method = 'OPTIONS') {      add_header 'Access-Control-Max-Age' 1728000;      add_header 'Content-Type' 'text/plain; charset=utf-8';      add_header 'Content-Length' 0;      return 204;    }    root   html;    index  index.html index.htm;    proxy_pass http://127.0.0.1:8080;    proxy_set_header Host $host;    proxy_set_header X-Real-IP $remote_addr;    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;    proxy_set_header X-Forwarded-Proto $scheme;    proxy_connect_timeout 5;  }   location / {    root   /var/www/xxx/wechat/webroot;    index  index.html index.htm;  }   error_page   500 502 503 504  /50x.html;  location = /50x.html {    root   html;  }}
```





![写代码的渣渣鹏](http://mmbiz.qpic.cn/mmbiz_png/Kp4CFraY7DjqmY12sz2MDPOyJwgtWxCyxXyd7bJLkHQZAee6dgZ4vsKlzU3ib4icg6xpYFOpcibjZIWrj52ABEUibg/0?wx_fmt=png)

**写代码的渣渣鹏**

关注我，学好Java.让Spring Boot, 微服务,高并发，包括多线程、JVM、Spring Cloud不再有难题，让java不再难懂。

43篇原创内容



公众号

阅读 894

分享收藏

赞10在看6

写下你的留言