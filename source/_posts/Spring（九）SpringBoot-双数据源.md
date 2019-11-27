---
title: Spring（九）SpringBoot 双数据源
comments: true
date: 2019-11-27 18:50:18
categories: Java Web
tags:
 - Java
 - Web
---

# 前言

---

# 配置项

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

---

# 配置数据源

mysql 配置

```java
@Configuration
@MapperScan(basePackages = {"com.jerry.demo.mysql.dao"},
              sqlSessionFactoryRef = "sqlSessionFactoryMysqlBean")
public class MysqlDSConfig{

    @Autowired
    @Qualifier("MYSQL_DS")
    private DataSource mysqlDS;

    @Bean(name = "sqlSessionFactoryMysqlBean")
    public SqlSessionFactoryBean SqlSessionFactoryBean() throws IOException{
      SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
      bean.setDataSource(mysqlDS);
      bean.setMapperLocations(new PathMatchingResourcePatternResolver()
                        .getResources("classpath:mybatis/oracle/mapper/*.xml");)
      return bean;
    }

}
```

oracle 配置

```java
@Configuration
@MapperScan(basePackages = {"com.jerry.demo.oracle.dao"})
public class MysqlDSConfig{

    @Autowired
    @Qualifier("MYSQL_DS")
    private DataSource mysqlDS;

    @Bean(name = "sqlSessionFactoryOracleBean")
    @Primary
    @ConfigurationProperties(prefix = "mybatis")
    public SqlSessionFactoryBean SqlSessionFactoryBean() throws IOException{
      SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
      bean.setDataSource(mysqlDS);
      bean.setMapperLocations(new PathMatchingResourcePatternResolver()
                        .getResources("classpath:mybatis/oracle/mapper*.xml");)
      return bean;
    }

}
```
