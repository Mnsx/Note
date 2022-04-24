# Spring和SpringBoot

## 为什么使用SpringBoot

> Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run".
>
> 
>
> 能快速创建出生产级别的Spring应用

## SpringBoot优点

- Create stand-alone Spring applications

- - 创建独立Spring应用

- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)

- - 内嵌web服务器

- Provide opinionated 'starter' dependencies to simplify your build configuration

- - 自动starter依赖，简化构建配置

- Automatically configure Spring and 3rd party libraries whenever possible

- - 自动配置Spring以及第三方功能

- Provide production-ready features such as metrics, health checks, and externalized configuration

- - 提供生产级别的监控、健康检查及外部化配置

- Absolutely no code generation and no requirement for XML configuration

- - 无代码生成、无需编写XML

> SpringBoot是整合Spring技术栈的一站式框架
>
> SpringBoot是简化Spring技术栈的快速开发脚手架

## SpringBoot缺点

* 人称版本帝，迭代快需要时刻关注变化
* 封装太深，内部原理复杂，不容易精通

## 微服务

* 微服务是一种架构风格
* 一个应用拆分为一组小型服务
* 每个服务运行在自己的进程内，也就是可独立部署和升级

- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署
- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

## 分布式

![img](D:\Picture\Note\SpringBoot\分布式模型)

**分布式的解决**

SpringBoot + SpringCloud

![img](D:\Picture\Note\SpringBoot\Spring微服务解决)

## 云原生

原生应用如何上云。Cloud Native

# SpringBoot2入门

## 修改Maven镜像

```xml
<mirrors>
      <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>central</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
      </mirror>
  </mirrors>
 
  <profiles>
         <profile>
              <id>jdk-1.8</id>
              <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>1.8</jdk>
              </activation>
              <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
              </properties>
         </profile>
  </profiles>
```

## 创建Maven工程

## 引入依赖

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
    </parent>


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

    </dependencies>
```

## 创建主程序

```java
/**
 * 主程序类
 * @SpringBootApplication: 这是一个SpringBoot应用
 */
@SpringBootApplication
public class Boot01HelloworldApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot01HelloworldApplication.class, args);
    }

}
```

## 编写业务

```java
//@Controller
//@ResponseBody

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String handle01() {
        return "Hello Spring Boot!";
    }
}
```

## 测试

直接进入main方法运行

## 简化配置

```properties
server.port = 8888
```

## 简化部署

```xml
	<build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

将项目打成jar包，直接在目标服务器执行即可

# 自动配置原理

## 依赖管理

```xml
<!--本地项目的父项目-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>

<!--本地项目的父项目的父项目-->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.4.RELEASE</version>
</parent>
```

**几乎声明了所有开发中常用的依赖的版本号,自动版本仲裁机制**

* 开发导入starter场景启动器

  * spring-boot-starter-\*
  * 只要引入starter，这个场景的所有常规依赖，就会自动引入
  * 通过Spring-boot官方文档可以查看官方提供的场景启动器
  * \*-spring-boot-start，是第三方提供的场景启动器

  ```xml
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  ```

* 无需关注版本号，自动仲裁

  * 使用非版本仲裁的依赖时需要自己写入引入版本号

* 可以修改版本号

  * 查看spring-boot-dependencies里面规定的当前依赖的版本
  * 在当前项目里面重写配置

  ```xml
  <!--重写案例-->
  <properties>
  	<mysql.version>5.1.43</mysql.version>
  </properties>
  ```

## 自动配置

* 自动配置Tomcat

  * 引入Tomcat依赖
  * 配置Tomcat

  ```xml
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <version>2.6.7</version>
        <scope>compile</scope>
      </dependency>
  ```

* 自动配置Spring

  * 引入SpringMVC全套组件
  * 自动配置SpringMVC常用组件

  ```xml
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.19</version>
        <scope>compile</scope>
      </dependency>
      <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.19</version>
        <scope>compile</scope>
      </dependency>
  ```

  * SpringBoot配置好了所有Web开发常见场景

  * 默认包结构

    * 主程序所在包以及其下所有子包里面的组件都会默认扫描进来

    * 无需配置包扫描

    * 想要该表扫描路径——

      `@SpringBootApplication(scanBasePackages="top.mnsx")`

      `@ComponentScan 指定扫描路径`

      ```java
      @SpringBootApplication(scanBasePackages="top.mnsx")
      等同于
      @SpringBootConfiguration
      @EnableAutoConfiguration
      @ComponentScan("top.mnsx")
      ```

  * 各种配置拥有默认值

    * 默认配置最终都是映射到对应的properties文件中
    * 配置文件的值最中会绑定每个类上，这个类会在容器中创建对象

  * 按需加载所有自动配置项

    * 非常多的starter
    * 引入了场景，这个场景的自动配置才会开启
    * SpringBoot所有的自动配置功能都在`spring-boot-autoconfigure`包里面

## 容器功能

