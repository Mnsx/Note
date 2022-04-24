# SpringSecurity框架概述

Spring是非常流行和成功的Java应用开发框架，SpringSercurity正式Spring家族中的成员。SpringSecurity基于Spring框架，提供了一套Web应用安全性的完整解决方案

关于安全方面的两个重要区域是**认证**和**授权**（或者访问控制），一般来说，Web应用的安全性包括**用户认证和用户授权**两个部分，这两点也是SpringSecurity重要核心功能

1. 用户认证是指：验证某个用户是否为系统中的合法主题，也就是说用户能够访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。通俗点说就是系统认为用户是否能登录
2. 用户授权指的是某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限不同。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列权限。通俗点就是系统判断用户是否有权限去做某些事情

# 快速入门

## 入门案例

1. 创建SpringBoot工程

   略

2. 引入相关依赖

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
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
```

3. 编写Controller测试

```java
@RestController
@RequestMapping("/test")
public class TestController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello Security";
    }
}
```

## 基本原理

Spring Security本质是一个过滤器链

* FilterSecurityInterceptor：是一个方法级的权限过滤器，基本位于过滤链的最底层‘
* ExceptionTranslationFilter：是个异常过滤器，用来处理在认证授权过程中抛出的异常
* UsernamePasswordAuthenticationFilter：对/login的POST请求做拦截，校验表单中用户名和密码

## 接口讲解

* UserDetailService

  如果需要自定义逻辑时，只需要实现UserDetailService接口即可——查询数据库中登录名和密码的过程

  * 创建类继承UsernamePasswordAuthenticationFilter，重写三个方法
  * 创建类实现UserDetailService，编写查询数据过程，返回User对象
  * 这个User对象是安全框架提供

* passwordEncoder

  数据加密接口，用于返回User对象里面密码的加密

# Web权限方案

## 设置登录的用户名和密码

* 通过配置文件

```properties
spring.security.user.name=mnsx
spring.security.user.password=123123
```

* 通过配置类

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
        String password = passwordEncoder.encode("123456");
        auth.inMemoryAuthentication().withUser("xkq").password(password).roles("admin");
    }

    @Bean
    public PasswordEncoder getPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

* 自定义编写实现类
  1. 创建配置类，设置使用的UserDetailService实现类
  2. 编写实现类，返回User对象有用户名、密码、权限

```java
    // 自定义实现类
    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(getPasswordEncoder());
    }
    
    @Bean
    PasswordEncoder getPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
```

```java
@Service
public class MyUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("main");
        return new User("mnsx", new BCryptPasswordEncoder().encode("123123"), auths);
    }
}
```

## 连接数据库进行登录认证

0. 添加依赖

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
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
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
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>
    </dependencies>
```

1. 新添加对应数据库

```sql
create database spring_security;

use spring_security;

create table t_user(
id int auto_increment primary key,
username varchar(20),
password varchar(20));
```

2. 创建用户实体类

```java
@Data
public class User {
    private Integer id;
    private String username;
    private String password;
}
```

3. 创建实体类的持久层

```java
@Repository
public interface UserMapper extends BaseMapper<User> {
}
```

4. 修改逻辑层，使用UserMapper查询数据库得到数据

```java
@Service
public class MyUserDetailsService implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        QueryWrapper<top.mnsx.spring_security_study.entity.User> wrapper = new QueryWrapper<>();
        wrapper.eq("username", username);
        top.mnsx.spring_security_study.entity.User user = userMapper.selectOne(wrapper);
        if(user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("main");
        return new User(user.getUsername(), new BCryptPasswordEncoder().encode(user.getPassword()), auths);
    }
}
```

5. 添加数据库配置

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/spring_security?serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=123123

mybatis-plus.global-config.db-config.table-prefix=t_
```

## 自定义登录界面

1. 重写SecurityConfig方法，重写configure(HttpSecurity http)方法

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // 自定义实现类
    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(getPasswordEncoder());
    }

    @Bean
    PasswordEncoder getPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login.html") // 登录页面设置
                .loginProcessingUrl("/user/login") // 登录访问的路径
                .defaultSuccessUrl("/test/index").permitAll() // 登录成功之后，跳转路径
                .and().authorizeRequests()
                .antMatchers("/", "/test/hello", "/user/login").permitAll() // 设置白名单
                .anyRequest().authenticated()
                .and().csrf().disable(); // 关闭csrf防护
    }
}
```

2. 创建表单，**需要注意用户名和密码的name必须是指定的username和password**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>login</title>
</head>
<body>
    <form action="/user/login" method="post">
        用户名：<input type="text" name="username"/>
        <br/>
        密码：<input type="text" name="password"/>
        <br/>
        <input type="submit" value="login"/>
    </form>
