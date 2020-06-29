---
title: （十六）SpringSecurity实现图形验证码校验
date: 2019-02-26 19:22:56
tags: spring-boot
categories: spring-boot
top: 71
---

# SpringSecurity图形验证码校验

:tada: :tada: :tada: **这里有丰富的 Spring 框架学习案例**

**仓库地址：[spring-learn](https://github.com/TyCoding/spring-learn)**
**欢迎star、fork，给作者一些鼓励**


接着上一篇文章：[SpringBoot整合SpringSecurity](https://tycoding.cn/2019/04/11/boot/spring-boot-security/)，这次我们学习Spring Security如何实现图形验证码校验。

<!--more-->

> 依赖引入

实现验证码登录，后台除了负责生成验证码，还应该维护验证码的生命周期，而为了给验证码赋予一个生命周期，通常，我们会将验证码数据保存在Session中。比如页面长时间未刷新，此时的验证码就应该失效并重新刷新验证码的值。那么我们可以通过`HttpSession`实现，像之前学习Shiro一样。而在Spring Secuirty中，提供了`spring-social`类库，他提供了一些方法实现系统的第三方登录，比如QQ、微信等，为了配合后面学习`Security-OAuth`，这里我们导入`spring-social`的依赖：

```xml
<dependency>
    <groupId>org.springframework.social</groupId>
    <artifactId>spring-social-security</artifactId>
    <version>1.1.6.RELEASE</version>
</dependency>
```

`SessionStrategy`的使用：

```java
public interface SessionStrategy {
    void setAttribute(RequestAttributes var1, String var2, Object var3);

    Object getAttribute(RequestAttributes var1, String var2);

    void removeAttribute(RequestAttributes var1, String var2);
}
```

## 工具类封装

>  1.创建`ImageCode.java`，封装验证码对象属性

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ImageCode {

    /**
     * 验证码BufferedImage对象
     */
    private BufferedImage image;

    /**
     * 验证码String值
     */
    private String code;

    /**
     * 验证码过期时间
     */
    private LocalDateTime expireTime;

    public ImageCode(BufferedImage image, String code, int expireIn) {
        this.image = image;
        this.code = code;
        this.expireTime = LocalDateTime.now().plusSeconds(expireIn);
    }

    public boolean isExpire() {
        return LocalDateTime.now().isAfter(expireTime);
    }
}
```

> 2.创建`ImageCodeGenerator.java`

```java
public class ImageCodeGenerator {

    public static ImageCode createImageCode() {
        int width = ImageConstants.CODE_WIDTH;
        int height = ImageConstants.CODE_HEIGHT;
        int length = ImageConstants.CODE_LENGTH;
        int fontSize = ImageConstants.CODE_FONT_SIZE;
        int expireIn = ImageConstants.CODE_EXPIREIN;
        BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_RGB);
        Graphics g = image.getGraphics();
        Random random = new Random();

        g.setColor(getRandColor(200, 250));
        g.fillRect(0, 0, width, height);
        g.setFont(new Font("Times New Roman", Font.ITALIC, fontSize));
        g.setColor(getRandColor(160, 200));
        for (int i = 0; i < 155; i++) {
            int x = random.nextInt(width);
            int y = random.nextInt(height);
            int xl = random.nextInt(12);
            int yl = random.nextInt(12);
            g.drawLine(x, y, x + xl, y + yl);
        }

        StringBuilder sRand = new StringBuilder();
        for (int i = 0; i < length; i++) {
            String rand = String.valueOf(random.nextInt(10));
            sRand.append(rand);
            g.setColor(new Color(20 + random.nextInt(110), 20 + random.nextInt(110), 20 + random.nextInt(110)));
            g.drawString(rand, 13 * i + 6, 16);
        }
        g.dispose();
        return new ImageCode(image, sRand.toString(), expireIn);
    }

    private static Color getRandColor(int fc, int bc) {
        Random random = new Random();
        if (fc > 255) {
            fc = 255;
        }
        if (bc > 255) {
            bc = 255;
        }
        int r = fc + random.nextInt(bc - fc);
        int g = fc + random.nextInt(bc - fc);
        int b = fc + random.nextInt(bc - fc);
        return new Color(r, g, b);
    }
}
```

上面都是Java生成图片的一些方法，不是重点不再说。其中上面都用到了一个类：`ImageConstants`，这个是我简单封装的储存图片验证码相关常量的接口：

```java
public interface ImageConstants {

    /**
     * 验证码宽度
     */
    int CODE_WIDTH = 67;

    /**
     * 验证码高度
     */
    int CODE_HEIGHT = 23;

    /**
     * 验证码的过期时间
     */
    int CODE_EXPIREIN = 60;

    /**
     * 验证码的位数
     */
    int CODE_LENGTH = 4;

    /**
     * 验证码字体大小
     */
    int CODE_FONT_SIZE = 20;

    /**
     * 验证码格式
     */
    String CODE_TYPE = "JPEG";

    /**
     * Session中存验证码的key值
     */
    String SESSION_KEY_CODE = "SESSION_KEY_IMAGE_CODE";

    /**
     * 登录表单验证码name值
     */
    String LOGIN_FORM_CODE = "code";
}
```

比起写一个class存放一些final变量，感觉用接口更加方便。

## 获取验证码的接口

上面创建了工具类，此时可以提供一个专门用于获取验证码的Controller:

```java
@RestController
public class ValidateCodeController {

    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();

    /**
     * 获取验证码
     *
     * @param request
     * @param response
     */
    @GetMapping("/code/image")
    public void getCode(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ImageCode imageCode = ImageCodeGenerator.createImageCode();
        sessionStrategy.setAttribute(new ServletWebRequest(request), ImageConstants.SESSION_KEY_CODE, imageCode);
        ImageIO.write(imageCode.getImage(), ImageConstants.CODE_TYPE, response.getOutputStream());
    }
}
```

1. 首先调用`ImageGenerator`工具类的`createImageCode`方法生成一个验证码对象`ImageCode`。
2. 调用`SessionStrategy`把当前的`ImageCode`对象储存在Session中。
3. 调用`ImageIO.write()`方法将`BufferedImage`对象转换为一个图片格式的文件并写入到response响应流中。

## 测试

上面通过一个接口可以向前端返回一个验证码图片，但需要配置此接口在未登录情况时不拦截：

```java
.antMatchers("/login", "/code/image").permitAll()
```

同时，在前端登录表单上加上验证码选项：

```html
<form action="/auth/login" method="post">
    <input name="username" type="text" placeholder="Username"><br/>
    <input name="password" type="password" placeholder="Password"><br/>
    <input name="code" type="text" placeholder="Code">
    <img src="/code/image" style="margin-bottom: -12px;">
    <br/>
    <input type="submit" value="Login">
</form>
```

启动项目：

![](http://cdn.tycoding.cn/20200629092250.png)

至此，我们仅仅完成了图形验证码的绘制即展示，即使在Controller层将验证码对象储存在Session中，但是，并没有任何地方使用了这个Session。

## 验证码过滤器

![](http://cdn.tycoding.cn/20200629092254.png)

在之前学习的Spring Security过滤器链中，我们说`SecurityContextPersistenceFilter`是直接与请求交互的最前端的过滤器，`UsernamePasswordAuthenticationFilter`用于处理这个请求，而因为加了验证码校验，应该在`UsernamePasswordAuthenticationFilter`过滤器前面再判断如果验证码校验通过再校验用户名和密码，否者就直接抛出异常。

所以，我们需要创建一个验证码过滤器用于专门校验输入的验证码值；并且，这个验证码过滤器应该放在`UsernamePasswordAuthenticationFilter`过滤器前面执行。

在`cn/tycoding/filter/`下创建`ValidateCodeFilter`并继承`OncePerRequestFilter`(表示这个过滤器只执行一次，避免了每次请求都会执行这个过滤器)：

```java
public class ValidateCodeFilter extends OncePerRequestFilter {

    @Getter
    @Setter
    private AuthenticationFailureHandler authenticationFailureHandler;
    @Getter
    @Setter
    private SessionStrategy sessionStrategy = new HttpSessionSessionStrategy();

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        if (StringUtils.equals("/auth/login", request.getRequestURI()) && StringUtils.equalsIgnoreCase(request.getMethod(), "POST")) {
            try {
                validate(new ServletWebRequest(request));
            } catch (ValidateCodeException e) {
                authenticationFailureHandler.onAuthenticationFailure(request, response, e);
                return;
            }
        }
        filterChain.doFilter(request, response);
    }

    private void validate(ServletWebRequest request) throws ServletRequestBindingException {
        ImageCode codeInSession = (ImageCode) sessionStrategy.getAttribute(request, ImageConstants.SESSION_KEY_CODE);
        String codeInRequest = ServletRequestUtils.getStringParameter(request.getRequest(), ImageConstants.LOGIN_FORM_CODE);
        if (StringUtils.isBlank(codeInRequest)) {
            throw new ValidateCodeException("验证码的值不能为空");
        }
        if (codeInSession == null) {
            throw new ValidateCodeException("验证码不存在");
        }
        if (codeInSession.isExpire()) {
            sessionStrategy.removeAttribute(request, ImageConstants.SESSION_KEY_CODE);
            throw new ValidateCodeException("验证码已过期");
        }
        if (!StringUtils.equals(codeInSession.getCode(), codeInRequest)) {
            throw new ValidateCodeException("验证码不匹配");
        }
        sessionStrategy.removeAttribute(request, ImageConstants.SESSION_KEY_CODE);
    }
}
```

1. 过滤器首先判断请求的URL是不是登录的URL，其次还要符合是POST请求方法，然后才视为登录的验证码校验
2. 调用`SessionStrategy`获取在验证码接口`/code/image`中设置进Session中的验证码对象`ImageCode`。
3. 调用`ServletRequestUtils`获取到表单提交时指定`name`文本框的值，也就是表单中输入的验证码值。因为整个登录请求都是由Spring Security内部的过滤器实现的，所以登录请求携带的参数可以直接通过Spring的方法获取到。
4. 判断表单中输入的值和Session中储存的值是否存在、是否相符，判断验证码是否过期(根据本地当前时间和验证码过期时间比较)；如果条件不符合就直接抛出异常。

> 创建`ValidateCodeException`

```java
public class ValidateCodeException extends AuthenticationException {
    
