# 1.SpringSecurity简介

Spring 是非常流行和成功的 Java 应用开发框架，而Spring Security 正是其中的一员。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方案。
关于安全方面的两个核心功能是“认证”和“授权”，一般来说，Web 应用的安全性包括**用户认证（Authentication）和用户授权（Authorization）**两个部分，这两点也是 SpringSecurity 重要核心功能。

1. 用户认证：验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码，系统通过校验用户名和密码来完成认证过程。
2. 用户授权：验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。



# 2.Shiro和Security的对比

## 2.1 Shiro的特点

- 轻量级。主张的理念是把复杂的事情变简单。针对对性能有更高要求的互联网应用有更好表现。
- 通用性。不局限于 Web 环境，可以脱离 Web 环境使用。
- 依赖性低。不需要任何框架和容器，可以独立运行
- 配置和使用比较简单
- Shiro的三个核心组件：Subject、SecurityManager 和 Realms。

## 2.2 Security的特点

- 全面的权限控制。
- 专门为 Web 开发而设计。
- Spring Security 依赖Spring容器
- 新版本对整个框架进行了分层抽取，分成了核心模块和 Web 模块。单独引入核心模块就可以脱离 Web 环境。

## 2.3 二者的相同点

- 认证功能
- 授权功能
- 加密功能
- 会话管理
- 缓存支持
- rememberMe功能
- 基于以上，一般来说，常见的安全管理技术栈的组合是这样的：
  - SSM + Shiro
  - Spring Boot/Spring Cloud + Spring Security



# 3.Security实现权限

若要对Web资源进行保护，最好的办法是使用Filter对方法调用进行保护，当然也可以使用AOP的方式。
Spring Security进行认证和鉴权的时候，就是利用的一系列的Filter来进行拦截的。

![image-20230221111126841](D:\Java学习\java笔记\spring Security\assets\image-20230221111126841.png)

一个请求想要访问到API就会从左到右经过蓝线框里的过滤器，其中绿色部分是负责认证的过滤器，蓝色部分是负责异常处理，橙色部分则是负责授权。进过一系列拦截最终访问到我们的API。

这里面我们只需要重点关注两个过滤器即可：UsernamePasswordAuthenticationFilter负责登录认证，FilterSecurityInterceptor负责权限授权。

## 3.1springSecurity基本原理

springSecurity本质上是一个过滤器链， 有十几个过滤器

**源码分析：**

```markdown
FilterSecurityInterceptor是一个方法级的权限过滤器，基本位于过滤链的最底部
ExceptionTranslationFilter是一个异常过滤器，用来处理在认证授权过程中抛出的异常
UsernamePasswordAuthenticationFilter对/login的POST请求进行拦截，校验表单中的用户名和密码
```



# 4.用户认证流程

![image-20230221111303611](D:\Java学习\java笔记\spring Security\assets\image-20230221111303611.png)

当前登录用户在Spring Security中的体现就是 Authentication，它存储了认证信息，代表当前登录用户。使用时通过 SecurityContext 来获取Authentication，SecurityContext就是我们的上下文对象！这个上下文对象则是交由 SecurityContextHolder 进行管理。你可以这样使用：

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
```

SecurityContextHolder原理非常简单，就是使用ThreadLocal来保证一个线程中传递同一个对象。

Spring Security中三个核心组件：

- Authentication：存储了认证信息，代表当前登录用户
- SeucirtyContext：上下文对象，用来获取Authentication
- SecurityContextHolder：上下文管理对象，用来在程序任何地方获取SecurityContext

Authentication中的信息包含：

- Principal：用户信息，没有认证时一般是用户名，认证后一般是用户对象
- Credentials：用户凭证，一般是密码
- Authorities：用户权限

**Spring Security用户认证关键代码**

```java
// 生成一个包含账号密码的认证信息
Authentication authenticationToken = new UsernamePasswordAuthenticationToken(username, passwrod);
// AuthenticationManager校验这个认证信息，返回一个已认证的Authentication
Authentication authentication = authenticationManager.authenticate(authenticationToken);
// 将返回的Authentication存到上下文中
SecurityContextHolder.getContext().setAuthentication(authentication);
```



# 5.springSecurity初探

创建一个springboot项目

导入springSecurity依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

简写一个controller

```java
@RestController
public class UserController {

    @RequestMapping("/hello")
    public String hello() {
        return "hello";
    }
}
```

启动项目并访问，会跳转一个springSecurity登录页面

<img src="D:\Java学习\java笔记\spring Security\assets\image-20230221112054326.png" alt="image-20230221112054326" style="zoom:50%;" />

默认用户名为user

默认名密码在springboot控制台中随机生成

<img src="D:\Java学习\java笔记\spring Security\assets\image-20230221112234828.png" alt="image-20230221112234828" style="zoom:50%;" />

## 5.1两个重要接口

1. **UserDetailsService接口：**

   当什么也没有配置的时候，账号和密码是由 Spring Security 定义生成的。而在实际项目中账号和密码都是从数据库中查询出来的。 所以我们要通过自定义逻辑控制认证逻辑。

   查询数据库用户名和密码的过程：

   - 创建类继承UsernamePasswordAuthenticationFilter，重写三个方法
   - 创建类实现UserDetailService，编写查询数据过程，返回User对象女这个User 对象是安全框架提供对象

   ​		

2. **PasswordEncoder接口：**

数据加密接口，用于返回User对象里面的密码加密



# 6.Web的认证和授权

## 6.1认证

设置登录的用户名和密码，有三种方式

- 第一种方式:通过配置文件

  ```properties
  spring:
    security:
      user:
        name: husky
        password: 123456
  ```

- 第二种方式:通过配置类(该继承类WebSecurityConfigurerAdapter已过时)

  ```java
  @Configuration
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
      /*
      * 使用BCryptPasswordEncoder加密必须手动创建PasswordEncoder的Bean实例对象
      * 不然会报这个异常：There is no PasswordEncoder mapped for the id "null"
      * */
      @Bean
      PasswordEncoder passwordEncoder(){
          return new BCryptPasswordEncoder();
      }
  
      // 通过配置类进行用户名和密码的设置
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
          // 密码加密
          BCryptPasswordEncoder bCryptPasswordEncoder = new BCryptPasswordEncoder();
          String encode = bCryptPasswordEncoder.encode("123");
          // 设置，roles()用户类型
          auth.inMemoryAuthentication().withUser("admin").password(encode).roles("admin");
      }
  }
  ```

  

- 第三种方式:自定义编写实现类

  - 第一步创建配置类，设置使用哪个userDetailsService实现类

    ```java
    @Configuration
    @Slf4j
    public class SecurityConfig2 extends WebSecurityConfigurerAdapter{
    
        @Autowired
        private UserDetailsService userDetailsService;
    
        @Bean
        PasswordEncoder passwordEncoder(){
            return new BCryptPasswordEncoder();
        }
    
        @Override
        protected void configure(AuthenticationManagerBuilder auth) throws Exception {
            auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
        }
    }
    ```

  - 第二步 编写实现类，返回User对象，User对象有用户名密码和操作权限

    ```java
    @Service
    public class UserDetailServiceImpl implements UserDetailsService {
        @Override
        public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
            List<GrantedAuthority> auths =
                    AuthorityUtils.commaSeparatedStringToAuthorityList("role");
            return new User("admin",new BCryptPasswordEncoder().encode("123"),auths);
        }
    }
    ```

### 数据库查询认证

```java
    
