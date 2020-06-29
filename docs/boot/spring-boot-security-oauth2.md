---
title: Spring Security OAuth2实战
tags: oauth
categories: spring-boot
top: 73
date: 2019-04-22 12:55:06
---


在前面我们讲了：[Spring Security OAuth2概念引入](https://www.tycoding.cn/2019/04/22/boot/spring-boot-security-oauth2-start/) 这次我们讲解如何在Spring Boot中使用Spring Security OAuth2搭建权限脚手架项目。

:tada: :tada: :tada: **这里有丰富的 Spring 框架学习案例**

**仓库地址：[spring-learn](https://github.com/TyCoding/spring-learn)**
**欢迎star、fork，给作者一些鼓励**

<!--more-->

## 写在前面

如果你还不了解OAuth2，可以去查阅一些资料:

* [Spring Security OAuth2概念引入](https://www.tycoding.cn/2019/04/22/boot/spring-boot-security-oauth2-start/)
* [OAuth2官网](https://tools.ietf.org/html/rfc6749)
* [Spring Boot整合Spring Security](https://tycoding.cn/2019/04/11/boot/spring-boot-security/)

本次博文使用SpringBoot-2.x 和 Spring-Security-oauth2-2.x

## 引入依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependency>
    <groupId>org.springframework.security.oauth.boot</groupId>
    <artifactId>spring-security-oauth2-autoconfigure</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## Security配置

通常，我们直接创建一个配置类继承`WebSecurityConfigurerAdapter`类，重写其中的配置方法即可，并且需要`@EnableWebSecurity`注解标记即可：

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withUsername("user_1").password("123456").authorities("USER").build());
        return manager;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/hello")
                .permitAll()
                .anyRequest()
                .authenticated()

                .and()
                .csrf().disable();
    }

    @Bean
    @Override
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }
}
```

**注意**

* 这里重写了`userDetailsService()`方法，并`UserDetailsService`的一个实现类`InMemoryDetailsManager`将用户数据储存在内存中，其中用户名user_1(`withUsername`)、密码123456(`password`)、权限USER(`authorities`)。当然这里仅是为了模拟，实际开发中应该从数据库中读取用户数据。因此上述方式并不是唯一的，我们仅需要return一个实现了`UserDetailsService`接口的实现类即可。
* 重写`configure(HttpSecurity http)`方法，自定义一些请求过滤配置，这里开放了`/hello`接口，其他接口都被拦截。
* 最后一定要重写`authenticationManager()`方法，注入`AuthenticationManager`接口，目的是启用密码模式。如果不加这个配置很可能出现`Unsupported grant type:password`错误

## Security-OAuth2配置

在[Spring Security OAuth2概念引入](https://www.tycoding.cn/2019/04/22/boot/spring-boot-security-oauth2-start/)一文中我们已经了解了OAuth2的两个核心就是:1.认证服务器、2.资源服务器。那么这里就是配置OAuth2的认证服务器`AuthorizationServer`

通常我们直接创建一个配置类继承`AuthorizationServerConfigurerAdapter`类，并重写其中的方法完成配置，并且用`@EnableAuthorizationServer`注解标记即可：

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory().withClient("client_1")
                .resourceIds("info")
                .authorizedGrantTypes("client_credentials", "password", "refresh_token")
                .scopes("select")
                .authorities("client")
                .secret("123456");
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.authenticationManager(authenticationManager);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients();
        security.checkTokenAccess("permitAll()");
    }
}
```

**注意**

* 重写`configure(ClientDetailsServiceConfigurer clients)`方法，配置客户端的信息，这里仍然将客户端数据储存在内存中，实际开发中应该从数据库中读取客户端数据：
    * `widthClient()`客户端ID，相当于username
    * `authorizedGrantTypes()`客户端访问模式，客户端模式(`Client_credentials`)、密码模式(`password`)、授权码模式(`authorization_code`)
    * `scopes`: 客户端域，限制客户端的访问范围
    * `authorities()`: 客户端的权限
    * `secret()`: 客户端的安全码，相当于password
* 重写`configure(AuthorizationServerEndpointsConfigurer endpoints)`，将`AuthenticationManager`设置进去，支持密码模式的授权
* 重写`configure(AuthorizationServerSecurityConfigurer security)`，支持允许表单认证登录。配置checkTokenAccess为`permitAll()`，允许所有客户端发送请求，避免Spring Security拦截。默认是`denyAll()`

## Resource Server配置

上面Security-OAuth2配置，其实也就是`Authentication Server`认证服务的配置，但截至到目前，我们还未考虑过`Resource Server`资源服务配置，但对于我们目前的业务而言也不需要对Resource Server做什么自定义配置，如果需要按照Authentication Server的配置类似食用即可：

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
}
```



## PasswordEncoder配置

你可能会在搭建上述环境后启动项目还是报错：

```java
java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
```

原因就是在Spring Security新版本中，必须手动配置`PasswordEncoder`密码校验服务，在 [SpringBoot整合SpringSecurity](https://tycoding.cn/2019/04/11/boot/spring-boot-security/) 一文中也讲过。所以这里我们简单配置一个类，不做密码加密

```java
@Component
public class PasswordEncoderImpl implements PasswordEncoder {
    @Override
    public String encode(CharSequence charSequence) {
        return charSequence.toString();
    }

