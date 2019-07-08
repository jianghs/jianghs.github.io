---
title: Web 中的 Spring
date: 2019-07-01 22:08:13
tags: Spring
categories: coding
---

## 构建 Spring Web 应用

Spring MVC 基于模型-视图-控制器模式实现。

### Spring MVC 原理

1. `DispatcherServlet` 作为前端控制器，将请求发送给 Spring MVC 的控制器(`controller`)。通过查询一个或者多个处理器映射（`handler mapping`）来确定请求的控制器。

2. `controller` 处理完逻辑后，将模型数据打包，并标识出渲染输出的视图名，返回给 `DispatcherServlet`。

3. `DispatcherServlet` 使用视图解析器(`view resolver`)来将逻辑视图名匹配为一个特定的视图实现。

4. 最后一步就是视图的实现。

### 搭建 Spring MVC

#### 配置 DispatcherServlet

继承 `AbstractAnnotationConfigDispatcherServletInitializer` 类

1. getServletMappings()：将一个或者多个路径映射到 `DispatcherServlet`。
2. getRootConfigClasses()：该方法获取的配置类由 `ContextLoaderListener` 生成，对应的 bean 通常是驱动应用后端的中间层和数据层组件。
3. getServletConfigClasses()：该方法获取的配置类由 `DispatcherServlet` 生成。

#### 启用 Spring MVC

通过注解 `@EnableWebMvc` 启用。

#### 其他

1. 视图解析器
2. 静态资源处理

### 编写基本控制器

`@Controller` 声明是个控制器，作用于方法或者类。

`@RequestMapping(value="", method=xxx)` value 指定请求的路径，method 细化了 HTTP 请求处理的方法，可以作用于方法或者类。

#### 传递模型数据到视图中

1. Model\Map key 值的自动推断
2. 返回视图的自动推断

#### 请求输入的输入

1. 查询参数：`@RequestParam("xxx")`
2. 表单参数：展现表单、处理表单提交的数据、校验表单
3. 路径变量：通过添加占位符`{` xx `}` 实现。`@PathVaraiable("xx")` 表明占位符不论传什么值都会到 xx 参数。

## 渲染 Web 视图

### 创建 JSP 视图

#### 配置使用于 JSP 的视图解析器

推荐使用 `InternalResourceViewResolver` 来创建。

```java
public ViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views");
    resolver.setSuffix(".jsp");
    resolver.setExposeContextBeansAsAttributes(true);
    return resolver;
}
```

#### 解析 JSTL 视图

只需设置它的 viewClass 属性即可

```java
resolver.setViewClass(JstlView.class);
```

#### 使用 Spring 的 JSP 库

1. 渲染 HTML 表单标签
2. 工具类标签

### Apache Titles 视图定义布局

略

### 使用 Thymeleaf

#### 配置 Thymeleaf 视图解析器

1. 模板解析器
2. 模板引擎
3. 视图解析器

```java
@Bean
public SpringResourceTemplateResolver templateResolver(){
    SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();
    templateResolver.setApplicationContext(this.applicationContext);
    templateResolver.setPrefix("/WEB-INF/templates/");
    templateResolver.setSuffix(".html");
    // HTML is the default value, added here for the sake of clarity.
    templateResolver.setTemplateMode(TemplateMode.HTML);
    // Template cache is true by default. Set to false if you want
    // templates to be automatically updated when modified.
    templateResolver.setCacheable(true);
    return templateResolver;
}

@Bean
public SpringTemplateEngine templateEngine(){
    // SpringTemplateEngine automatically applies SpringStandardDialect and
    // enables Spring's own MessageSource message resolution mechanisms.
    SpringTemplateEngine templateEngine = new SpringTemplateEngine();
    templateEngine.setTemplateResolver(templateResolver());
    // Enabling the SpringEL compiler with Spring 4.2.4 or newer can
    // speed up execution in most scenarios, but might be incompatible
    // with specific cases when expressions in one template are reused
    // across different data types, so this flag is "false" by default
    // for safer backwards compatibility.
    templateEngine.setEnableSpringELCompiler(true);
    return templateEngine;
}

@Bean
public ThymeleafViewResolver viewResolver(){
    ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
    viewResolver.setTemplateEngine(templateEngine());
    return viewResolver;
}
```

#### 定义 Thymeleaf 模板

使用自定义的命名空间 `xmlns:th="http://www.thymeleaf.org"`

Thymeleaf 方言 `@{}`

## Spring MVC 的高级技术

## 使用 Spring Web Flow

## 保护 Web 应用
