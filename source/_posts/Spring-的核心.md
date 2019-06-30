---
title: Spring 的核心
date: 2019-06-30 14:51:01
tags: Spring
categories: coding
---
## Spring 核心

### 依赖注入（DI）或者控制反转（IoC）

对象的依赖关系将由系统中负责协调各对象的第三方组件在创建对象的时候进行设定。对象无需自行创建或者管理它们的依赖关系。

创建应用组件之间协作的行为通常称为装配。Spring 通常的装配方式：XML、Java 描述。

### 面向切面（AOP）

允许把遍布应用各处的功能分离出来形成可重用的组件。

### Spring 容器

#### bean 工厂

BeanFactory 是最简单的容器，提供最基本的 DI 支持。

#### 上下文

基于 BeanFactory 构建，并提供框架级别服务。

* AnnotationConfigApplicationContext
* AnnotationConfigWebApplicationContext
* ClassPathXmlApplicationContext
* FileSystemXmlApplicationContext
* XmlWebApplicationContext

#### bean 的生命周期

1. 实例化
2. 填充属性
3. 调用 BeanNameAware 的 setBeanName()方法
4. 调用 BeanFactoryAware 的 setBeanFactory()方法
5. 调用 ApplicationContextAware 的 setApplicationContext() 方法
6. 调用 BeanPostProcessor 的预初始化方法 postProcessBeforeInitialization()
7. 调用 InitializingBean 的 afterPropertiesSet() 方法
8. 调用自定义的初始化方法
9. 调用 BeanPostProcesser 的初始化后方法 postProcessAfterInitialization()
10. 调用 DisposableBean 的 destory() 方法。
11. 调用自定义的销毁方法

1-9步 bean 已经准备就绪，可以被应用程序使用，它们将一直驻留在应用上下文中，直至应用上下文被销毁。

## 装配 Bean

### 装配的可选方案

#### 隐式的 bean 发现机制和自动装配

* 组件扫描: Spring 会自动发现应用上下文中所创建的 bean.
* 自动装配: Spring 自动满足 bean 之间的依赖。

`@Component(id)` ：表明该类会作为组件类，并告知 Spring 要为这个类创建 bean. id 缺省时，将类名的第一个字母变小写。

`@Named(id)` ：Java 依赖注入规范，可作为 `@Component(id)`  的替代方案。

`@Configuration`：表明该类作为配置类。

`@ComponentScan(basepackages="com.xxx", "com.yyy")`：这个注解能够在 Spring 中启用组件扫描，默认会扫描与配置类相同的包。`basepackages` 可以设置多个扫描的基础包。

`@Autowired`：声明要自动装配。可以设置在构造器、Setter 方法或者其他方法上。

#### 在 Java 中进行显式配置

1. 创建配置类
2. 声明简答的 bean
3. 借助 JavaConfig 实现注入

`@Bean(name="xxx")`：告诉 Spring 这个方法将要返回一个对象，这个对象要注册为 Spring 应用上下文的 bean. name 可以缺省。

#### 在 XML 中进行显式配置

不做介绍

### 导入和混合配置

#### JavaConfig 中引入 XML 配置

`@Import({xxx.class, yyy.class})`:引入多个 Java 配置类。

`@ImportResource("classpath:xxx.xml")`:引入 xxx.xml 配置文件。

#### 在 XML 配置中引入 JavaConfig

## 高级装配

### Spring profile

`@Profile`：指定某个 bean 属于哪个 profile

#### 配置 profile bean

下面的例子为 dev 环境及 prd 环境配置数据源，dev 环境采用内置数据库，prd 正式环境采用 JNDI.