    @Override
    public boolean matches(CharSequence charSequence, String s) {
        return s.equals(charSequence.toString());
    }
}
```

## 测试

模拟创建Controller接口：

```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, this is test interface";
    }

    @GetMapping("/info/{name}")
    public String info(@PathVariable String name) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        return "hello " + name + ", this your info";
    }
}
```

访问`localhost:8080/hello`:

![](http://cdn.tycoding.cn/20200629092110.png)

不拦截，正常。

访问：`localhost:8080/info/tycoding`:

![](http://cdn.tycoding.cn/20200629092113.png)

为授权，被拦截，正常。

### `/oauth/token`获取Token

在Postman上测试`localhost:8080/oauth/token?username=user_1&password=123456&grant_type=password`接口：

![](http://cdn.tycoding.cn/20200629092117.png)

可以看到依然被拦截，这是因为Spring Security配置中默认食用Basic认证方式。我们直接将这个请求粘贴到浏览器上：

![](http://cdn.tycoding.cn/20200629092121.png)

要解决这个问题，Postman提供模拟Basic登陆的效果：

![](http://cdn.tycoding.cn/20200629092124.png)

### 使用Token访问受保护的资源

按照上述方式获取到了`access_token`值，在浏览器上直接访问`localhost:8080/info/tycoding?access_token=xxx`:

![](http://cdn.tycoding.cn/20200629092129.png)

看到，还是被拦截，按道理，既然第三方应用拿到了Token值，通过这个Token值应该可以获取到受保护的资源。

这时就需要配置Resource Server了，前面我们仅仅提到了，并没有配置，那么这里创建Resource Server的配置：

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {
}
```

如果启用了`@EnableResourceServer`的配置，那么应该重写`configure(HttpSecurity http)`方法，和`@EnableWebSecurity`标记类的配置类似，我们这里要配置不需要拦截的请求。如果你仅仅是在`@EnableWebSecurity`配置类中配置了`/hello`请求不拦截，而你启用了`@EnableResourceServer`配置，但并没有配置`HttpSecurity`的过滤配置，那么默认OAuth还是拦截所有，直接访问`/hello`反而也被拦截到：

