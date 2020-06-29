---
title: Spring Security OAuth2数据持久化
tags: oauth
categories: spring-boot
top: 74
date: 2019-04-23 14:55:52
---


在上一篇文章 [Spring Security OAuth2实战](https://tycoding.cn/2019/04/22/boot/spring-boot-security-oauth2/) 中我们尝试配置了一个最基础的Security OAuth2环境，但其中的用户数据都是模拟储存在内存中的，而实际开发中，这些数据应该是从数据库中获取的。这次我们就结合数据库、Redis进一步配置Security OAuth2环境。

:tada: :tada: :tada: **这里有丰富的 Spring 框架学习案例**

**仓库地址：[spring-learn](https://github.com/TyCoding/spring-learn)**
**欢迎star、fork，给作者一些鼓励**

<!--more-->

## 引入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.security.oauth.boot</groupId>
        <artifactId>spring-security-oauth2-autoconfigure</artifactId>
        <version>2.1.4.RELEASE</version>
    </dependency>

    <!-- 将token存储在redis中 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
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

**注意** 在使用的`spring-security-oauth2-autoconfigure`依赖和`spring-boot-starter-data-redis`实现将Token持久化到Redis数据库如果会遇到错误：

```java
Caused by: java.lang.NoSuchMethodError: org.springframework.data.redis.connection.RedisConnection.set([B[B)V
```

报错大概就是说没有`set`方法，导致Token数据不能持久化到Redis中，通常是因为版本较老的原因，这好像是老版本的一个BUG，但是这在新版本中已经修复的。

## application.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/springboot_oauth2?useUnicode=true&characterEncoding=UTF-8
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

  redis:
    host: 127.0.0.1
    port: 6379

#mybatis配置
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: cn.tycoding.entity

# 打印sql
logging:
  level:
    cn.tycoding.mapper: DEBUG
```

创建数据库：

```sql
CREATE DATABASE springboot_oauth2 CHARSET utf8;
CREATE TABLE `sys_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `username` varchar(255) DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) DEFAULT NULL COMMENT '密码',
  `salt` varchar(255) DEFAULT NULL COMMENT '随机盐',
  `authorities` varchar(255) DEFAULT NULL COMMENT '模拟权限列表',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
INSERT INTO `sys_user` VALUES(1, 'tycoding', '123', '', 'ADMIN');
```

## findByUsername

### Entity

```java
public class SysUser implements Serializable {

    /**
     * ID主键
     */
    private Long id;

    /**
     * 用户名
     */
    private String username;

    /**
     * 密码
     */
    private String password;

    /**
     * 随机盐
     */
    private String salt;

    /**
     * 权限，这里是方便模拟，其实应该从另一张表中获取
     */
    private String authorities;
}
```

### Mapper

```java
@Mapper
public interface UserMapper {

    @Select("select * from sys_user where username = #{username}")
    SysUser findByUsername(String username);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="cn.tycoding.mapper.UserMapper">
</mapper>
```

### Service

```java
public interface UserService {

    /**
     * 根据用户名查询
     *
     * @param username
     * @return
     */
    SysUser findByUsername(String username);
}
```

### ServiceImpl

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public SysUser findByUsername(String username) {
        return userMapper.findByUsername(username);
    }
}
```

### Controller

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



### 结

自此，`findByUsername`根据用户名查询用户数据的逻辑已经完成，很简单的逻辑，不再介绍

## Security Config

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 通过HttpSecurity实现Security的自定义过滤配置
     *
     * @param http
     * @throws Exception
     */
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

    /**
     * 注入AuthenticationManager接口，启用OAuth2密码模式
     *
     * @return
     * @throws Exception
     */
    @Bean
    @Override
    protected AuthenticationManager authenticationManager() throws Exception {
        return super.authenticationManager();
    }
}
```

在之前我们是重写`userDetailsService()`方法，模拟在内存中创建一个用户，然后将这个配置方法注入到Spring IOC容器中。但其实这种方式不是必须的，我们只需要向Spring IOC容器中注入一个实现了`UserDetailsService`接口的实现类就行了。而本例中我们从数据库中查询用户数据，所以这里不再模拟向内存中储存用户数据。

## AuthorizationServerConfig

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory().withClient("client_1")
                .authorizedGrantTypes("client_credentials", "password", "refresh_token")
                .scopes("select")
                .authorities("client")
                .secret("123456");
    }

    /**
     * 认证服务端点配置
     * 密码模式下需要配置认证管理器AuthenticationManager
     *
     * @param endpoints
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                .userDetailsService(userDetailsService)
                .tokenStore(new RedisTokenStore(redisConnectionFactory))
                .authenticationManager(authenticationManager);
    }

    /**
     * 认证安全检查流程配置
     * 配置checkTokenAccess为`permitAll()`，允许所有客户端发送请求，避免Spring Security拦截。默认是`denyAll()`，
     *
     * @param security
     * @throws Exception
     */
    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
        security.allowFormAuthenticationForClients();
        security.checkTokenAccess("permitAll()");
    }
}
```

**解释**

* 注入`RedisConnectionFactory`用户OAuth2将Token数据储存到Redis数据库中，我们并不需要配置具体的实现，Security-OAuth2本身提供了很多种方式储存Token数据

![](http://cdn.tycoding.cn/20200629091849.png)

* 注入`UserDetailsService`，他用于初始化用户数据，我们仅仅在这里注入了`UserDetailsService`接口，而需要在其他地方注入实现了`UserDetailsService`接口的实现类。这样Security OAuth2会自动使用这个实现类中的配置。
* 在`configure(AuthorizationServerEndpointsConfigurer endpoints)`中配置`UserDetailsService`，配置`tokenStore()`，使用Security OAuth的`RedisTokenStore`实现，将`RedisConnectionFactory`连接工厂注入即可。
* 在上面`configure(ClientDetailsServiceConfigurer clients)`客户端配置中仍然直接在内存中创建一个客户端数据，实际上，客户端的数据应该是第三方提供的，比如微信、QQ等，而在这里都是我们个人使用的，暂时不处理第三方应用。

## ResourceServerConfig

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

## UserDetailsServiceImpl

上面介绍了，我们需要向Spring IOC容器中注入一个实现了`UserDetailsService`接口的Bean，这样Security OAuth2在初始化用户数据或身份校验时就能自动使用该Bean初始化用户数据：

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserService userService;

    /**
     * 实现UserDetailsService中的loadUserByUsername方法，用于加载用户数据
     *
     * @param username
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        SysUser user = userService.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        //用户权限列表。这里是为了方便模拟，实际应该从权限表中查询用户的权限列表
        Collection<? extends GrantedAuthority> authorities = AuthorityUtils.createAuthorityList(user.getAuthorities());

        return new AuthUser(
                user.getId(),
                user.getUsername(),
                user.getPassword(),
                true,
                true,
                true,
                true,
                authorities);
    }
}
```