</body>
</html>
```

## 基于角色或权限进行访问控制

* hasAuthority方法

  如果当前主体具有指定权限，则返回true，否则返回false

  1. 在配置类中设置当前

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    // 自定义实现类
    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(getPasswordEncoder());
    }

    @Bean
    PasswordEncoder getPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login.html") // 登录页面设置
                .loginProcessingUrl("/user/login") // 登录访问的路径
                .defaultSuccessUrl("/test/index").permitAll() // 登录成功之后，跳转路径
                .and().authorizeRequests()
                .antMatchers("/", "/test/hello", "/user/login").permitAll() // 设置白名单
                .antMatchers("/test/index").hasAuthority("admins")
                .anyRequest().authenticated()
                .and().csrf().disable(); // 关闭csrf防护
    }
}
```

* 2. 修改service方法，将其中对应的访问权限改为，可以通过的权限

```java
@Service
public class MyUserDetailsService implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        QueryWrapper<top.mnsx.spring_security_study.entity.User> wrapper = new QueryWrapper<>();
        wrapper.eq("username", username);
        top.mnsx.spring_security_study.entity.User user = userMapper.selectOne(wrapper);
        if(user == null) {
            throw new UsernameNotFoundException("用户不存在");
        }
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("admins");
        return new User(user.getUsername(), new BCryptPasswordEncoder().encode(user.getPassword()), auths);
    }
}
```

* hasAnyAuthority方法

  如果当前的主体有任何提供的角色（给定的作为一个逗号分隔的字符串列表）的话返回true

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login.html") // 登录页面设置
                .loginProcessingUrl("/user/login") // 登录访问的路径
                .defaultSuccessUrl("/test/index").permitAll() // 登录成功之后，跳转路径
                .and().authorizeRequests()
                .antMatchers("/", "/test/hello", "/user/login").permitAll() // 设置白名单
                .antMatchers("/test/index").hasAnyAuthority("admin", "manager")
                .anyRequest().authenticated()
                .and().csrf().disable(); // 关闭csrf防护
    }
```

* hasRole方法

  如果当前用户具有指定角色，则返回true

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login.html") // 登录页面设置
                .loginProcessingUrl("/user/login") // 登录访问的路径
                .defaultSuccessUrl("/test/index").permitAll() // 登录成功之后，跳转路径
                .and().authorizeRequests()
                .antMatchers("/", "/test/hello", "/user/login").permitAll() // 设置白名单
                .antMatchers("/test/index").hasRole("sale")
                .anyRequest().authenticated()
                .and().csrf().disable(); // 关闭csrf防护
    }
```

`List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("ROLE_sale");`**需要注意的是hasRole的底层会将输入的role转换程ROLE_xxx，所以在定义service时需要在角色前加上ROLE_**

* hasAnyRole方法

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
                .loginPage("/login.html") // 登录页面设置
                .loginProcessingUrl("/user/login") // 登录访问的路径
                .defaultSuccessUrl("/test/index").permitAll() // 登录成功之后，跳转路径
                .and().authorizeRequests()
                .antMatchers("/", "/test/hello", "/user/login").permitAll() // 设置白名单
                .antMatchers("/test/index").hasAnyRole("sale", "manager")
                .anyRequest().authenticated()
                .and().csrf().disable(); // 关闭csrf防护
    }
```

## 自定义403界面

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.exceptionHandling().accessDeniedPage("/unauth.html");
        http.formLogin()
                .loginPage("/login.html") // 登录页面设置
                .loginProcessingUrl("/user/login") // 登录访问的路径
                .defaultSuccessUrl("/test/index").permitAll() // 登录成功之后，跳转路径
                .and().authorizeRequests()
                .antMatchers("/", "/test/hello", "/user/login").permitAll() // 设置白名单
                .antMatchers("/test/index").hasAnyRole("sale", "manager")
                .anyRequest().authenticated()
                .and().csrf().disable(); // 关闭csrf防护
    }
```

## SpringSecurity注解使用

1. @Secured

   判断是否具有角色，另外需要注意的时这里配置的字符串需要添加前缀“ROLE_“

   **使用注解需要开启注解功能**

   `@EnableGlobalMethodSecurity(securedEnabled=true)`

   1. 在启动类或者配置类上添加注解，开启注解

      ```java
      @Configuration
      @EnableGlobalMethodSecurity(securedEnabled = true)
      public class SecurityConfig extends WebSecurityConfigurerAdapter {
      ```

   2. 在controller的方法上使用注解，设置角色

      ```java
          @GetMapping("/update")
          @Secured({"ROLE_sale", "ROLE_manager"})
          public String update() {
              return "hello update";
          }
      ```

   3. userDetailService设置用户角色