// 数据库查询认证
@Service
public class UserDetailServiceImpl implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;

    /**
     * 查询数据库进行用户认证
     * @param username
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        LambdaQueryWrapper<com.husky.entity.User> queryWrapper = new LambdaQueryWrapper<>();
        queryWrapper.eq(com.husky.entity.User::getUsername, username);
        com.husky.entity.User user = userMapper.selectOne(queryWrapper);
        if (Objects.isNull(user)) {
            throw new UsernameNotFoundException("用户不存在");
        }
        List<GrantedAuthority> auths =
                AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
    }
}
```





## 6.2自定义设置登录页面

不需要通过springSecurity的登陆页面也可以访问

1. 配置类实现相关的配置

   ```java
   @Override
   protected void configure(HttpSecurity http) throws Exception {
       http
           .formLogin()   // 自定义自己编写的的登录页面
           .loginPage("/hello.html")   // 登录页面设置
           .loginProcessingUrl("/user/login")  // 登录访问路径（controller），业务逻辑不需要写，只需提供路径，Security会帮我们做好
           .defaultSuccessUrl("/index").permitAll()    // 登录成功之后，跳转的页面，permitAll()放行
           .and()
           .authorizeRequests()    // 标明哪些路径可以访问不需要认证。哪些不可以访问需要认证
           .antMatchers("/","/user/login","/hello").permitAll()    // 这些路径可以访问不需要认证
           .anyRequest().authenticated()   // 所有请求都可以访问
           .and()
           .csrf().disable()   // 关闭csrf防护
   }
   ```
   



## 6.3基于角色或权限进行访问控制

**如果当前的主体具有指定的权限，则返回 true,否则返回 false**

### 第一个方法：hasAuthority方法（只能设置一个权限）

- 步骤

  1.在配置类设置当前访问地址有哪些权限

  ```java
  .antMatchers("/index").hasAuthority("admins")   // 当前用户。只有具有admins权限才可以访问这个路径
  ```

  2.在UserDetailService的实现类中把返回的User对象设置权限

  ```java
  List<GrantedAuthority> auths =
                  AuthorityUtils.commaSeparatedStringToAuthorityList("admins");
          return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
  ```

### 第二个方法：hasAnyAuthority方法（可添加多个权限）

- ```java
  .antMatchers("/index").hasAnyAuthority("admins","manager")
  ```

- ```java
  List<GrantedAuthority> auths =
                  AuthorityUtils.commaSeparatedStringToAuthorityList("admins");
          return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
  ```





**如果用户具备给定角色就允许访问,否则出现 403。如果当前主体具有指定的角色，则返回 true。**

### 第三个方法：hasRole方法

查看源码得知使用该方法设置权限时，在UserDetailService的实现类的权限设置中要加上"ROLE_"前缀

- ```java
  .antMatchers("/index").hasRole("sale")
  ```

- ```java
  List<GrantedAuthority> auths =
                  AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_sale");
          return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
  ```

### 	第四个方法：hasAnyRole方法

- ```java
  .antMatchers("/index").hasAnyRole("sale","admins")
  ```

- ```java
  // 用户权限设置可写一个或多个权限
  List<GrantedAuthority> auths =
                  AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_sale, ROLE_admins");
          return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
  ```



## 6.4自定义403没有权限访问的页面

在配置类中直接设置

```java
// 配置没有权限访问跳转到自定义的页面
http
    .exceptionHandling().accessDeniedPage("/error.html");
```



## 6.5注解的使用

使用注解前需要开启注解功能

在启动类上或在配置类上添加**@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)**开启注解功能

### **@Secured**

使用该注解要在@EnableGlobalMethodSecurity(**securedEnabled = true**)加入加粗字体的参数

设置可以访问的权限，判断某个角色具有该权限，才可以使用方法。另外需要注意的是这里匹配的字符串需要添加前缀“ROLE_ “	

1.在controller的方法上面使用该注解

```java
@RequestMapping("/update")
@Secured({"ROLE_sale", "ROLE_admins"})
public String update() {
    return "hello update";
}
```

2.在UserDetailService的实现类中设置用户的权限，也得加上前缀“ROLE_ “，才能使用

```java
 List<GrantedAuthority> auths =
                AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_admins");
        return new User(user.getUsername(),new BCryptPasswordEncoder().encode(user.getPassword()),auths);
```



### @PreAuthorize

使用该注解要在@EnableGlobalMethodSecurity(**prePostEnabled = true**)加入加粗字体的参数

@PreAuthorize : 注解是进入方法前的权限验证，@ PreAuthorize 可以将登录用户的 roles/permissions 参数传到方法中。

<img src="D:\Java学习\java笔记\spring Security\assets\image-20230222151559549.png" alt="image-20230222151559549" style="zoom:50%;" />

该注解和配置类权限控制类似，**不过该注解在controller的方法中使用**

```java
@RequestMapping("/test")
@PreAuthorize("hasAnyAuthority('test')")
public String test() {
    return "hello test";
}
```



### @PostAuthorize

@PostAuthorize 注解使用并不多，在方法执行后再进行权限验证，适合验证带有返回值的权限.

使用该注解要在@EnableGlobalMethodSecurity(**prePostEnabled = true**)加入加粗字体的参数

```java
@RequestMapping("/add")
@PostAuthorize("hasAnyAuthority('admins')")
public String add() {
    System.out.println("add");
    return "hello add";
}
```



### @PostFilter

方法返回的数据进行过滤

```java
@GetMapping("/getAll")
@PreAuthorize("hasAnyAuthority('admins')")
@PostFilter("filterObject.username == 'lisi'")
public List<User> getAll() {
    List<User> userList = new ArrayList<>();
    userList.add(new User(11,"zhangsan","123456"));
    userList.add(new User(12,"lisi","123456"));
    return userList;
}

// 返回userList的时候先对返回的数据进行了过滤，username == 'lisi'的，才返回
```



### @PreFilter

传入方法的数据进行过滤

```java
@GetMapping("/testPre")
@PreAuthorize("hasAnyAuthority('admins')")
@PreFilter("filterObject.id%2==0")
public List<User> testPre(@RequestBody List<User> users) {
    users.forEach(t->{
        System.out.println(t.getId()+"\n"+t.getUsername());
    });
    return users;
}

// 对传入的users数据进行过滤，若id能被整除，就返回该数据
```



## 6.6用户的注销

在配置类中添加退出映射地址

```java
// 退出并返回指定url, "/logout"的业务逻辑同样不需要写，只需和前端的退出url保持一致就行
        http
                .logout().logoutUrl("/logout").logoutSuccessUrl("/hello").permitAll();
```



## 6.7SpringSecurity实现自动登录

### 实现原理

1. 当用户登录认证成功之后，SpringSecurity会将加密后的cookie（加密串）存进一份至浏览器，在数据库中也存入一份加密串和用户信息字符串；
2. 当下次再访问时，SpringSecurity会获取cookie信息，拿着cookie信息到数据库进行比对，如果查询到对应信息，认证成功，就自动登录

![image-20230222162644733](D:\Java学习\java笔记\spring Security\assets\11-web权限方案-记住用户流程.png)



### 具体实现

第一步，创建数据库表，可在源码**JdbcTokenRepositoryImpl**中找到建表语句

```sql
CREATE TABLE `persistent_logins`(

	`username` VARCHAR(64) NOT NULL,

	`series` VARCHAR(64) NOT NULL,

	`token` VARCHAR(64) NOT NULL,

	`last_used` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

	PRIMARY KEY(`series`)

) ENGINE=INNODB DEFAULT CHARSET=utf8;
```

第二步，在配置类中，注入数据源，配置操作数据库对象

```java
import javax.sql.DataSource;

// 注入数据源
@Autowired
private DataSource dataSource;

// 配置对象, 创建JdbcTokenRepositoryImpl的接口的Bean，返回它的实现类
@Bean
public PersistentTokenRepository persistentTokenRepository(){
    JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
    jdbcTokenRepository.setDataSource(dataSource);
    return jdbcTokenRepository;
}
```

第三步，配置类中配置自动登录

```java
// 配置自动登录
http
    .rememberMe().tokenRepository(persistentTokenRepository())
    .tokenValiditySeconds(60)   // 设置有效时长，以秒为单位
    .userDetailsService(userDetailsService);
```

第四步，前端的自动登录选项的**name="remember-me"**,必须

```html
<input type="checkbox" name="remember-me"/>自动登录<br/>
```



## 6.7 CSRF

​		跨站请求伪造(英语: Cross-site requestforgery) ，也被称为 one-clickattack 或者 session riding，通常缩写为 CSRF 或者 XSRF，是一种挟制用户在当前已登录的 Web 应用程序上执行非本意的操作的攻击方法。跟跨网站脚本(XSS)相比，XSS利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。

​		跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并运行一些操作(如发邮件，发消息，其至财产操作如转账和购买商品)。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去运行。这利用了 web 中用户身份验证的一个漏洞: 简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。

​		从 Spring Security 4.0 开始，默认情况下会启用 CSRF 保护，以防止 CSRF 攻击应用程序，Spring Security CSRF 会针对 PATCH，POST，PUT 和 DELETE 方法进行防护。



## 6.8配置类总结

```java
package com.husky.config;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.rememberme.JdbcTokenRepositoryImpl;
import org.springframework.security.web.authentication.rememberme.PersistentTokenRepository;
import javax.sql.DataSource;

/**
 * Created by IntelliJ IDEA.
 * User: 周圣杰
 * Date: 2023/2/21
 * Time: 15:29
 */
