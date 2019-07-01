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

## 渲染 Web 视图

## Spring MVC 的高级技术

## 使用 Spring Web Flow

## 保护 Web 应用
