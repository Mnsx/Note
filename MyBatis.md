# MyBatis简介

## MyBatis特性

1. MyBatis是支持定制化SQL、存储过程以及高级映射得优秀得持久层框架
2. MyBatis避免了几乎所有得JDBC代码和手动设置参数以及获取结果集
3. MyBatis可以使用简单得XML或者注解用于配置和原始映射，将接口和java得POJO映射成数据库中得记录
4. MyBatis是一个半自动的ORM框架

## 和其他持久化层技术对比

* JDBC
  * SQL夹杂在java代码中耦合度高，导致硬编码内伤
  * 维护不易且实际开发需求中SQL有变化，频繁修改的情况多见
  * 代码冗长，开发效率低
* Hibernate和JPA
  * 操作简便，开发效率高
  * 程序中的长难复杂SQL需要绕过框架
  * 内部自动生产的SQL，不容易做特殊优化
  * 基于全映射的全自动框架，大量字段的POJO进行部分映射时比较困难
  * 反射操作太多，导致数据库性能下降
* MyBatis
  * 轻量级，性能出色
  * SQL和java编码分开，功能边界清晰，java代码专注业务、SQL语句专注数据
  * 开发效率稍逊于Hibernamte，但是完全能够接受

# 搭建MyBatis

## 引入依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.7</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.3</version>
        </dependency>
    </dependencies>
```

## 创建MyBatis的核心配置文件

> 习惯上命名为mybatis-config.xml，这个文件名建议，并非强制要求。将来整合Spring之后，这个配置文件可以省略
>
> 核心配置文件主要用于配置连接数据库的环境以及MyBatis的全局配置信息
>
> 核心配置文件存放的位置是src/main/resources目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```

## 创建mapper接口

> MyBatis中的mapper接口相当于dao，但是不需要实现类

```java
public interface UserMapper {
    /**
     * 添加用户信息
     */
    int insertUser();
}
```

## 创建MyBatis的映射文件

1. 映射文件的命名规则：

   表所对应的实体类的类名+Mapper.xml

   因此一个映射文件对应一个实体类，对应一张表的操作

   MyBatis映射文件用于编写SQL，访问以及操作表中的数据

   MyBatis映射文件存放的位置是src/main/resources/mappers目录下

2. MyBatis中可以面向接口操作数据，要保证两个一致：

   mapper接口的全类名和映射文件的名称空间保持一致

   mapper接口中方法的方法名和映射文件中编写SQL的标签的id属性保持一致

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="top.mnsx.mapper.UserMapper">
    <insert id="insertUser">
        insert into t_user values(1, 'admin', '123456', 23, '男', '110@qq.com');
    </insert>
</mapper>   
```

## 测试功能

```java
public class MyBatisTest {

    @Test
    public void testMyBatis() throws IOException {
        InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
        SqlSessionFactory build = sqlSessionFactoryBuilder.build(resourceAsStream);
        SqlSession sqlSession = build.openSession(true);
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        int i = userMapper.insertUser();
        System.out.println(i);
        sqlSession.commit();
        sqlSession.close();
    }
}
```

# 核心配置文件详解

核心配置文件中标签必须按照固定的顺序——

properties——》setting——》typeAliases——》typeHandlers——》objectFactory——》objectWrapperFactory——》reflectorFactory——》plugins——》environments——》databaseIdProvider——》mappers

## environments属性

environments：配置多个连接数据库的环境

​	default：设置默认使用的环境的id

environment：配置莫格具体的环境

​	id：表示连接数据库的环境的唯一标识，不能重复

transactionManager：设置事务管理方式

​	type：jdbc|managed

​		jdbc：表示当前环境中，操作SQL时使用的时jdbc中的原生事务管理方式，事务的提交或者回滚需要手动处理

​		managed：被管理

datasource：配置数据源

​	type：设置数据源的类型（POOLED|UNPOOLED|JNDI）

​	POOLED：表示使用数据库连接池，缓存数据库连接

​	UNPOOLED：不使用数据库连接池

​	JNDI：表示使用上下文中的数据源

## properties属性

```xml
<properties resource="jdbc.properties"/>
```

引入properties文件，通过使用${...}的方式来获取properties中的值

## typeAliases属性

typeAlias：设置某个类型的别名

​	type：设置需要设置别名的类型

​	alias：设置某个类型的别名

​		不设置alias属性那就表示为该类的类名且不区分大小写

package：以包为单位，将包下所有的类型设置默认的类型别名，也就是类名且不区分大小写

```xml
    <typeAliases>
        <typeAlias type="top.mnsx.entity.User" alias="User"/>
    </typeAliases>
```

**类型别名不区分大小写**

## mappers属性

mappers

​	package：以包为单位引入映射文件

要求：

1. mapper接口所在的包和映射文件所在的包一致
2. mapper接口要和映射文件名字一致

```xml
<package name="top.mnsx.mybatis.xml"
```