@Configuration
@EnableWebSecurity
@Slf4j
public class SecurityConfig2 extends WebSecurityConfigurerAdapter{

    @Autowired
    private UserDetailsService userDetailsService;

    // 注入数据源
    @Autowired
    private DataSource dataSource;

    // 配置对象
    @Bean
    public PersistentTokenRepository persistentTokenRepository(){
        JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
        jdbcTokenRepository.setDataSource(dataSource);
        return jdbcTokenRepository;
    }

    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .formLogin()   // 自定义自己编写的的登录页面
                .loginPage("/login.html")   // 登录页面设置
                .loginProcessingUrl("/user/login")  // 登录访问路径（controller），业务逻辑不需要写，只需提供路径，Security会帮我们做好
                .defaultSuccessUrl("/success.html").permitAll()    // 登录成功之后，跳转的页面，permitAll()放行
                .and()
                .authorizeRequests()    // 标明哪些路径可以访问不需要认证。哪些不可以访问需要认证
                .antMatchers("/","/user/login","/hello").permitAll()    // 这些路径可以访问不需要认证
                // .antMatchers("/index").hasAuthority("admins")   // 当前用户。只有具有admins权限才可以访问这个路径
                // .antMatchers("/index").hasAnyAuthority("admins","manager")
                // .antMatchers("/index").hasRole("sale")
                .antMatchers("/index").hasAnyRole("sale","admins")
                .anyRequest().authenticated()   // 所有请求都可以访问
                .and()
                .csrf().disable();  // 关闭csrf防护
        // 配置没有权限访问跳转到自定义的页面
        http
                .exceptionHandling().accessDeniedPage("/error.html");

        // 退出并返回指定url
        http
                .logout().logoutUrl("/logout").logoutSuccessUrl("/hello").permitAll();

        // 配置自动登录
        http
                .rememberMe().tokenRepository(persistentTokenRepository())
                .tokenValiditySeconds(60*5)   // 设置有效时长，以秒为单位
                .userDetailsService(userDetailsService);
    }
}

```



# 7.微服务的认证和授权

## 认证授权过程分析

（1）如果是基于 Session，那么 Spring-security 会对 cookie 里的 sessionid 进行解析，找 到服务器存储的 session 信息，然后判断当前用户是否符合请求的要求。 

（2）如果是 token，则是解析出 token，然后将当前请求加入到 Spring-security 管理的权限 信息中去

![image-20230222203342727](D:\Java学习\java笔记\spring Security\assets\image-20230222203342727.png)

1. 如果系统的模块众多，每个模块都需要进行授权与认证，所以我们选择基于 token 的形式 进行授权与认证，
2. 用户根据用户名密码认证成功，然后获取当前用户角色的一系列权限值，
3. 并以用户名为 key，权限列表为 value 的形式存入 redis 缓存中，
4. 根据用户名相关信息生成 token 返回，浏览器将 token 记录到 cookie 中，
5. 每次调用 api 接口都默认将 token 携带到header 请求头中，
6. Spring-security 解析 header 头获取 token 信息，解析 token 获取当前 用户名，
7. 根据用户名就可以从 redis 中获取权限列表，这样 Spring-security 就能够判断当前请求是否有权限访问



## 项目用例

### 创建工程

导入相关的依赖，[参考文件](D:\Java学习\SpringSecurityStudy\资料\案例pom文件)

<img src="D:\Java学习\java笔记\spring Security\assets\image-20230223160445349.png" alt="image-20230223160445349" style="zoom:50%;" />

### service_base工程

#### 添加工具类

[参考文件](D:\Java学习\SpringSecurityStudy\资料\案例工具类)

<img src="D:\Java学习\java笔记\spring Security\assets\image-20230223160841282.png" alt="image-20230223160841282" style="zoom:50%;" />

### spring_security工程



#### 密码处理工具类

```java
@Component
public class DefaultPasswordEncoder implements PasswordEncoder {

    public DefaultPasswordEncoder(){
        this(-1);
    }
    public DefaultPasswordEncoder(int len) {

    }

    // 使用MD5加密
    @Override
    public String encode(CharSequence rawPassword) {
        return MD5.encrypt(rawPassword.toString());
    }

    // 进行密码比对
    @Override
    public boolean matches(CharSequence rawPassword, String encodedPassword) {
        return encodedPassword.equals(MD5.encrypt(rawPassword.toString()));
    }
}
```



#### token工具类

```java
@Component
public class TokenManager {
    // 设置过期时间
    private static final long EXPIRE = 1000 * 60 * 60 * 24;

