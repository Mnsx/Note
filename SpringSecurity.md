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