1. 组件添加

   * @Configuration

     Full模式和Lite模式

     **最佳实战**

     * 配置 类组件之间无依赖关系用Lite模式加速容器启动过程，减少判断
     * 配置类组件之间有依赖关系，方法会被调用得到之前单实例组件，用Full模式

     ```java
     /**
      * 配置类里面使用@Bean标注在方法上给容器注册主键，默认也是单实例的
      * 配置类本身也是组件
      * proxyBeanMethods：代理bean方法
      * Full(proxyBeanMethods = true，
      * Lite(proxyBeanMethods = false
      */
     @Configuration(proxyBeanMethods = true) // 告诉SpringBoot这是一个配置类 == 配置文件
     public class MyConfig {
     
         /**
          * 外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
          */
         @Bean // 给容器添加组件。以方法名作为组件的id，返回类型就是组件类型，返回的值就是组件在容器中的实例
         public User user01() {
             User user = new User("mnsx", 20);
             user.setPet(tomcatPet());
             return user;
         }
     
         @Bean("pet01")
         public Pet tomcatPet() {
             return new Pet("tomcat");
         }
     }
     ```

     ```java
     /**
      * 主程序类
      * @SpringBootApplication: 这是一个SpringBoot应用
      */
     //@ComponentScan("")
     //@SpringBootApplication(scanBasePackages="top.mnsx")
     @SpringBootConfiguration
     @EnableAutoConfiguration
     @ComponentScan("top.mnsx")
     public class Boot01HelloworldApplication {
     
         public static void main(String[] args) {
             // 1.返回IOC容器
             ConfigurableApplicationContext run = SpringApplication.run(Boot01HelloworldApplication.class, args);
     
             // 2.查看容器里的组件
             String[] names = run.getBeanDefinitionNames();
             for (String name : names) {
                 System.out.println(name);
     
             }
             
             // 3.从容器中获取组件
     
             Pet pet01 = run.getBean("pet01", Pet.class);
             Pet pet02 = run.getBean("pet01", Pet.class);
             System.out.println("组件" + (pet01 == pet02));
     
             // top.mnsx.boot_01_helloworld.confit.MyConfig$$EnhancerBySpringCGLIB$$3c423deb@6eafb10e
             MyConfig bean = run.getBean(MyConfig.class);
             System.out.println(bean);
     
             // 如果@Configuration(proxyBeanMethods = true)代理对象调用方法。SpringBoot总会检查这个组件是否在容器中有
             // 保持组件单例
             User user01 = bean.user01();
             User user02 = bean.user01();
             System.out.println(user01 == user02);
     
             User user03 = run.getBean("user01", User.class);
             Pet pet03 = run.getBean("pet01", Pet.class);
     
             System.out.println("用户的宠物：" + (user03.getPet() == pet03));
         }
     
     }
     ```

     ```java
     @Data
     @AllArgsConstructor
     @NoArgsConstructor
     public class User {
         private String name;
         private Integer age;
         private Pet pet;
     
         public User(String name, Integer age) {
             this.name = name;
             this.age = age;
         }
     }
     ```

     ```java
     @Data
     @AllArgsConstructor
     @NoArgsConstructor
     public class Pet {
         private String name;
     }
     ```

   
   * @Bean、@Component、@Contoller、@Service、@Repository
   
   * @ComponentScan、@Import
   
     ```java
     /**
      * 配置类里面使用@Bean标注在方法上给容器注册主键，默认也是单实例的
      * 配置类本身也是组件
      * proxyBeanMethods：代理bean方法
      * Full(proxyBeanMethods = true，
      * Lite(proxyBeanMethods = false
      *
      *
      * @Import: 给容器中自动创建出指定类型的组件，默认组件的名字就是全类名
      */
     @Import({User.class})
     @Configuration(proxyBeanMethods = true) // 告诉SpringBoot这是一个配置类 == 配置文件
     public class MyConfig {
     ```
   
   * @Conditional
   
     条件装配：满足Conditional指定的条件，则进行组件注入
   
     ```java
     /**
      * 配置类里面使用@Bean标注在方法上给容器注册主键，默认也是单实例的
      * 配置类本身也是组件
      * proxyBeanMethods：代理bean方法
      * Full(proxyBeanMethods = true，
      * Lite(proxyBeanMethods = false
      *
      *
      * @Import: 给容器中自动创建出指定类型的组件，默认组件的名字就是全类名
      */
     @Import({User.class})
     @Configuration(proxyBeanMethods = true) // 告诉SpringBoot这是一个配置类 == 配置文件
     public class MyConfig {
     
         /**
          * 外部无论对配置类中的这个组件注册方法调用多少次获取的都是之前注册容器中的单实例对象
          */
         @Bean("user01") // 给容器添加组件。以方法名作为组件的id，返回类型就是组件类型，返回的值就是组件在容器中的实例
         public User user01() {
             User user = new User("mnsx", 20);
             user.setPet(tomcatPet());
             return user;
         }
     
         @Bean("pet01")
         @ConditionalOnBean(name = "user01")
         public Pet tomcatPet() {
             return new Pet("tomcat");
         }
     }
     ```
   
     ```java
             System.out.println("容器中pet01组件" + run.containsBean("pet01"));
     
             System.out.println("容器中user01组件" + run.containsBean("user01"));
     ```
   
     