    // Jwt签名 自定义
    private static final String SIGNATURE = "Husky-jie";

    /**
     * 使用jwt根据用户名生成token
     * @param username
     * @return
     */
    public static String getToken(String username){
        if (username == null){
            return null;
        }
        /*
         * builder()：构建Jwt
         * setSubject(SUBJECT)：设置主题，自定义*/
        return Jwts.builder().setSubject(username)
                /* payload 通常⽤来存放⽤⼾信息
                 * */
                .setIssuedAt(new Date())
                // 设置过期时间
                .setExpiration(new Date(System.currentTimeMillis()+EXPIRE))
                //signature 签名值
                .signWith(SignatureAlgorithm.HS256, SIGNATURE)
                /*
                 * compact()用来拼接以下三个值：
                 * header可以不写，有默认值
                 * payload
                 * signature*/
                .compact();
    }

    // 校验token
    public static Claims checkJwt(String token){
        try {
            /*Jwts.parser()解密
             * setSigningKey(签名) 通过签名解密
             * parseClaimsJwt(token) 需要解密的token
             * getBody()获取Claims对象，该对象封装了用户的信息*/
            return Jwts.parser().setSigningKey(SIGNATURE).parseClaimsJws(token).getBody();
        } catch (Exception e) {
            // 若篡改token会导致校验失败，走到异常分支，这里返回null
            return null;
        }
    }
}
```



#### 退出处理器

```java
public class TokenLogoutHandler implements LogoutHandler {

    private RedisTemplate<Object, Object> redisTemplate;

    // 构造器注入
    public TokenLogoutHandler(RedisTemplate<Object, Object> redisTemplate){
        this.redisTemplate = redisTemplate;
    }

    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        // 获取token
        String token = request.getHeader("token");
        // 可能token不在头信息中
        if (token == null) {
            token = request.getParameter("token");
        }
        // 移除token
        TokenManager.removeToken(token);
        // 获取用户名
        String username = TokenManager.getUserInfoFromToken(token);
        if (username != null) {
            // 从redis中移除用户名
            redisTemplate.delete(username);
        }
    }
}
```



####  未授权统一处理类

```java
public class UnAuthEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        ResponseUtil.out(response, R.error());
    }
}
```



#### 登录认证过滤器

```java
public class TokenLoginFilter extends UsernamePasswordAuthenticationFilter {

    private TokenManager tokenManager;
    private RedisTemplate<Object, Object> redisTemplate;
    private AuthenticationManager authenticationManager;

    public TokenLoginFilter(TokenManager tokenManager, RedisTemplate<Object, Object> redisTemplate, AuthenticationManager authenticationManager) {
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
        this.authenticationManager = authenticationManager;
        this.setPostOnly(false);
        // 设置登录的请求路径，交给spring Security处理， 该设置等同于在配置类中设置 .loginProcessingUrl("/admin/acl/login")
        this.setRequiresAuthenticationRequestMatcher(new AntPathRequestMatcher("/admin/acl/login", "POST"));
    }

    // 获取表单提交的用户名和密码
    @Override
    public Authentication attemptAuthentication(HttpServletRequest request,
                                                HttpServletResponse response) throws AuthenticationException {
        // 获取表单提交的数据
        try {
            // readValue方法将数据转换为对象
            // getInputStream()获取表单提交的所有内容
            User user = new ObjectMapper().readValue(request.getInputStream(), User.class);
            return authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(user.getUsername(),
                    user.getPassword(),null));
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException();
        }
    }

    // 认证成功调用的方法
    @Override
    protected void successfulAuthentication(HttpServletRequest request,
                                            HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        // 得到认证成功之后的用户信息
        SecurityUser user = (SecurityUser) authResult.getPrincipal();
        // 认证成功根据用户名生成token
        String token = tokenManager.getToken(user.getCurrentUserInfo().getUsername());
        // 把用户名和用户权限列表存入redis中
        redisTemplate.opsForValue().set(user.getCurrentUserInfo().getUsername(), user.getPermissionValueList());
        // 返回token
        ResponseUtil.out(response, R.ok().data("token", token));
    }

    // 认证失败调用的方法
    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request,
                                              HttpServletResponse response, AuthenticationException failed) throws IOException, ServletException {
        ResponseUtil.out(response, R.error());
    }
}
```



#### 授权过滤器

```java
public class TokenAuthFilter extends BasicAuthenticationFilter {

    private TokenManager tokenManager;
    private RedisTemplate<Object, Object> redisTemplate;

    public TokenAuthFilter(AuthenticationManager authenticationManager,TokenManager tokenManager, RedisTemplate<Object, Object> redisTemplate) {
        super(authenticationManager);
        this.tokenManager = tokenManager;
        this.redisTemplate = redisTemplate;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 获取当前用户的权限信息
        UsernamePasswordAuthenticationToken authentication = getAuthentication(request);

        // 判断如果有权限信息，放到权限上下文中
        if (authentication != null) {
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        chain.doFilter(request, response);
    }

    // 获取当前用户的权限信息
    public UsernamePasswordAuthenticationToken getAuthentication (HttpServletRequest request){
        String token = request.getHeader("token");
        if (token == null) {
            token = request.getParameter("token");
        }
        // 获取用户名
        String username = tokenManager.getUserInfoFromToken(token);
        // 从redis中获取当前用户的权限列表
        List<String> permissionValueList = (List<String>) redisTemplate.opsForValue().get(username);
        Collection<GrantedAuthority> authorities = new ArrayList<>();
        for (String permission :
                permissionValueList) {
            SimpleGrantedAuthority authority = new SimpleGrantedAuthority(permission);
            authorities.add(authority);
        }
        return new UsernamePasswordAuthenticationToken(username, token, authorities);
    }
}
```



#### 核心配置类

```java
@Configuration
public class TokenWebSecurityConfig extends WebSecurityConfigurerAdapter {

    private TokenManager tokenManager;
    private DefaultPasswordEncoder defaultPasswordEncoder;
    private UserDetailsService userDetailsService;
    private RedisTemplate<Object, Object> redisTemplate;

    public TokenWebSecurityConfig(TokenManager tokenManager, DefaultPasswordEncoder defaultPasswordEncoder,
                                  UserDetailsService userDetailsService, RedisTemplate<Object, Object> redisTemplate) {
        this.tokenManager = tokenManager;
        this.defaultPasswordEncoder = defaultPasswordEncoder;
        this.userDetailsService = userDetailsService;
        this.redisTemplate = redisTemplate;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.exceptionHandling()
                .authenticationEntryPoint(new UnAuthEntryPoint())   // 没有权限访问的统一处理
                .and().csrf().disable()
                .authorizeRequests()
                .anyRequest().authenticated()   // 所有请求都可以访问
           		// 登录的url已在登录认证过滤器中设置
                .and().logout().logoutUrl("/admin/acl/index/logout")    // 退出的url
                // 添加退出处理器
                .addLogoutHandler(new TokenLogoutHandler(tokenManager, redisTemplate)).and()
                // 添加过滤器
                .addFilter(new TokenLoginFilter(tokenManager, redisTemplate, authenticationManager()))
                .addFilter(new TokenAuthFilter(authenticationManager(), tokenManager, redisTemplate))
                .httpBasic();
    }