**解释**

可以看`UserDetailsService`接口：

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
}
```

可以看到这个接口直接一个方法，这个方法用于根据用户名加载用户数据，最后需要返回一个`UserDetails`对象，可以看到`UserDetailsService`接口默认的实现：

![](http://cdn.tycoding.cn/20200629091857.png)

如上，可以通过`InMemoryUserDetailsManager`从内存中获取数据、通过`JdbcUserDetailsManager`通过封装的JDBC操作从数据库中获取数据…。

> 重写这个方法，该怎么返回一个`UserDetails`对象呢？

查看`UserDetails`对象源码：

```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword(); //密码
    String getUsername(); //用户名
    boolean isAccountNonExpired(); //账户是否过期
    boolean isAccountNonLocked(); //账户是否锁定
    boolean isCredentialsNonExpired(); //账户凭证时候过期
    boolean isEnabled(); //账户是否可用
}
```

可以看到这个接口对象中封装了常见用户数据的获取方法。那么我们应该创建一个类实现这个接口，实现其中的方法并将正确的用户数据封装进去。这里，我们直接创建`AuthUser`类继承Spring Security内部实现了`UserDetails`接口的一个类`org.springframework.security.core.userdetails.User`:

```java
public class User implements UserDetails, CredentialsContainer { ... }
```

`User`类中已经提供了很多实现方法，我们也没必要再复写：

```java
public class AuthUser extends User {