```java
package com.jianghs.config;

import com.jianghs.model.DataSourceExistsCondition;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseBuilder;
import org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType;
import org.springframework.jndi.JndiObjectFactoryBean;

import javax.sql.DataSource;

/**
 * @author jianghongsen
 * @title: DataSourceConfig
 * @projectName spring-study
 * @description: TODO
 * @date 2019-06-2317:34
 */
@Configuration
public class DataSourceConfig {

    /**
     * @description: Conditional 表明是个条件 bean
     * @param
     * @return javax.sql.DataSource
     * @throws
     * @author jianghongsen
     * @date 2019-06-23 18:08
     */
    @Bean
    @Profile("dev") // 为 dev profile 装配的 bean
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder()
                .setType(EmbeddedDatabaseType.H2)
                .addScript("")
                .addScript("")
                .build();
    }

    @Bean
    @Profile("prd") // 为 prd profile 装配的 bean
    public DataSource prdDataSource() {
        JndiObjectFactoryBean jndiObjectFactoryBean = new JndiObjectFactoryBean();
        jndiObjectFactoryBean.setJndiName("");
        jndiObjectFactoryBean.setResourceRef(true);
        jndiObjectFactoryBean.setProxyInterface(javax.sql.DataSource.class);
        return (DataSource)jndiObjectFactoryBean.getObject();
    }
}
```

#### 激活 profile

* spring.profiles.active: 激活的 profile
* spring.profiles.default: 默认的 profile

如果没有设置 `spring.profiles.active` 属性的话，那么 Spring 将查找 `spring.profiles.default` 的值。如果均没有设置的话，那就没有激活的 profile.

* 作为 DispatcherServlet 的初始化参数；
* 作为 Web 应用的上下文参数；
* 作为 JNDI 条目；
* 作为 环境变量；
* 作为 JVM 系统属性；
* 在集成测试类上，使用 `@ActiveProfiles` 注解设置；

### 条件化的 bean 声明

`@Conditioanl(xxx.class)`：可以用到带有 `@Bean` 注解的方法上。如果给定的条件计算结果为 true，就会创建这个 bean，否则的话，这个 bean 被忽略。

`ConditionContext` 接口

`AnnotatedTypeMetadata` 接口

```java
package com.jianghs.model;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * @author jianghongsen
 * @title: DataSourceExistsCondition
 * @projectName spring-study
 * @description: TODO
 * @date 2019-06-2318:04
 */
public class DataSourceExistsCondition implements Condition {

    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment env = conditionContext.getEnvironment();
        return env.containsProperty("magic");
    }
}

```

### 自动装配与歧义性

如果多个 bean 可以匹配结果，这种歧义性会阻碍 Spring 自动装配属性、构造器参数或方法参数。

#### 标示首选的 bean

`@Primary`：可以和 `@Component` 或者 `@Bean` 搭配使用。

#### 限定自动装配的 bean

`@Qualifier(xxx)`：使用限定符的主要方式。

### bean 的作用域

* 单例（Singleton）:默认情况，整个应用，只创建一个 bean 的实例。
* 原型（Prototype）:每次注入或者通过上下文获取的时候，都会创建一个新的 bean 实例。
* 会话（Session）:在 Web 应用中，为每个会话创建一个 bean 实例。
* 请求（Request）:在 Web 应用中，为每个请求创建一个 bean 实例。

`@Scope(ConfigurableBeanFactory.xxx)`：标示作用域范围。

### 运行时值注入

#### 属性占位符

`@PropertySource("classpath:xxx/yyy/app.properties")`：引入资源配置文件

占位符形式：${...}

#### Spring 表达式语言（SpEL）

形式：#{...}

## 面向切面

### 概念

AOP 可以实现横向关注点（日志、声明式事务、安全、缓存等）与业务逻辑分离。

#### 术语

* 通知（Advice）:切面需要完成的工作，定义了切面是什么及何时使用。
  * 前置通知（Before）：在目标方法被调用之前调用通知功能。
  * 后置通知（After）：在目标方法被调用之后调用通知功能。
  * 返回通知（After-returning）：在目标方法成功执行后调用通知。
  * 异常通知（After-throwing）：在目标方法抛出异常之后调用通知功能。
  * 环绕通知（Around）：在目标方法调用前后执行通知功能。