    // 调用userDetailService和密码处理
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(defaultPasswordEncoder);
    }

    // 不进行认证的路径，可以直接访问
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/api/**");
    }
}
```



#### UserDetail自定义实现类

```java
@Data
public class SecurityUser implements UserDetails {
    private static final long serialVersionUID = -8875001389123722444L;

    //当前登录用户
    private transient User currentUserInfo;
    //当前权限
    private List<String> permissionValueList;
    public SecurityUser() {
    }
    public SecurityUser(User user) {
        if (user != null) {
            this.currentUserInfo = user;
        }
    }
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        Collection<GrantedAuthority> authorities = new ArrayList<>();
        for(String permissionValue : permissionValueList) {
            if(StringUtils.isEmpty(permissionValue)) continue;
            SimpleGrantedAuthority authority = new
                    SimpleGrantedAuthority(permissionValue);
            authorities.add(authority);
        }
        return authorities;
    }
    @Override
    public String getPassword() {
        return currentUserInfo.getPassword();
    }
    @Override
    public String getUsername() {
        return currentUserInfo.getUsername();
    }
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    @Override
    public boolean isEnabled() {
        return true;
    }

}
```



### service_acl工程

#### UserDetailServiceImpl编写

```java
@Service("userDetailsService")
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserService userService;

    @Autowired
    private PermissionService permissionService;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 根据用户名查询用户，该用户是数据库中的对象
        User user = userService.selectByUsername(username);

        if (user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        
        // 按需求设置的用户对象，在spring_security工程下
        com.husky.entity.User curUser = new com.husky.entity.User();
        BeanUtils.copyProperties(user, curUser);

        // 根据用户查询用户权限列表
        List<String> permissionValueList = permissionService.selectPermissionValueByUserId(user.getId());
        SecurityUser securityUser = new SecurityUser();
        securityUser.setCurrentUserInfo(curUser);
        securityUser.setPermissionValueList(permissionValueList);
        return securityUser;
    }
}
```



#### 配置文件

```properties
# 服务端口
server.port=8009
# 服务名
spring.application.name=service-acl

# mysql数据库连接
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=zsj1583893320

#返回json的全局时间格式
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8

spring.redis.host=192.168.108.128
spring.redis.port=6379
spring.redis.database= 0
spring.redis.timeout=1800000

spring.redis.lettuce.pool.max-active=20
spring.redis.lettuce.pool.max-wait=-1
#最大阻塞等待时间(负数表示没限制)
spring.redis.lettuce.pool.max-idle=5
spring.redis.lettuce.pool.min-idle=0
#最小空闲

#配置mapper xml文件的路径
mybatis-plus.mapper-locations=classpath:com/husky/aclservice/mapper/xml/*.xml

# nacos服务地址
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848

#mybatis日志
mybatis-plus.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```



### api_gateway工程

#### 启动类

并添加上需要在nacos中注册的注解

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}
```



#### 配置类

```java
package com.husky.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.reactive.CorsWebFilter;
import org.springframework.web.cors.reactive.UrlBasedCorsConfigurationSource;
import org.springframework.web.util.pattern.PathPatternParser;

/**
 * Created by IntelliJ IDEA.
 * User: 周圣杰
 * Date: 2023/2/24
 * Time: 11:14
 */
@Configuration
public class CorsConfig {