    /**
     * 用户ID
     */
    private Long id;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public AuthUser(Long id, String username, String password, boolean enabled, boolean accountNonExpired, boolean credentialsNonExpired, boolean accountNonLocked, Collection<? extends GrantedAuthority> authorities) {
        super(username, password, enabled, accountNonExpired, credentialsNonExpired, accountNonLocked, authorities);
        this.id = id;
    }
}
```

构造方法`public AuthUser()`中包含了`UserDetails`接口中所涉及的属性值。下面我们把重点放在`authorities`这个属性上，他包含了用户的权限集合值，通常我们会建立五张表：1.用户表、2.角色表、3.权限表、4.用户角色表、5.角色权限表，以此来获取用户的角色、权限列表。这种方式方便维护，设计清晰。但是在本例中我们仅仅模拟在用户表中添加`authorities`这个字段，大家明白意思就好。

发现`authorities`这个属性是一个`Collection<? extends GrantedAuthority>`对象，也就是他是一个实现了`Collection`接口的集合，并且这个集合有一个泛型，标明这个接口存放的是实现了`GrantedAuthority`接口的对象。那么，总结而言，我们需要：

1. 一个实现了`Collection`接口的集合，作为权限集合
2. 一堆实现了`GrantedAuthority`接口的对象，作为权限对象

```java
Collection<? extends GrantedAuthority> authorities = AuthorityUtils.createAuthorityList(user.getAuthorities());
```

没错，Spring Security内部提供了一个工具类用于提供一个`List<GrantedAuthority>`对象：

```java
public abstract class AuthorityUtils {
    
    public static List<GrantedAuthority> commaSeparatedStringToAuthorityList(String authorityString) {
        ...
    }
    public static Set<String> authorityListToSet(Collection<? extends GrantedAuthority> userAuthorities) {
       ...
    }
    public static List<GrantedAuthority> createAuthorityList(String... roles) {
		...
    }
}
```

最后，我们准备好了`UserDetails`接口所需的全部数据，直接`return AuthUser(…)`即可。

## 测试

打开本地Redis

![](http://cdn.tycoding.cn/20200629091903.png)

启动项目，观察断点:

首先，观察启动项目时`configure(AuthorizationServerEndpointsConfigurer endpoints)`方法：

![](http://cdn.tycoding.cn/20200629091909.png)

此时，我们`@Autowire`的`UserDetailsService`对象已经包含了我们设置的一写逻辑 ==> 通过Mybatis调用`findByUsername`方法查询数据库得到用户对象。

访问：`localhost:8080/hello`:

![](http://cdn.tycoding.cn/20200629091914.png)

访问：`localhost:8080/info/tycoding`:

![](http://cdn.tycoding.cn/20200629091919.png)

使用Postman工具访问`localhost:8080/oauth/token`接口获取Token：

![](http://cdn.tycoding.cn/20200629091922.png)

注意Basic认证：

![](http://cdn.tycoding.cn/20200629091926.png)

查看断点，经过自定义实现了`UserDetailsService`接口的实现类：

![](http://cdn.tycoding.cn/20200629091930.png)

这里根据`findByUsername`查询到了用户数据，最终将数据封装到`AuthUser`对象中再返回即可：

![](http://cdn.tycoding.cn/20200629091937.png)

再次访问：`localhost:8080/info/tycoding?access_token=xxx`:

![](http://cdn.tycoding.cn/20200629091942.png)

查看Redis，可以看到认证数据已经储存在Redis中：

![](http://cdn.tycoding.cn/20200629091946.png)



<br/>

# 交流

QQGroup：671017003   

WeChatGroup:  关注公众号查看

# 联系

- [Blog@TyCoding's blog](http://www.tycoding.cn)
- [GitHub@TyCoding](https://github.com/TyCoding)
- [ZhiHu@TyCoding](https://www.zhihu.com/people/tomo-83-82/activities)