2. @PreAuthorize

   可以在执行方法之前进行权限验证

   **使用注解需要开启注解功能**

   `@EnableGlobalMethodSecurity(prePostEnabled=true)`

   1. 在启动类上开启注解功能

      ```java
      @Configuration
      @EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
      public class SecurityConfig extends WebSecurityConfigurerAdapter {
      ```

   2. 在Controller方法上添加注解

      ```java
          @GetMapping("/update")
          @PreAuthorize("hasAnyAuthority('admin')")
          public String update() {
              return "hello update";
          }
      ```

   3. userDetailService设置用户角色

3. @PostAuthorize

   可以在执行方法之后进行权限验证

   **使用注解需要开启注解功能**

   `@EnableGlobalMethodSecurity(prePostEnabled=true)`

   1. 在启动类上开启注解功能

      ```java
      @Configuration
      @EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
      public class SecurityConfig extends WebSecurityConfigurerAdapter {
      ```

   2. 在Controller方法上添加注解

      ```java
          @GetMapping("/update")
          @PostAuthorize("hasAnyAuthority('admins')")
          public String update() {
              return "hello update";
          }
      ```

   3. userDetailService设置用户角色

4. @PostFilter

   权限验证之后对数据进行过滤，留下用户名admin1的数据，表达式中的filterObject引用的是方法返回值List中一个元素

   ```java
       @GetMapping("getAll")
       @PostAuthorize("hasAnyAuthority('admin')")
       @PostFilter("filterObject.username == 'admin1'")
       public List<User> getAllUser(){
           List<User> list = new ArrayList<>();
           list.add(new User(1, "admin", "123"));
           list.add(new User(2, "admin1", "321"));
           return list;
       }
   ```

5. @PreFilter

   进入控制器之前对数据进行过滤

   ```java
   	@GetMapping("getTestPreFilter")
   	@PreAuthorize("hasRole('ROLE_管理员')")
   	@preFIlter(value = "filterObject.id % 2 == 0")
   	public List<User> getTestPreFilter(@RequestBody List<User> list) {
           list.forEach(t -> {
               System.out.println("t.getId() + "\t" + t.getUsername());
           });
           return list;
       }
   ```

## 用户注销

在配置类中添加退出配置

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/hello").permitAll();
        http.exceptionHandling().accessDeniedPage("/unauth.html");
        http.formLogin()
                .loginPage("/login.html") // 登录页面设置
                .loginProcessingUrl("/user/login") // 登录访问的路径
                .defaultSuccessUrl("/test/index").permitAll() // 登录成功之后，跳转路径
                .and().authorizeRequests()
                .antMatchers("/", "/test/hello", "/user/login").permitAll() // 设置白名单
                .antMatchers("/test/index").hasAnyRole("sale", "manager", "admin")
                .anyRequest().authenticated()
                .and().csrf().disable(); // 关闭csrf防护
    }
```

## 基于数据库的自动登录

![image-20220423104556675](D:\Picture\Note\SpringSecurity\token流程)

1. 创建数据库表（可以通过SpringSecurity自动生成）

   ```sql
   create table persistent_logins (
   username varchar(64) not null,
   series varchar(64) not null,
   token varchar(64) not null,
   last_used TIMESTAMP not null default CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP,
   primary key (series))
   ```

2. 配置类，注入数据源，配置操作数据库的对象

   ```java
   @Configuration
   public class TokenConfig {
       @Autowired
       private DataSource dataSource;
   
       @Bean
       public PersistentTokenRepository repository() {
           JdbcTokenRepositoryImpl jdbcTokenRepository = new JdbcTokenRepositoryImpl();
           jdbcTokenRepository.setDataSource(dataSource);
           // 自动生成数据库token表
   //        jdbcTokenRepository.setCreateTableOnStartup(true);
           return jdbcTokenRepository;
       }
   }
   ```

3. 配置类中配置自动登录

   ```java
       @Override
       protected void configure(HttpSecurity http) throws Exception {
           http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/hello").permitAll();
           http.exceptionHandling().accessDeniedPage("/unauth.html");
           http.formLogin()
                   .loginPage("/login.html") // 登录页面设置
                   .loginProcessingUrl("/user/login") // 登录访问的路径
                   .defaultSuccessUrl("/success.html").permitAll() // 登录成功之后，跳转路径
                   .and().authorizeRequests()
                   .antMatchers("/", "/test/hello", "/user/login").permitAll() // 设置白名单
                   .antMatchers("/test/index").hasAnyRole("sale", "manager", "admin")
                   .anyRequest().authenticated()
                   .and().rememberMe().tokenRepository(persistentTokenRepository)
                   .tokenValiditySeconds(60) // 设置有效时常
                   .userDetailsService(userDetailsService)
                   .and().csrf().disable(); // 关闭csrf防护
       }
   ```

   