    // 解决跨域
    @Bean
    public CorsWebFilter corsWebFilter() {
        // 所有请求均可访问
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedMethod("*");
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**", corsConfiguration);

        return new CorsWebFilter(source);
    }
}

```



#### 配置文件

```properties
# 服务端口
server.port=8222
# 服务名
spring.application.name=service-gateway
# nacos服务地址
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
# 使用服务发现路由
spring.cloud.gateway.discovery.locator.enabled=true

# 配置路由规则
spring.cloud.gateway.routes[0].id=service-acl
# 设置路由uri  lb://服务名称
spring.cloud.gateway.routes[0].uri=http://localhost:8009
# 具体路径规则
spring.cloud.gateway.routes[0].predicates= Path=/*/acl/**
```



# 8.Spring Security工作流程

Spring Security 采取过滤链实现认证与授权，只有当前过滤器通过，才能进入下一个过滤器:

<img src="D:\Java学习\java笔记\spring Security\assets\image-20230225152053770.png" alt="image-20230225152053770" style="zoom:50%;" />

绿色部分是认证过滤器，需要我们自己配置，可以配置多个认证过滤器。认证过滤器可以使用Spring Security 提供的认证过滤器，也可以自定义过滤器(例如:短信验证)。认证过滤器要在 configure(HttpSecurity http)方法中配置，没有配置不生效。下面会重点介绍以下三个过滤器:
UsernamePasswordAuthenticationFilter 过滤器:该过滤器会拦截前端提交的 POST 方式的登录表单请求，并进行身份认证。

ExceptionTranslationFilter 过滤器:该过滤器不需要我们配置，对于前端提交的请求会直接放行，捕获后续抛出的异常并进行处理(例如: 权限访问限制)。

FilterSecurityInterceptor 过滤器:该过滤器是过滤器链的最后一个过滤器，根据资源权限配置来判断当前请求是否有权限访问对应的资源。如果访问受限会抛出相关异常，并由ExceptionTranslationFilter 过滤器进行捕获和处理



## 8.1认证流程

**认证流程是在UsernamePasswordAuthenticationFilter过滤器中处理的，具体流程如下：**

<img src="D:\Java学习\java笔记\spring Security\assets\23-认证流程.jpg" alt="image-20230225152253396" style="zoom:50%;" />



### UsernamePasswordAuthenticationFilter

当前端提交的是一个 POST 方式的登录表单请求，就会被该过滤器拦截，并进行身份认证。该过滤器的 doFilter( )方法实现在其抽象父类

### AbstractAuthenticationProcessingFilter

在UsernamePasswordAuthenticationFilter抽象父类AbstractAuthenticationProcessingFilter，查看相关源码：(查看过程)

#### **第一步：过滤的方法，判断提交方式是否post提交**

```java
// 过滤器doFilter()方法
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
    throws IOException, ServletException {
    if (!requiresAuthentication(request, response)) {
        // 首先会判断该请求是否为POST方式的登录表单提交请求，如果不是则直接放行，进入下一个过滤器
        chain.doFilter(request, response);
        return;
    }
```

#### **第二步：调用子类的方法进行身份认证，认证成功之后，把认证信息封装到对象里面**

```java
try {
   // Authenticatton 是用来存储用户认证信息的类，后续会进行详细介绍
   // 调用子类 UsernamePasswordAuthenticationFilter 重写的方法进行身份认证
   // 返回的 authenticationResult 对象封装认证后的用户信息
   Authentication authenticationResult = attemptAuthentication(request, response);
   if (authenticationResult == null) {
      // return immediately as subclass has indicated that it hasn't completed
      return;
   }
```

#### **第三步：session策略处理**

```java
// Session 策略处理（如果配置了用户 session 最大并发数，就是在此处进行判断并处理）
this.sessionStrategy.onAuthentication(authenticationResult, request, response);
```

#### **第四步：其一，认证失败抛出异常，执行认证失败的方法**

```java
catch (InternalAuthenticationServiceException failed) {
   this.logger.error("An internal error occurred while trying to authenticate the user.", failed);
    // 认证失败，调用认证失败的处理器
   unsuccessfulAuthentication(request, response, failed);
}
```

#### **第四步：其二，认证成功，调用认证成功的方法**

```java
// 认证成功处理
if (this.continueChainBeforeSuccessfulAuthentication) {
    // 默认的continueChainBeforeSuccessfulAuthentication为false，所以认证成功之后不会进入下一个过滤器
      chain.doFilter(request, response);
   }
	// 调用认证成功的处理器
   successfulAuthentication(request, response, chain, authenticationResult);
}
```



**上述的 第二步 过程调用了 UsernamePasswordAuthenticationFilter 的 attemptAuthentication() 方法，源码如下：**



### UsernamePasswordAuthenticationFilter 的 attemptAuthentication() 方法

```java
public class UsernamePasswordAuthenticationFilter extends AbstractAuthenticationProcessingFilter {
    public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
    public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";
    // 默认登录表单提交路径为 /login，POST 方式请求
    private static final AntPathRequestMatcher DEFAULT_ANT_PATH_REQUEST_MATCHER = new AntPathRequestMatcher("/login", "POST");
    // 默认表单用户名参数为username
    private String usernameParameter = "username";
    // 默认密码参数为password
    private String passwordParameter = "password";
    // 默认请求方式只能为POST
    private boolean postOnly = true;

    public UsernamePasswordAuthenticationFilter() {
        // 默认登录表单提交路径为 /login，POST 方式请求
        super(DEFAULT_ANT_PATH_REQUEST_MATCHER);
    }
```



#### **attemptAuthentication() 方法**

```java
// 上述的 doFilter() 方法调用此 attemptAuthentication() 方法进行身份认证
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
    if (this.postOnly && !request.getMethod().equals("POST")) {
        // 默认情况下，如果请求方式不是 POST，会抛出异常
        throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
    } else {
        // 获取请求携带的 username 和 password
        String username = this.obtainUsername(request);
        username = username != null ? username.trim() : "";
        String password = this.obtainPassword(request);
        password = password != null ? password : "";
        // 使用前端传入的 username、password参数，构造 Authentication 对象，标记该对象为未认证状态
        UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username, password);
        // 将请求中的一些属性信息设置到 Authentication 对象中，如: remoteAddress，sessionId
        this.setDetails(request, authRequest);
        // 调用 ProviderManager 类的 authenticate()方法进行身份认证
        return this.getAuthenticationManager().authenticate(authRequest);
    }
}
```



**上述的attemptAuthentication() 方法过程创建的 UsernamePasswordAuthenticationToken 是 Authentication 接口的实现类，该类有两个构造器，一个用于封装前端请求传入的未认 证的用户信息，一个用于封装认证成功后的用户信息：**

```java
// 用于封装前端请求传入的未认证的用户信息，前面的 authRequest 对象就是调用该构造器进行构造的
public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
   // 用户权限为null
   super(null);
   // 前端传入的用户名
   this.principal = principal;
   // 前端传入的密码
   this.credentials = credentials;
   // 标记为未认证
   setAuthenticated(false);
}

// 用于封装认证成功后的用户信息
public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
			Collection<? extends GrantedAuthority> authorities) {
	// 用户权限集合	
    super(authorities);
    // 封装认证用户的UserDetails对象，不再是用户名
    this.principal = principal;
    // 前端传入的密码
    this.credentials = credentials;
    // 标记为认证成功
    super.setAuthenticated(true); 
}
```



上述的attemptAuthentication() 方法返回的Authentication 接口的实现类用于存储用户认证信息，查看该接口具体定义

```java
public interface Authentication extends Principal, Serializable {
	// 用户权限集合
   Collection<? extends GrantedAuthority> getAuthorities();
   	// 用户密码
   Object getCredentials();
	// 请求携带的一些属性信息(例如: remoteAddress，sessionId)
   Object getDetails();
	// 未认证时为前端请求传入的用户名，认证成功后为封装认证用户信息的 userDetails 对象
   Object getPrincipal();
   	// 是否被认证(true: 认证成功，false: 未认证)
   boolean isAuthenticated();
  	// 设置是否被认证(true: 认证成功，false: 未认证)
   void setAuthenticated(boolean isAuthenticated) throws IllegalArgumentException;

}
```



### ProviderManager源码中的authenticate方法

上述过程中，UsernamePasswordAuthenticationFilter 过滤器的 attemptAuthentication() 方法的return过程将未认证的 Authentication 对象传入 ProviderManager 类的 **authenticate() 方法**进行身份认证

ProviderManager 是 AuthenticationManager 接口的实现类，该接口是认证相关的核心接 口，也是认证的入口。在实际开发中，我们可能有多种不同的认证方式，例如：用户名+ 密码、邮箱+密码、手机号+验证码等，而这些认证方式的入口始终只有一个，那就是 AuthenticationManager。在该接口的常用实现类 ProviderManager 内部会维护一个 List列表，存放多种认证方式，实际上这是委托者模式 （Delegate）的应用。每种认证方式对应着一个 AuthenticationProvider， AuthenticationManager 根据认证方式的不同（根据传入的 Authentication 类型判断）委托 对应的 AuthenticationProvider 进行用户认证。

```java
// 传入未认证的Authentication对象
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
   	// (1) 获取传入的 Authentication 类型，即 UsernamePasswordAuthenticationToken.class
    Class<? extends Authentication> toTest = authentication.getClass();
    AuthenticationException lastException = null;
    AuthenticationException parentException = null;
    Authentication result = null;
    Authentication parentResult = null;
    int currentPosition = 0;
    int size = this.providers.size();
    /**
    getProviders()方法：
    (2)获取认证方式列表 ListsAuthenticationProvider>
    public List<AuthenticationProvider> getProviders() {
		return this.providers;
	}
    */
    for (AuthenticationProvider provider : getProviders()) {
        // (3)判断当前AuthenticationProvider是否适用UsernamePasswordAuthenticationToken.class类型的AuthenticationProvider
        if (!provider.supports(toTest)) {
            continue;
        }
        if (logger.isTraceEnabled()) {
            logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",
                                           provider.getClass().getSimpleName(), ++currentPosition, size));
        }
        // 成功找到适配当前认证方式的 AuthenticationProvider，此处为 DaoAuthenticationProvider
        try {
            // (4)调用 DaoAuthenticationProvider 的 authenticate() 方法进行认证;
            // 如果认证成功，会返回标记已认证的 Authen	tication 对象
            result = provider.authenticate(authentication);
            if (result != null) {
                // (5)认证成功后，将传入的Authentication对象中的details信息拷贝到已认证的Authentication对象中
                copyDetails(authentication, result);
                break;
            }
        }
        catch (AccountStatusException | InternalAuthenticationServiceException ex) {
            prepareException(ex, authentication);   
            throw ex;
        }
        catch (AuthenticationException ex) {
            lastException = ex;
        }
    }
    if (result == null && this.parent != null) {  
        try {
            // (5) 认证失败，用父类型 AuthenticationManager 进行验证
            parentResult = this.parent.authenticate(authentication);
            result = parentResult;
        }
        catch (ProviderNotFoundException ex) {  
        }
        catch (AuthenticationException ex) {
            parentException = ex;
            lastException = ex;
        }
    }
    if (result != null) {
        // (6)认证成功之后，去除 result 的敏感信息，要求相关类实现 CredentialsContainer 接口
        if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) { 
            // 去除过程就是调用 CredentialsContainer 接口的 eraseCredentials() 方法
            ((CredentialsContainer) result).eraseCredentials();
        }
		// (7)发布认证成功的事件
        if (parentResult == null) {
            this.eventPublisher.publishAuthenticationSuccess(result);
        }

        return result;
    }
	//(8) 认证失败之后，抛出失败的异常信息
    if (lastException == null) {
        lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",
                                                                               new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));
    }

    if (parentException == null) {
        prepareException(lastException, authentication);
    }
    throw lastException;
}
```

**上述认证成功之后的（6）过程，调用 CredentialsContainer 接口定义的 eraseCredentials() 方法去除敏感信息。查看 UsernamePasswordAuthenticationToken 实现的 eraseCredentials() 方法，该方 法实现在其父类中：**

```java
public void eraseCredentials() {
   	// credentials(前端传入的密码)会设置为 null
    eraseSecret(getCredentials());
    // principal 在已认证的 Authentication 中是 userDetails 实现类;
    // 如果该实现类想要去除敏感信息，需要实现 CredentialsContatner 接口的 eraseCredentials() 方法
    // 由于我们自定义的 user 类没有实现该接口，所以不进行任何操作。
   	eraseSecret(getPrincipal());
   	eraseSecret(this.details);
}
```



### 认证成功和认证失败

上述过程就是认证流程的最核心部分，接下来重新回到 UsernamePasswordAuthenticationFilter 过滤器的 doFilter() 方法，查看认证成 功/失败的处理：

![image-20230225213921389](D:\Java学习\java笔记\spring Security\assets\image-20230225213921389.png)

**查看successfulAuthentication()方法和unsuccessfulAuthentication()方法的源码**

```java
// 认证成功后的处理
protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
      Authentication authResult) throws IOException, ServletException {
   // (1)将认证成功的用户信息对象Authentication 封装进 SecurityContext 对象中，并存入SecurityContextHolder
   // SecurityContextHolder 是对 ThreadLocal 的一个封装
   SecurityContext context = SecurityContextHolder.createEmptyContext();
   context.setAuthentication(authResult);
   SecurityContextHolder.setContext(context);
   this.securityContextRepository.saveContext(context, request, response);
   if (this.logger.isDebugEnabled()) {
      this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
   }
    // (2) rememberMe 的处理
   this.rememberMeServices.loginSuccess(request, response, authResult);
   if (this.eventPublisher != null) {
       // (3) 发布认证成功的事件
      this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));
   }
    // (4) 调用认证成功处理器
   this.successHandler.onAuthenticationSuccess(request, response, authResult);
}

