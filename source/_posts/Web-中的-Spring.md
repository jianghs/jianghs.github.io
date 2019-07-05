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

## Spring MVC 的高级技术

## 使用 Spring Web Flow

## 保护 Web 应用
