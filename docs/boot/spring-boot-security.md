---
title: （十五）SpringBoot2.x整合SpringSecurity
date: 2019-04-11 09:31:49
tags: springboot
categories: springboot
top: 70
---



# Spring Boot整合SpringSecurity

:tada: :tada: :tada: **这里有丰富的 Spring 框架学习案例**

**仓库地址：[spring-learn](https://github.com/TyCoding/spring-learn)**
**欢迎star、fork，给作者一些鼓励**

> 写在前面

之前都是在用Shiro做权限管理框架，并且写过一个不错的项目：[permission](https://github.com/TyCoding/permission/) (欢迎star,fork)。

这次我们学习下Spring Security权限框架。注意，这里使用的Spring Boot-2.1.4版本，那么Spring Security也是最新的5.1.5

<!--more-->

## 起步

### 依赖导入

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 创建测试接口

在`/cn/tycoding/controller`下创建`UserController.java`:

```java
@Controller
public class UserController {

    @GetMapping("/test")
    @ResponseBody
    public String test() {
        return "Hello I'm TyCoding, this is Login test";
    }
}
```

### 测试

启动项目，在浏览器上访问：`127.0.0.1:8080/test`接口： 

![](http://tycoding.cn/imgs/20200629092351.png)

> 可以看到，URL立即跳转到`/login`，并且，这个接口和页面并不是我们写的。

所以，Spring Security帮我们写了一个默认的登录页面，并且项目引入了Spring Security，Spring Security就默认配置拦截所有URL请求。

> 观察启动项目的控制台log

可看到类似这段日志：

```
Using generated security password: 8b0e38a6-6af6-40cd-8882-a8c697fd803e
```

这个就是Spring Security默认进行URL拦截并默认提供的登录密码，用户名是`user`。在登录页面上输入`user`和`8b0e38a6-6af6-40cd-8882-a8c697fd803e`，登录成功:

![](http://tycoding.cn/imgs/20200629092359.png)

### 简单的配置

新建`cn/tycoding/config`，创建`WebSecurityConfig.java`继承`WebSecurityConfigurerAdapter`，并重写`void configure(HttpSecurity http)`方法：

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        /**
         * fromLogin(): 表单认证
         * httpBasic(): 弹出框认证
         * authorizeRequests() 身份认证请求
         * anyRequest(): 所有请求
         * authenticated(): 身份认证
         */

//        http.formLogin()
        http.httpBasic()
                .and()
                .authorizeRequests()
                .anyRequest()
                .authenticated();
    }
}
```

最开始演示的是Spring Security默认的表单认证方式，当我们修改为`http.httpBasic()`后：

![](http://tycoding.cn/imgs/20200629092403.png)

### 过滤器

Spring Security基础的过滤器：

![](http://tycoding.cn/imgs/20200629092407.png)

当请求经过Spring Security，就会经过Spring Security的一系列过滤器，当都满足过滤器的条件，最终通过请求，响应请求。其中：

* `UsernamePassword`是最前端的过滤器，用于处理基础的表单login请求
* 图中绿色的是用户可定制的过滤器，而后面的非绿色过滤器是用户不能控制的
* `FilterSecurityInterceptor`是最后段的过滤器，用于最终处理所有的请求过滤


## 进阶

### 自定义登录实现

#### 1.自定义登录接口

在前面看到，Spring Security默认为拦截的URL提供登录页面，并内部提供一个接口用于处理默认的登录请求。但这在实际项目中显然是不行了，所以我们要定制化登录接口。

在`cn/tycoding/config/service/`下创建`UserDetailsServiceImpl.java`并实现`UserDetailsService`接口：

```java
@Slf4j
@Component
public class UserDetailsServiceImpl implements UserDetailsService {

    /**
     * Spring Security接收login请求调用UserDetailsService这个接口中的loadUserByUsername方法
     * loadUserByUsername根据传进来的用户名进行校验工作，最后将查询到的用户信息封装到UserDetails这个接口的实现类中
     *
     * @param username 用户名
     * @return
     * @throws UsernameNotFoundException 登录异常类
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        log.info("登录，用户名：{}", username);
        //根据用户名查询用户数据，比如：从数据库中查询
        return new User(username, "123456", AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
```

`UserDetailsService`接口的唯一一个方法`loadUserByUsername`，负责接收登录请求传递的用户名，并根据用户名进行身份校验（比如查询数据库）。最后该方法返回一个封装了用户身份信息的`UserDetails`实现类对象。
这里我们返回一个Spring提供的`UserDetails`的实现类`User`（当然并不必须，我们只需要返回实现了`UserDetails`接口的实现类即可）：

```java
public User(String username, String password, Collection<? extends GrantedAuthority> authorities) {}

public User(String username, String password, boolean enabled, boolean accountNonExpired, boolean credentialsNonExpired, boolean accountNonLocked, Collection<? extends GrantedAuthority> authorities) {}
```

该实现类有两个带参构造方法。其中`authorities`参数是携带了用户权限集合。为了模拟效果，我们写死数据。

##### 测试

启动项目，访问`localhost:8080/test`，URL跳转到`/login`:

根据我们写死的密码，在页面上输入: username:`tycoding`, password:`123456`。点击登录发现页面刷新了，但并未跳转到`/test`，查看控制台日志：

```
java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"
```

原来在因为使用了Spring Security-5.x版本，需要手动提供一个`PasswordEncoder`实现类类，进行密码校验。那么我们在`cn/tycoding/config/`下创建`PasswordEncoderImpl`类并实现`PasswordEncoder`接口:

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

重启项目，输入用户名、密码，正常访问。

#### 2.自定义密码加密

上面我们创建了`PasswordEncoderImpl`实现类，实现了`PasswordEncoder`接口，在`PasswordEncoderImpl`中可以进行一些密码加密(`encode`)、密码校验(`matches`)操作。

而我们上面创建的`PasswordEncoderImpl`并没有进行任何加密、校验，仅仅是`toString()`的比对。若要实现密码的加密，我们可以直接使用Spring提供的一个`BCryptPasswordEncoder`类，他实现了`PasswordEncoder`接口，并重写了加密、校验方法。

修改`WebSecurityConfig.java`类：

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    //...
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

修改`UserDetailsServiceImpl`类：

```java
@Slf4j
@Component
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private PasswordEncoder passwordEncoder;

    /**
     * Spring Security接收login请求调用UserDetailsService这个接口中的loadUserByUsername方法
     * loadUserByUsername根据传进来的用户名进行校验工作，最后将查询到的用户信息封装到UserDetails这个接口的实现类中
     *
     * @param username 用户名
     * @return
     * @throws UsernameNotFoundException 登录异常类
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        String password = passwordEncoder.encode("123456");
        log.info("登录，用户名：{}, 密码：{}", username, password);
        //根据用户名查询用户数据，比如：从数据库中查询
        return new User(username, password, AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
```

注意，此时使用了我们像Spring中注入了`BCryptPasswordEncoder`类，他实现了`PasswordEncoder`接口，那么我们再次调用`PasswordEncoder`中的`encoder`方法其实会调用其实现类`BCryptPasswordEncoder`中的`encoder`进行密码加密操作。

所以，我们需要将之前注入的`PasswordEncoderImpl`类去掉。

> 测试

重启项目，访问`localhost:8080/test`，在登录页中输入用户名、密码。可以看到控制台打印类似：

```
登录，用户名：tycoding, 密码：$2a$10$WAri0CDq74nwaS/L1iU03en/1bgvwTZGsudVczRt5MNA/pO4nTnaa
```

密码被加密了。并且每次登录，这个密码都会不同，原理和Shiro类似，Spring Security在进行密码加密的时候，也生成了一份随机salt，最终加密的密码=密码+随机salt。

如果我们需要冲数据库中读取数据，这个salt应该直接从数据库中取，而不是再次随机生成。

#### 3.自定义登录请求

在上面的案例中，我们完成了自定义登录接口、自定义密码加密，但始终都是在Spring Security提供的登录页面操作的，下面我们实现自定义登录请求：

> 1.修改控制器类

```java
@Slf4j
@Controller
public class UserController {

    @GetMapping("/test")
    @ResponseBody
    public String test() {
        return "Hello I'm TyCoding, this is Login test";
    }

    /**
     * 页面路由，当使用GET请求访问/login接口，会自动跳转到`/templates/login.html`页面
     *
     * @return login登录页面路由地址
     */
    @GetMapping("/login")
    public String login() {
        return "login";
    }

    /**
     * 页面路由，当使用GET请求访问/login接口，会自动跳转到`/templates/index.html`页面
     *
     * @return index首页面路由地址
     */
    @GetMapping("/index")
    public String index() {
        return "index";
    }
}
```

> 2.创建`index.html`和`login.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登录页</title>
</head>
<body>
<form action="/auth/login" method="post">
    <input name="username" placeholder="Username"><br/>
    <input name="password" type="password" placeholder="Password"><br/>
    <input type="submit" value="Login">
</form>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
<h1>首页</h1>
</body>
</html>
```

> 3.修改`WebSecurityConfig`

```java
@Configuration
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {

        /**
         * fromLogin(): 表单认证
         * httpBasic(): 弹出框认证
         * authorizeRequests() 身份认证请求
         * anyRequest(): 所有请求
         * authenticated(): 身份认证
         * loginPage(): 登录页面地址（因为SpringBoot无法直接访问页面，所以这通常是一个路由地址）
         * loginProcessingUrl(): 登录表单提交地址
         * .csrf().disable(): 关闭Spring Security的跨站请求伪造的功能
         */
        http.csrf().disable().formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/auth/login")
//        http.httpBasic()
                .and()
                .authorizeRequests()
                .antMatchers("/login").permitAll()
                .anyRequest()
                .authenticated();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**注意**

可以看到，上面我们定义了两个路由接口`/login`和`/index`分别路由到`/login.html`和`/index.html`页面。**但是**，我们在login.html登录表单中定义的提交地址`/auth/login`并不在`UserController`中定义，而是在`WebSecurityConfig`的`loginProcessingUrl`中配置了，为什么呢？

回想之前没有进行定制化操作时，登录系统，Spring Security默认拦截所有请求，并将URL 302重定向到`/login`这个地址，并提供了一个表单登录页面，我们使用Spring Security初始化的用户名和密码即可登录。显然，这个登录页面的地址和表单提交地址在Spring Security中都提供好了。

所以，这里我们直接使用Spring Security提供的登录表单提交地址，即可实现Spring Security的登录校验，这和Shiro的`Subject.login()`是一个道理，最终都是要将登录数据交由权限框架进行身份校验处理。

在前面Spring Security的过滤器链中提到了`UsernamePasswordAuthenticationFilter`是最前端的过滤器，用于处理表单提交数据：

```java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
    public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";
    private String usernameParameter = "username";
    private String passwordParameter = "password";
    private boolean postOnly = true;

    public UsernamePasswordAuthenticationFilter() {
        super(new AntPathRequestMatcher("/login", "POST"));
    }
}
```

看源码得知`UsernamePasswordAuthenticationFilter`接收`/login` POST请求，那么我们只需要指定我们自己的登录表单最终提交到这个地址即可。所以我们在`WebSecurityConfig`中定义`loginProcessingUrl("/auth/login")`，当表单提交`/auth/login`这个地址时，其实最终提交到`UsernamePasswordAuthenticationFilter`的`/login`接口，交由`UsernamePasswordAuthenticationFilter`进行第一层身份过滤，然后经过一系列的过滤器，最判断如果条件满足就通过校验，跳转到目的请求。

#### 5.自定义登录请求状态

在未进行前后端分离时，通常我们的登录请求是同步的，登录成功或失败都是返回一个视图地址，这用上面的学习完全可以实现（配置`.loginPage()`即可）。但若前后端分离，登录请求通常是异步的，比如通过ajax请求，此时返回页面视图地址显然不合适。

如果需要自定义登录成功、失败的状态，我们仅需要了解两个handle处理器：`AuthenticationSuccessHandler`和`AuthenticationFailureHandler`。他们两个分别用于登录成功和登录失败的处理，我们仅需要创建两个类分别实现这两个接口然后自定义一些逻辑处理即可：

> 1.在`cn/tycoding/handler/`下创建`AuthenticationSuccessHandlerImpl`:

```java
@Slf4j
@Component
public class AuthenticationSuccessHandlerImpl implements AuthenticationSuccessHandler {

    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
        log.info("登录成功");
        httpServletResponse.setContentType("application/json;charset=utf-8");
        httpServletResponse.getWriter().write(objectMapper.writeValueAsString(authentication));
    }
}
```

> 2.创建`AuthenticationFailureHandlerImpl`:

```java
@Slf4j
@Component
public class AuthenticationFailureHandlerImpl implements AuthenticationFailureHandler {

    @Autowired
    private ObjectMapper objectMapper;

    @Override
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
        log.info("登录失败");
        httpServletResponse.setStatus(HttpStatus.INTERNAL_SERVER_ERROR.value());
        httpServletResponse.setContentType("application/json;charset=utf-8");
        httpServletResponse.getWriter().write(objectMapper.writeValueAsString(e));
    }
}
```

> 3.测试

启动项目，分别模拟登录成功和登录失败的情况。

![](http://tycoding.cn/imgs/20200629092421.png)

![](http://tycoding.cn/imgs/20200629092429.png)

-----

## 源码分析

### 身份校验流程

![](http://tycoding.cn/imgs/20200629092432.png)

![](http://tycoding.cn/imgs/20200629092438.png)

![](http://tycoding.cn/imgs/20200629092446.png)

![](http://tycoding.cn/imgs/20200629092459.png)

![](http://tycoding.cn/imgs/20200629092506.png)

![](http://tycoding.cn/imgs/20200629092510.png)

![](http://tycoding.cn/imgs/20200629092514.png)

![](http://tycoding.cn/imgs/20200629092518.png)

![](http://tycoding.cn/imgs/20200629092522.png)

![](http://tycoding.cn/imgs/20200629092527.png)

### 认证结果在多个请求中共享

通常保证认证信息能在多个请求获取，会将认证信息储存在Session中。

![](http://tycoding.cn/imgs/20200629092531.png)

![](http://tycoding.cn/imgs/20200629092535.png)

`SecurityContext`其实是`Authentication`的一个封装，而`SecurityContextHolder`是`ThreadLocal`的一个封装。

一般情况下一个请求和响应都是在一个线程中的，而通过`SecurityContextHolder`将认证信息放到了当前线程中，那么在任何使用该线程的地方调用`SecurityContextHolder`都能获取到认证数据。

那么最终是由`SecurityontextPersistenceFilter`使用`SecurityContextHolder`进行认证信息的读取。

![](http://tycoding.cn/imgs/20200629092540.png)

`SecurityontextPersistenceFilter`过滤器位于`UsernamePasswordAuthenticationFilter`过滤器前面，请求和响应都会经过这个过滤器。

* **请求**：检查Session，如果有认证信息，就把认证信息放到当前线程中
* **响应**：检查线程，如果有认证信息，就把认证信息放到Session中

### 获取当前登录用户信息

和Shiro不同，我们不需要写很长的代码用于获取当前用户信息，只需要简单的在请求接口参数列表中加入`Authentication`对象参数即可获取到封装了用户信息的`Authentication`对象。

修改`UserController.java`，创建一个新的接口：

```java
/**
 * 获取当前登录用户信息
 *
 * @param authentication 封装了当前登录用户信息的Authentication对象
 * @return 用户信息集合对象
 */
@GetMapping("/info")
@ResponseBody
public Object getCurrentUser(Authentication authentication) {
    return authentication;
}
```

![](http://tycoding.cn/imgs/20200629092544.png)



<br/>

# 交流

QQGroup：671017003   

WeChatGroup:  关注公众号查看

# 联系

- [Blog@TyCoding's blog](http://www.tycoding.cn)
- [GitHub@TyCoding](https://github.com/TyCoding)
- [ZhiHu@TyCoding](https://www.zhihu.com/people/tomo-83-82/activities)