// 认证失败后的处理
protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,
			AuthenticationException failed) throws IOException, ServletException {
	// (1)清除该线程在 SecurityContextHolder 中对应的 SecurityContext 对象	
    SecurityContextHolder.clearContext();
    this.logger.trace("Failed to process authentication request", failed);
    this.logger.trace("Cleared SecurityContextHolder");
    this.logger.trace("Handling authentication failure");
    // (2) rememberMe 的处理
    this.rememberMeServices.loginFail(request, response);
    // (3) 调用认证失败处理器
    this.failureHandler.onAuthenticationFailure(request, response, failed);
	}

```



## 8.2认证流程图解

![](D:\Java学习\java笔记\spring Security\assets\24-认证流程中各核心类和接口的关系图.jpg)



## 8.3权限访问流程

上一个部分通过源码的方式介绍了认证流程，下面介绍权限访问流程，主要是对 ExceptionTranslationFilter 过滤器和 FilterSecurityInterceptor 过滤器进行介绍。



###  ExceptionTranslationFilter 过滤器

该过滤器是用于处理异常的，不需要我们配置，对于前端提交的请求会直接放行，捕获后续抛出的异常并进行处理（例如：权限访问限制）。具体源码如下：

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
      throws IOException, ServletException {
   try {
       // (1) 对于前端提交的请求会直接放行，不进行拦截
      chain.doFilter(request, response);
   }
   catch (IOException ex) {
      throw ex;
   }
   catch (Exception ex) {
      // (2) 捕获后续出现的异常进行处理
      Throwable[] causeChain = this.throwableAnalyzer.determineCauseChain(ex);
       // 访问需要认证的资源，当前请求未认	证所抛出的异常
      RuntimeException securityException = (AuthenticationException) this.throwableAnalyzer
            .getFirstThrowableOfType(AuthenticationException.class, causeChain);
      if (securityException == null) {
		// 访问权限受限的资源所抛出的异常
         securityException = (AccessDeniedException) this.throwableAnalyzer
               .getFirstThrowableOfType(AccessDeniedException.class, causeChain);
      }
      if (securityException == null) {
         rethrow(ex);
      }
      if (response.isCommitted()) {
         throw new ServletException("Unable to handle the Spring Security Exception "
               + "because the response is already committed.", ex);
      }
      handleSpringSecurityException(request, response, chain, securityException);
   }
}
```



### FilterSecurityInterceptor 过滤器

FilterSecurityInterceptor 是过滤器链的最后一个过滤器，该过滤器是过滤器链 的最后一个过滤器，根据资源权限配置来判断当前请求是否有权限访问对应的资源。如果 访问受限会抛出相关异常，最终所抛出的异常会由前一个过滤器 ExceptionTranslationFilter 进行捕获和处理。具体源码如下：

```java
// 过滤器的 doFilter() 方法
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
      throws IOException, ServletException {
    // 调用invoke()方法
   invoke(new FilterInvocation(request, response, chain));
}

public void invoke(FilterInvocation filterInvocation) throws IOException, ServletException {
		if (isApplied(filterInvocation) && this.observeOncePerRequest) {
			filterInvocation.getChain().doFilter(filterInvocation.getRequest(), filterInvocation.getResponse());
			return;
		}
		if (filterInvocation.getRequest() != null && this.observeOncePerRequest) {
			filterInvocation.getRequest().setAttribute(FILTER_APPLIED, Boolean.TRUE);
		}
    // (1)根据资源权限配置来判断当前请求是否有权限访问对应的资源。如果不能访问，则抛出相应的异常
		InterceptorStatusToken token = super.beforeInvocation(filterInvocation);
		try {
            //(2) 访问相关资源，通过 springMvC 的核心组件 dispatcherServlet 进行访问
			filterInvocation.getChain().doFilter(filterInvocation.getRequest(), filterInvocation.getResponse());
		}
		finally {
			super.finallyInvocation(token);
		}
		super.afterInvocation(token, null);
	}
```