    public ValidateCodeException(String msg) {
        super(msg);
    }
}
```

可以看到这个自定义异常处理器还是继承的`AuthenticationException`异常处理器。所以这个自定义异常并不是必须的，但为了更加规范的表示这是验证码校验时出现的错误就写上。

5. 如果验证码校验失败就清除Session中存在的验证码值。
6. 如果在以上整个过程任意地方出现错误就直接`return;`不再往下执行过滤器链。

### 配置

将当前的`ValidateCodeFilter`过滤器配置到`UsernamePasswordFilter`过滤器前面执行：

```java
        http
                .addFilterBefore(validateCodeFilter, UsernamePasswordAuthenticationFilter.class)
                .formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/auth/login")
                .successHandler(authenticationSuccessHandler)
                .failureHandler(authenticationFailureHandler)
```



## 测试

启动项目，在页面输入用户名密码、验证码登录，在`ValidateCodeFilter`过滤器处打断点：

![](http://cdn.tycoding.cn/20200629092301.png)

# SpringSecurity记住我

记住我，顾名思义，即系统能在一段时间内记住当前登录的用户信息，避免用户频繁的输入用户名密码登录系统。在Shiro学习中，记住我功能需要配置`RememberCookie`，将其注入到`RememberMeManager`，最终由Shiro内部对这个rememberMe数据管理。

而在Spring Security中，Spring Security内部提供了对rememberMe数据的JDBC工具封装，也就是说如果需要记住我，那么Spring Security会将用户信息都持久化到数据库中。再次请求时会根据请求中携带的Cookie信息和数据库中储存的信息比对来判断当前用户是否已登录。

![](http://cdn.tycoding.cn/20200629092307.png)

并且，我们再回顾之前的Spring Security过滤器链：

![](http://cdn.tycoding.cn/20200629092310.png)

这个记住我相关的过滤器配置应该放在`UsernamePasswordAuthenticationFilter`过滤器的后面。当用户登录后，再次发送请求，此时并不是一个认证请求，Spring Security就不再执行`UsernamePasswordAuthenticationFilter`，而是直接调用`RememberMeAuthenticationFilter`过滤器，查询数据库中储存的Token，和浏览器携带的Cookie信息比对。

## 创建一个Repository

创建一个`PersistentTokenRepository`对象用于配置JDBC相关信息，实现Token数据保存到数据库中：

```java
@Autowired
private DataSource dataSource;
@Bean
public PersistentTokenRepository persistentTokenRepository() {
    JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
    tokenRepository.setDataSource(dataSource);
    tokenRepository.setCreateTableOnStartup(true);
    return tokenRepository;
}
```

`setCreateTableOnStartup`如果数据库中不存在`persistent_logins`这张表，就设为true，下次执行会在数据库中创建该表，如果存在，就将该配置去掉。

添加数据库相关的配置：

```yaml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/springboot_security?useUnicode=true&characterEncoding=UTF-8
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

