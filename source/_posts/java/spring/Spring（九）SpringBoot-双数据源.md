---
title: Spring（九）SpringBoot 双数据源
categories:
- Java Web
- Spring
tags:
  - Java
  - Web
abbrlink: a0972c39
date: 2019-11-27 18:50:18
toc: true
---

# 前言

最近项目中需要用到 Springboot + Mybatis 双数据源，一边接入 Oracle，一边接入 MySQL ，折腾了一下。搞了个 demo 出来。

<!-- more -->

---

# 修改配置项

先自定义两个datasource：

application.yml

```
spring:
  datasource-oracle:
    driver-class-name: oracle.jdbc.driver.OracleDriver
    jdbc-url:
    username:
    password:
  datasource-mysql:
    driver-class-name: com.mysql.jdbc.Driver
    jdbc-url:
    username:
    password:
```

---

# 定义数据源Bean

通过 java config 的方式，定义两个 bean ：

DSConfig

```java
@Configuration
public class DSconfig {

    @Bean(name = "ORACLE_DS")
    @ConfigurationProperties(prefix = "spring.datasource-oracle")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "MYSQL_DS")
    @ConfigurationProperties(prefix = "spring.datasource-mysql")
    public DataSource secondDataSource() {
        return DataSourceBuilder.create().build();
    }
}
```

其中，`@ConfigurationProperties` 对应配置项里自定义的名称。

---

# 配置双数据源

有了数据源的 bean 之后，需要进行一些配置，不同的 datasource bean 创建不同的 SqlSession，注意，必须指定其中一个为 `@Primary`

mysql 配置

```java
@Configuration
@MapperScan(basePackages = {"com.jerry.demo.mysql.dao"},
              sqlSessionFactoryRef = "sqlSessionFactoryMysqlBean" )
public class MysqlDSConfig{

    @Autowired
    @Qualifier("MYSQL_DS")
    private DataSource mysqlDS;

    @Bean(name = "sqlSessionFactoryMysqlBean")
    public SqlSessionFactoryBean SqlSessionFactoryBean() throws IOException{
      SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
      bean.setDataSource(mysqlDS);
      bean.setMapperLocations(new PathMatchingResourcePatternResolver()
                        .getResources("classpath:mybatis/mysql/mapper/*.xml");)
      return bean;
    }

}
```



oracle 配置

```java
@Configuration
@MapperScan(basePackages = {"com.jerry.demo.oracle.dao"})
public class OracleDSConfig{

    @Autowired
    @Qualifier("ORACLE_DS")
    private DataSource oracleDS;

    @Bean(name = "sqlSessionFactoryOracleBean")
    @Primary
    public SqlSessionFactoryBean SqlSessionFactoryBean() throws IOException{
      SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
      bean.setDataSource(mysqlDS);
      bean.setMapperLocations(new PathMatchingResourcePatternResolver()
                        .getResources("classpath:mybatis/oracle/mapper*.xml");)
      return bean;
    }

}
```

接下来就可以到对应的目录下编写 xml 的 SQL 语句了。

---

# 在dao层指定数据源

有时候我们想通过指定某一个 dao 使用指定的数据源，可以再数据源配置的 MapperScan 注解上上加 `annotationClass`

```java
@Configuration
@MapperScan(basePackages = {"com.jerry.demo.mysql.dao"},
              sqlSessionFactoryRef = "sqlSessionFactoryMysqlBean",
               annotationClass = me.jerrysheh.demo.annotation.MysqlMapper.class))
public class MysqlDSConfig{
```

然后自定义一个注解标记

```java
package me.jerrysheh.demo.annotation;

/**
 * @author jerrysheh
 * @date 2020/7/4
 */
public @interface MysqlMapper {
}

```


这样 mybatis 会自动去扫 basePackages 下面所有标记了 `MysqlMapper` 注解的接口，使用 mysql 数据源。

```java
/**
 * @author jerrysheh
 * @date 2020/7/4
 */
@MysqlMapper
public interface ProductMapper {

    Object selectOne();

    List selectAll();

    // ...

}

```

---

- demo参考github：https://github.com/JerrySheh/springboot-mybatis-multidatasource-demo
