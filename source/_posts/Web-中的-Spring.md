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

### Spring MVC 的替代方案

#### 自定义 DispatcherServlet 配置

通过重载 `customizeRegistration()` 方法，我们可以进行额外配置。

#### 添加其他的 Servlet 和 Filter

定义任意数量的初始化器类。最简单的方法是实现 Spring 的 `WebApplicationInitializer` 接口。

#### 在 web.xml 中声明 DispatcherServlet

略

### 处理文件上传

#### 配置 multipart 解析器

委托给 Spring 中的 `MultipartResolver` 策略接口的实现。

1. `CommonMultipartResolver`：使用 Jakarta Commons FileUpload 解析 multipart 请求。
2. `StandardServletMultipartResolver`：依赖于 Servlet 3.0 对 multipart 请求的支持。

#### 处理 multipart 请求

`@RequestPart`：作用于控制器方法参数上。

#### 接受 MultipartFile

`MultipartFile` 接口

### 在控制器处理异常

1. 特定的 Spring 异常将自动映射为特定的 HTTP 状态码
2. 异常上添加 `@ResponseStatus` 注解，从而将其映射为某一个 HTTP 状态码
3. 在方法上添加 `@ExceptionHandler` 注解，使其用来处理异常。

#### 将异常映射为 HTTP 状态码

默认情况下，Spring 会将自身的一些异常自动转换成合适的状态码。

`@ResponseStatus` 注解将异常映射为 HTTP 状态码。

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "spittle not found")
public class SpittleNotFoundException extends RuntimeException {
}
```

#### 编写异常处理的方法

`@ExceptionHandler` 当抛出指定的异常时，将会委托该方法处理。它能处理同一个控制器所有方法抛出的这类异常。

### 为控制器添加通知

`@ControllerAdvice` 修饰控制器通知类，这个类会包含一个或者多个如下类型

* `@ExceptionHandler` 注解标注的方法
* `@InitBinder` 注解标注的方法
* `@ModelAttribute` 注解标注的方法

### 跨重定向请求传输数据

对于重定向来说，模型并不能用来传递数据。可以通过其他方案处理：

* 使用 URL 模板以路径变量和 / 或查询参数的形式传递数据
* 使用 flash 属性发送数据

#### 通过 URL 模板进行重定向

占位符自动填充，所有不安全字符会进行转义。

如果属性找不到占位符，会自动以查询参数的形式附加到重定向 URL 上。

```java
model.addAttribute("name", xxx.getName());
return "redirect:/xxx/{name}";
```

#### 使用 flash 属性

```java
model.addFlashAttribute("xxx", xxx);
```

flash 属性会一直携带这些数据知道下一次请求，才回消失。

在执行重定向之前，所有的 flash 属性都会复制到会话中。重定向之后，存在会话中的 flash 属性会被取出，并从会话转移到模型中。