注入RememberMe的相关配置：

```java
@Autowired
private UserDetailsService userDetailsService;

http
    .and()
    .rememberMe()
    .tokenRepository(persistentTokenRepository())
    .tokenValiditySeconds(60) //60s
    .userDetailsService(userDetailsService)
```

在前端表单中添加记住我选项：

```html
<form action="/auth/login" method="post">
    <input name="username" type="text" placeholder="Username"><br/>
    <input name="password" type="password" placeholder="Password"><br/>
    <input name="code" type="text" placeholder="Code">
    <img src="/code/image" style="margin-bottom: -12px;">
    <br/>
    <input type="checkbox" name="remember-me" value="true">记住我
    <br/>
    <input type="submit" value="Login">
</form>
```



## 测试

> 1.启动项目

在登录界面勾选记住我并输入用户名密码进入系统。

可以看到数据库`springboot_security`中生成了一张表：

![](http://cdn.tycoding.cn/20200629092315.png)

> 2.关闭浏览器，重新打开并访问系统

在`RememberMeAuthenticationFilter`中打断点：

![](http://cdn.tycoding.cn/20200629092318.png)

可以看到此时过滤器得到了RememberMe记住的数据，那么这次访问就不需要登录直接可进入系统了。





<br/>

# 交流

QQGroup：671017003   

WeChatGroup:  关注公众号查看

# 联系

- [Blog@TyCoding's blog](http://www.tycoding.cn)
- [GitHub@TyCoding](https://github.com/TyCoding)
- [ZhiHu@TyCoding](https://www.zhihu.com/people/tomo-83-82/activities)

