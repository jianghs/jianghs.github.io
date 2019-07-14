---
title: 后端中的 Spring
date: 2019-07-09 23:10:11
tags: Spring
categories: coding
---

## Spring 与 JDBC

### Spring 的数据访问哲学

数据访问功能的组件称为数据访问对象 DAO 或者 Repository。

#### 数据访问的异常体系

Spring 提供的平台无关的持久化异常。解决了两个问题：

* 解决 JDBC 异常过于简单
* 解决了类似了 Hibernate 异常体系其本身独有的问题，即平台无关性。

#### 数据访问模板化

模板方法将过程中与特定实现相关的部分委托给接口，而这个接口的不同实现定义了过程中的具体行为。

Spring 将数据访问过程中固定的和可变的部分划分为：模板和回调。

1. 模板：事务控制、资源管理、处理异常
2. 回调：语句、绑定参数、整理结果集

### 配置数据源

1. 通过 JDBC 驱动程序定义的数据源
2. 通过 JNDI 查找的数据源
3. 连接池的数据源

#### 使用 JNDI 数据源

优点：

1. 数据源独立于应用程序管理。
2. 应用服务器管理的数据源通常以池的方式组织，具备更好的性能。
3. 支持系统管理员对其进行热切换。

```java
@Bean
public JndiObjectFactoryBean dataSource() {
    JndiObjectFactoryBean jndiObjectFactoryBean = new JndiObjectFactoryBean();
    jndiObjectFactoryBean.setJndiName("jdbc/xxx");
    jndiObjectFactoryBean.setResourceRef(true);
    jndiObjectFactoryBean.setProxyInterface(javax.sql.DataSource.class);
    return jndiObjectFactoryBean;
}
```

#### 使用数据源连接池

1. Apache Common DBCP
2. c3p0
3. BoneCP

```java
@Bean
public BasicDataSource dataSource() {
    BasicDataSource dataSource = new BasicDataSource();
    dataSource.setDriverClassName("org.h2.Driver");
    dataSource.setUrl("jdbc:h2:tcp://localhost/~/spitter");
    dataSource.setUsername("jianghs");
    dataSource.setPassword("jianghs");
    dataSource.setInitialSize(5);
    dataSource.setMaxActive(10);
    return dataSource;
}
```

#### 基于 JDBC 的数据源

不推荐用在生产环境。

* DriverManagerDataSource：每个连接请求都会返回一个新建的连接，未池化。
* SimpleDriverDataSource：与 DriverManagerDataSource 类似，解决特定环境下的类加载问题。
* SingleConnectionDataSource：每个连接请求都会返回同一个连接，可以视为只有一个连接的池。

```java
@Bean
public DataSource dataSourceJdbc() {
    DriverManagerDataSource driverManagerDataSource = new DriverManagerDataSource();
    driverManagerDataSource.setDriverClassName("org.h2.Driver");
    driverManagerDataSource.setUrl("jdbc:h2:tcp://localhost/~/spitter");
    driverManagerDataSource.setUsername("jianghs");
    driverManagerDataSource.setPassword("jianghs");
    return driverManagerDataSource;
}
```

#### 使用嵌入式数据源

适用于开发和测试环境。

* H2
* Apache Derby

#### 使用 profile 选择数据源

参见 《Spring 的核心》章节。

### 在 Spring 中使用 JDBC

使用 JdbcTemplate 来插入数据，实例略。

## 使用对象-关系映射持久化数据

### 集成 Hibernate

### Spring 与 Java 持久化 API

### 借助 Spring Data 实现自动化的 JPA Repository

## 缓存数据

### 启用对缓存的支持

### 为方法添加注解以支持缓存

### 使用 XML 声明缓存

## 使用 NoSQL 数据库

### MongoDB

### Neo4j

### Redis