**需要注意，Spring Security 的过滤器链是配置在 SpringMVC 的核心组件 DispatcherServlet 运行之前。也就是说，请求通过 Spring Security 的所有过滤器， 不意味着能够正常访问资源，该请求还需要通过 SpringMVC 的拦截器链。**



# 9.SpringSecurity 请求间共享认证信息

一般认证成功后的用户信息是通过 Session 在多个请求之间共享，那么 Spring  Security 中是如何实现将已认证的用户信息对象 Authentication 与 Session 绑定的进行 具体分析。

![image-20230225222154785](D:\Java学习\java笔记\spring Security\assets\image-20230225222154785.png)

在前面认证成功的处理方法 successfulAuthentication() 时，有以下代码：

![image-20230225222528685](D:\Java学习\java笔记\spring Security\assets\image-20230225222528685.png)

SecurityContext 接 口 及 其 实 现 类 SecurityContextImpl ， 该 类 其 实 就 是 对 Authentication 的封装；

SecurityContextHolder 类 ， 该 类 其 实 是 对 ThreadLocal 的 封 装 ， 存 储 SecurityContext 对象



## SecurityContextHolder源码

```java
private static void initialize() {
   initializeStrategy();
   initializeCount++;
}

private static void initializeStrategy() {
   if (MODE_PRE_INITIALIZED.equals(strategyName)) {
      Assert.state(strategy != null, "When using " + MODE_PRE_INITIALIZED
            + ", setContextHolderStrategy must be called with the fully constructed strategy");
      return;
   }
   if (!StringUtils.hasText(strategyName)) {
      // 默认使用 MODE_THREADLOCAL 模式
      strategyName = MODE_THREADLOCAL;
   }
   if (strategyName.equals(MODE_THREADLOCAL)) {
       // 默认使用 ThreadLocalSecurityContextHolderStrategy 创建 strategy， 其内部使用 ThreadLocal 对 SecurityContext 进行存储
      strategy = new ThreadLocalSecurityContextHolderStrategy();
      return;
   }
    
    public static SecurityContext getContext() {
        // 需要注意，如果当前线程对应的 ThreadLocal<SecurityContext> 没有任何对象存储，
        // strategy.getContext() 会创建并返回一个空的 SecurityContext 对象，
        // 并且该空的 SecurityContext 对象会存入 ThreadLocal<SecurityContext>
        return strategy.getContext();
    }
    
    public static void setContext(SecurityContext context) {
        //设置当前线程对应的 ThreadLocal<SecurityContext> 的存储
        strategy.setContext(context);
	}

    public static void clearContext() {
        //清除当前线程对应的 ThreadLocal<SecurityContext> 的存储
		strategy.clearContext();
	}

```



## 上述的ThreadLocalSecurityContextHolderStrategy源码

```java
// 使用 ThreadLocal 对 SecurityContext 进行存储
private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();

public SecurityContext getContext() {
    	// 需要注意，如果当前线程对应的 ThreadLocal<SecurityContext> 没有任何对象存储，
        // getContext() 会创建并返回一个空的 SecurityContext 对象，
        // 并且该空的 SecurityContext 对象会存入 ThreadLocal<SecurityContext>
		SecurityContext ctx = contextHolder.get();
		if (ctx == null) {
			ctx = createEmptyContext();
			contextHolder.set(ctx);
		}
		return ctx;
	}

@Override
	public void setContext(SecurityContext context) {
        //设置当前线程对应的 ThreadLocal<SecurityContext> 的存储
		Assert.notNull(context, "Only non-null SecurityContext instances are permitted");
		contextHolder.set(context);
	}

@Override
public SecurityContext createEmptyContext() {
    // 创建一个空的 SecurityContext 对象
    return new SecurityContextImpl();
}

@Override
public void clearContext() {
     //清除当前线程对应的 ThreadLocal<SecurityContext> 的存储
    contextHolder.remove();
}

```



## SecurityContextPersistenceFilter 过滤器

前面提到过，在 UsernamePasswordAuthenticationFilter 过滤器认证成功之 后，会在认证成功的处理方法中将已认证的用户信息对象 Authentication 封装进 SecurityContext，并存入 SecurityContextHolder。

 之后，响应会通过 SecurityContextPersistenceFilter 过滤器，该过滤器的位置 在所有过滤器的最前面，请求到来先进它，响应返回最后一个通过它，所以在该过滤器中 处理已认证的用户信息对象 Authentication 与 Session 绑定。 

认证成功的响应通过 SecurityContextPersistenceFilter 过滤器时，会从 SecurityContextHolder 中取出封装了已认证用户信息对象 Authentication 的 SecurityContext，放进 Session 中。当请求再次到来时，请求首先经过该过滤器，该过滤 器会判断当前请求的 Session 是否存有 SecurityContext 对象，如果有则将该对象取出再次 放入 SecurityContextHolder 中，之后该请求所在的线程获得认证用户信息，后续的资源访 问不需要进行身份认证；当响应再次返回时，该过滤器同样从 SecurityContextHolder 取出 SecurityContext 对象，放入 Session 中。具体源码如下

```java
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
      throws IOException, ServletException {
   if (request.getAttribute(FILTER_APPLIED) != null) {
      chain.doFilter(request, response);
      return;
   }
   request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
   if (this.forceEagerSessionCreation) {
      HttpSession session = request.getSession();
      if (this.logger.isDebugEnabled() && session.isNew()) {
         this.logger.debug(LogMessage.format("Created session %s eagerly", session.getId()));
      }
   }
   HttpRequestResponseHolder holder = new HttpRequestResponseHolder(request, response);
    // (1)请求到来时，检查当前 session 中是否存有 SecurityContext 对象
    // 如果有，从 sesston 中取出该对象，如果没有，创建一个空的 SecurityContext 对象
   SecurityContext contextBeforeChainExecution = this.repo.loadContext(holder);
   try {
       // (2)将上述获得 securityContext 对象放入 SecurityContextHolder 中
      SecurityContextHolder.setContext(contextBeforeChainExecution);
      if (contextBeforeChainExecution.getAuthentication() == null) {
         logger.debug("Set SecurityContextHolder to empty SecurityContext");
      }
      else {
         if (this.logger.isDebugEnabled()) {
            this.logger
                  .debug(LogMessage.format("Set SecurityContextHolder to %s", contextBeforeChainExecution));
         }
      }
       // (3)进入下一个过滤器
      chain.doFilter(holder.getRequest(), holder.getResponse());
   }
   finally {
       // (4)响应返回时，从 SecurityContextHolder 中取出 SecurityContext
      SecurityContext contextAfterChainExecution = SecurityContextHolder.getContext();
       // (5)移除 SecurityContextHolder 中的 SecurityContext 对象
      SecurityContextHolder.clearContext();
       // (6)将取出的 SecurityContext 对象放进 Session
      this.repo.saveContext(contextAfterChainExecution, holder.getRequest(), holder.getResponse());
      request.removeAttribute(FILTER_APPLIED);
      this.logger.debug("Cleared SecurityContextHolder to complete request");
   }
}
```