* 连接点（Join Point）：应用通知的时机。可以是调用方法时、抛出异常时、修改字段时等。
* 切点（Pointcut）：切点的定义会匹配通知要织入的一个或者多个连接点。
* 切面（Aspect）：通知和切点的结合。通知和切点定义了切面的全部内容-它是什么，在何时和何处完成其功能。
* 引入（Introduction）：允许我们向现有的类添加新方法或属性。
* 织入（Weaving）：把切面应用到目标对象并创建新的代理对象的过程。
  * 编译期：AspectJ
  * 类加载期：AspectJ
  * 运行期：SpringAOP

#### Spring 对 AOP 的支持

* 基于代理的经典 Spring AOP;
* 纯 POJO 切面;
* `@AspectJ` 注解驱动的切面;
* 注入式 AspectJ 切面。

1、基于代理的经典 Spring AOP 使用 ProxyFactory Bean,过于复杂和笨重。

2、纯 POJO 切面需要 XML 技术。

3、`@AspectJ` 注解驱动的切面，能够不使用 XML 来完成功能。

#### 关键知识

* Spring 通知是 Java 编写的
* Spring 在运行时通知对象
* Spring 只支持方法级别的连接点

### 通过切点来选择连接点

切点用于准确定位应该在什么地方应用切面的通知。

`execution()` 指示器是实际执行匹配的。

#### 编写切点

```java
execution(* 包名.类名.方法(..))
```

`*` 表明不关心返回类型，`..` 表明切点要选择任意参数列表的方法。

#### 在切点中选择 bean

`bean()` 指示器允许在切点表达式中使用 bean 的 ID 来标识 bean.

```java
execution(* 包名.类名.方法(..)) and bean('aaa')
```

限定 bean 的 ID 为 aaa.

```java
execution(* 包名.类名.方法(..)) and !bean('aaa')
```

限定 bean 的 ID 除了 aaa.

### 使用注解创建切面

`@Aspect` 创建切面。

`@PointCut` 能够在 `@Aspect` 切面内定义可重用的切点。

```java
package com.jianghs.concert;

import org.aspectj.lang.annotation.*;

/**
 * @author jianghongsen
 * @title: Audience
 * @projectName spring-study
 * @description: TODO
 * @date 2019-06-3022:36
 */

@Aspect
public class Audience {

    @Pointcut("execution(* com.jianghs.concert.Performance.perform(..))")
    public void performance() {}

    @Before("performance()")
    public void silenceCellPhones() {
        System.out.println("手机静音");
    }

    @Before("performance()")
    public void takeSeats() {
        System.out.println("就坐");
    }

    @AfterReturning("performance()")
    public void applause() {
        System.out.println("鼓掌");
    }

    @AfterThrowing("performance()")
    public void demandRefund() {
        System.out.println("退款");
    }
}
```

`@EnableAspectJAutoProxy` 启用自动代理功能。

```java
package com.jianghs.concert;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

/**
 * @author jianghongsen
 * @title: ConcertConfig
 * @projectName spring-study
 * @description: TODO
 * @date 2019-06-3022:51
 */

@Configuration
@EnableAspectJAutoProxy
public class ConcertConfig {

    @Bean
    public Audience audience() {
        return new Audience();
    }
}
```

### 创建环绕通知

一定不能忘了调用 `proceed()`.

```java
package com.jianghs.concert;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;

/**
 * @author jianghongsen
 * @title: Audience
 * @projectName spring-study
 * @description: TODO
 * @date 2019-06-3022:36
 */

@Aspect
public class Audience {

    @Pointcut("execution(* com.jianghs.concert.Performance.perform(..))")
    public void performance() {}

    @Around("performance()")
    public void watchPerformance(ProceedingJoinPoint joinPoint) {
        try {
            System.out.println("手机静音");
            System.out.println("就坐");
            joinPoint.proceed();
            System.out.println("鼓掌");
        } catch (Throwable e) {
            System.out.println("退款");
        }
    }
}

```

### 处理通知中的参数

`args()` 限定符会将参数传递到通知中。

### 在 XML 中声明切面

当无法获得通知类的源码添加注解的时候，需要通过 XML 作为补充。

### 注入 AspectJ 切面

当 Spring AOP 无法满足要求时，需要通过 AspectJ 实现。
