# 网站安全

在Web开发中，目前使用的安全措施是——过滤器、拦截器

网站设计之初就应该考虑，进行安全保障

shiro、SpringSecurity：很像除了类不同，名字不同

认证、授权

* 功能权限
* 访问权限
* 菜单权限

# SpringSecurity简介

Spring Security是针对Spring项目的安全框架，也是SpringBoot底层安全模块的默认技术选型，他可以实现强大的web安全控制，对于安全控制，我们仅需要引入spring-boot-starter-security模块，进行少量的配置，即可实现强大的安全管理

记住几个类——

* WebSecurityConfigurerAdapter：自定义security策略
* AuthenticationManagerBuilder：自定义认证策略
* @EnableWebSecurity：开启WebSecurity莫事

Spring Security的两个主要目标是“认证“和”授权“（访问控制）

这些概念是通用的，并非只有SpringSecurity中存在

# 组合SpringBoot使用

* 配置类SpringSecurity

  ```java
  @EnableWebSecurity
  public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
      // 授权
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          // 首页所有人可以访问，功能也只有对应有权限的人可以访问
          http.authorizeRequests().antMatchers("/").permitAll()
                  .antMatchers("/level1/**").hasRole("vip1")
                  .antMatchers("/level2/**").hasRole("vip2")
                  .antMatchers("/level3/**").hasRole("vip3");
  
          // 没有权限自动进入登录页，需要手动开启登录页面
          http.formLogin().loginPage("/toLogin").usernameParameter("username").passwordParameter("password").loginProcessingUrl("/login");
  
          // 开启注销功能
          http.csrf().disable(); // 关闭csrf功能
          http.logout().deleteCookies("remove").invalidateHttpSession(true).logoutSuccessUrl("/");
  
          // 记住密码 默认保存两周
          http.rememberMe().rememberMeParameter("rememberMe");
  
          //
  
      }
  
      // SpringBoot 2.1.x之后可以直接使用
      // 密码编码：PasswordEncoder
      // SpringBoot 5.0之后添加了很多的加密方法
      // 认证
      @Override
      protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  
          // 正常情况数据应该从数据库中读
          auth.inMemoryAuthentication().withUser("mnsx").password(passwordEncoder().encode("123123")).roles("vip1")
                  .and().withUser("user").password(passwordEncoder().encode("123123")).roles("vip3", "vip2", "vip1");
      }
  
      @Bean
      public PasswordEncoder passwordEncoder() {
          return new BCryptPasswordEncoder();
      }
  }
  ```

  