![](http://cdn.tycoding.cn/20200629092133.png)

所以，应该配置为：

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Override
    public void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/hello")
                .permitAll()
                .anyRequest()
                .authenticated()

                .and()
                .csrf().disable();
    }
}
```



重新运行项目，获取到Token值，直接去浏览器访问：`localhost:8080/info/tycoding?access_token=xx`:

![](http://cdn.tycoding.cn/20200629092137.png)

出现这个错误可能是你的Token值需要更新了。

![](http://cdn.tycoding.cn/20200629092140.png)

正常访问

### `/oauth/check_token`

这个接口用于校验此token是否过期

如果错误或过期：

![](http://cdn.tycoding.cn/20200629092143.png)

如果正确为过期：

![](http://cdn.tycoding.cn/20200629092147.png)

显示该Token对应第三方应用和用户的数据。

## Error

以下是我们在整合Spring Security OAuth2常见的错误：

### Request method &#39;GET&#39; not supported"

```json
{
  "error": "method_not_allowed",
  "error_description": "Request method &#39;GET&#39; not supported"
}
```

1. 错误描述

通常是因为我们直接使用浏览器请求，那么就采用的GET请求

2. 解决方案

使用Postman等工具测试接口，发送POST请求即可

### Unsupported grant type: password

```json
{
    "error": "unsupported_grant_type",
    "error_description": "Unsupported grant type: password"
}
```

1. 错误描述

出现这个错误的原因通常是因为你没有配置支持OAuth2密码模式，这个常出现

2. 解决方案

Spring Security配置中将`AuthenticationManager`类注入到Spring IOC容器中：

```java
@Bean
@Override
protected AuthenticationManager authenticationManager() throws Exception {
    return super.authenticationManager();
}
```

Security OAuth认证服务器(`@EnableAuthorizationServer`)中配置`AuthenticationManager`:

```java
@Autowired
private AuthenticationManager authenticationManager;

@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    endpoints.authenticationManager(authenticationManager);
}
```

### Missing grant type

```json
{
    "error": "invalid_request",
    "error_description": "Missing grant type"
}
```

1. 错误描述

通产是请求URL中没有配置`grant_type`的值

2. 解决方案

请求中配置`grant_type`类型，比如：`grant_type:password`

### 401 Unauthorized

```json
{
    "timestamp": "2019-04-22T07:18:52.341+0000",
    "status": 401,
    "error": "Unauthorized",
    "message": "Unauthorized",
    "path": "/oauth/token"
}
```

1. 错误描述

如果是使用的Postman工具请求的URL，报这个错误，通常是因为配置的Spring Security拦截了这个请求，并启用了Basic的验证方式，我们直接将这个URL放在浏览器上访问，则会弹出Basic密码验证框：

![](http://cdn.tycoding.cn/20200629092154.png)

正确输入客户端(Client)的账户（clientId）、密码(secret)，即可访问。

2. 解决方案

要解决这个问题，就要考虑怎么能让Postman请求不经过Basic登录框这个验证方式，这就需要知道Basic的加密方式。其实Basic登录框最终是将输入的用户名和密码通过Base64加密，然后在前面加上`Basic base64`，然后去请求这个URL接口，所以我们需要在 [在线加密解密网站](http://tool.oschina.net/encrypt?type=3) 将Client用户名和密码按照：`name:password`格式加密，最终在Postman中设置请求头`Authorization: Basic xxx`，这个xxx是按照`name:password`格式加密的数据：

![](http://cdn.tycoding.cn/20200629092157.png)

### Full authentication is required to access this resource

```json
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```

1. 错误描述

如果此时后台也报错：`There is no PasswordEncoder mapped for the id "null"`，通常就是因为没有配置密码校验服务的原因。

**产生这个错误的原因有很多**

2. 解决方案

注入任意一个实现了`PasswordEncoder`接口的实现类。





<br/>

# 交流

QQGroup：671017003   

WeChatGroup:  关注公众号查看

# 联系

- [Blog@TyCoding's blog](http://www.tycoding.cn)
- [GitHub@TyCoding](https://github.com/TyCoding)
- [ZhiHu@TyCoding](https://www.zhihu.com/people/tomo-83-82/activities)

