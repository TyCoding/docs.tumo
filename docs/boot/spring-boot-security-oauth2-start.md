---
title: Spring Security OAuth2概念引入
date: 2019-04-22 12:56:51
tags: oauth2
categories: spring-boot
top: 72
---



:tada: :tada: :tada: **这里有丰富的 Spring 框架学习案例**

**仓库地址：[spring-learn](https://github.com/TyCoding/spring-learn)**
**欢迎star、fork，给作者一些鼓励**

# Spring Security OAuth2概念引入

​	OAUTH协议为用户资源的授权提供了一个安全的、开放而又简易的标准。与以往的授权方式不同之处是OAUTH的授权不会使第三方触及到用户的帐号信息（如用户名与密码），即第三方无需使用用户的用户名与密码就可以申请获得该用户资源的授权，因此OAUTH是安全的。oAuth是Open Authorization的简写。

<!--more-->

上面摘选自百度百科。想象一下，我们开发了一个微信小程序，他的功能是可以批量修改(美化)用户相册中的照片，那么这小程序就要获取用户相册数据。如果按照传统方式直接让用户通过用户名密码登录该小程序，那么该小程序通过用户名密码就能获取到用户的全部数据，这就造成了密码泄露的风险，且想要弃用这个小程序只能修改密码。这种方式显然不好。

而OAuth协议的出现正是解决了这些问题。他将用户名密码授权方式改为通过`令牌`授权，第三方应用请求访问资源，资源所有者(用户)同意授权后，再次访问服务提供商(微信官方)去向认证服务器申请令牌(Token)，认证服务器同意后发放令牌(Token)，然后第三方应用再拿这个令牌(Token)去向资源服务器(微信官方)申请想要的资源，资源服务器验证通过后开放这个应用所需的资源。

我们可以以下图简单的了解下Oauth2

![](http://tycoding.cn/imgs/20200629092039.png)

 这种方式就完全避免了把用户名密码暴露给第三方应用的风险。

下面我们把重点放在右侧**资源提供商(Provider)**

# 服务提供商 Provider

Spring Security Oauth2的服务提供商实际上分为：

- 认证服务 Authorization Server
- 资源服务 Resource Server

上图中，我们把认证服务和资源服务放在一起，但实际上并不一定是这样的，资源服务可以放在不同的应用上，而且可以存在多个资源服务，他们共享同一个中央认证服务器。

获取令牌(Token)的请求以及其他OAuth内部的请求都会放在Spring Security的请求过滤器链中，但这些请求都不会被Spring Security拦截。

下面是配置一个认证服务必须实现的endpoints:

- `AuthorizationEndpoint`: 用来作为请求者获得授权的服务，默认URL是：`/oauth/authorize`
- `TokenEndpoint`: 用来作为请求者获取令牌(Token)的服务，默认URL是：`/oauth/token`

下面是配置一个资源服务必须实现的过滤器：

- `OAuth2AuthenticationProcessingFilter`: 用来作为令牌(Token)的一个处理流程过滤器。只有当过滤器通过后，请求者才能获取受保护的资源

## 认证服务器 Authorization Server

通过`EnableAuthorizationServer`注解配置

配置一个授权服务(即Authorization Server)，需要考虑集中授权类型(Grant Type)，不同授权类型的客户端(Client)可能提供了不同的方式获取令牌(Token)，为了实现并确定这几种授权，需要配置使用ClientDetailsService和TokenService来开启和禁用这几种授权机制。并且，在实际开发中，我们通常指定`authorizedGrantTypes`授权类型，Spring OAuth2提供了几种常用的授权模式：

- `authorization_code`: 授权码模式
- `implicit`: 简易模式(隐式授权类型)
- `password`: 密码模式
- `client_credentials`: 客户端模式
- `refresh_token`: 通过以上授权获取的令牌(Token)来刷新令牌(获取新的令牌)

通常，我们直接使用`@EnableAuthorizationServer`标记此服务是认证服务，然后继承`AuthorizationServerConfigurerAdapter`类来配置具体的认证授权服务。下面介绍一些常见的配置：

- `ClientDetailsServiceConfigurer`: 配置客户端的详情服务(ClientDetailsService)，客户端的信息在这里初始化。通常这些信息从数据库中读取
- `AuthorizationServerSecurityConfigurer`: 配置令牌端点(Token Endpoint)的安全约束
- `AuthorizationServerEndpointsConfigurer`: 配置授权(authorization)以及令牌(Token)的访问端点和令牌服务(Token services)

对应在`AuthorizationServerConfigurerAdapter`类中的方法：

```java
public class AuthorizationServerConfigurerAdapter implements AuthorizationServerConfigurer {
    public AuthorizationServerConfigurerAdapter() {
    }

    public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
    }

    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    }

    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
    }
}
```

### 配置客户端详情(Client Details)

`ClientDetailsServiceConfigurer`可以使用内存或者JDBC来配置客户端详情服务(ClientDeyailsService)，其中有几个重要属性：

- `clientId`: 用来标识客户端的ID，相当于username
- `secret`: 客户端的安全码，相当于password
- `scope`: 客户端的域，用来限制客户端的访问范围
- `authorizedGrantTypes`: 此客户端可以使用的授权类型
- `authorities`: 此客户端的权限

### 管理令牌(Managing Token)

`AuthorizationServerTokenServices`接口定义了一些操作实现对令牌的一些必要的管理。其中`DefaultTokenServices`这个实现类提供了一些方法操作Token，而Token数据的持久化是委托给`TokenStore`接口实现的，`TokenStore`有很多实现类用于不同的方式持久化Token数据：

- `InMemoryTokenStore`: 默认采用，将Token数据储存在内存中
- `JdbcTokenStore`: 封装一些JDBC的操作，将数据保存在关系型数据库中(需要`spring-jdbc`支持)
- `JwtTokenStore`: 将令牌相关的数据进行编码传输(需要`spring-security-jwt`支持)

### JWT令牌(JWT Tokens)

使用JWT令牌，需要在守群服务中使用`JwtTokenStore`，资源服务器也需要一个解码Token令牌的类`JwtAccessTokenConverter`，`JwtTokenStore`依赖这个类进行编码和解码，因此你的授权服务和资源服务都需要使用这个转换类。Token令牌默认是有签名的，并且资源服务需要验证这个签名，因此需要使用一个对称的Key值来参与签名的计算，这个Key存在于授权服务以及资源服务之中。或者可以使用非对称加密加密算法对Token进行签名，Public Key公布在`/oauth/token_key`这个URL中，默认的访问规则是`denyAll()`，即在默认情况下他是关闭的，可以配置`permitAll()`来开启（`tokenKeyAccess`）。

### 端点URL

Spring Security OAuth2内部提供了一些接口作为认证和授权：

- `/oauth/token`: 令牌端点
- `/oauth/authorize`: 授权端点
- `/oauth/confirm_access`: 用户确认授权提交端点
- `/oauth/error`: 授权服务错误信息端点
- `/oauth/check_token`: 用于资源服务访问的令牌解析端点
- `/oauth/token_key`: 提供公有的秘钥的端点，比如使用JWT令牌时用到

## 资源服务器 Resource Server

通过`@EnableResourceServer`注解配置

资源服务中储存着受保护或为受保护的资源，简单来说资源就是Controller中的一写接口，因为我们通常是通过接口来获取数据的。整个应用中所有的请求都受Spring Security的过滤器链保护，过滤器链决定了那些接口是直接可以访问的哪些是必须经过授权才能访问的。

通常我们使用`@EnableResourceServer`注解标记，再继承`ResourceServerConfigurerAdapter`类，重写其中的方法实现对资源服务器的配置，比如设置资源服务的ID，自定义一些权限保护规则等：

```java
public class ResourceServerConfigurerAdapter implements ResourceServerConfigurer {
    public ResourceServerConfigurerAdapter() {
    }

    public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
    }

    public void configure(HttpSecurity http) throws Exception {
        ((AuthorizedUrl)http.authorizeRequests().anyRequest()).authenticated();
    }
}
```



<br/>

# 交流

QQGroup：671017003   

WeChatGroup:  关注公众号查看

# 联系

- [Blog@TyCoding's blog](http://www.tycoding.cn)
- [GitHub@TyCoding](https://github.com/TyCoding)
- [ZhiHu@TyCoding](https://www.zhihu.com/people/tomo-83-82/activities)

