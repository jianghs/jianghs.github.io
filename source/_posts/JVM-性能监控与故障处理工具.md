---
title: JVM-性能监控与故障处理工具
date: 2019-07-28 23:08:16
tags: JVM
categories: coding
---
## jps

jvm process status tool

可以列出正在运行的虚拟机进程，并显示虚拟机执行主类（Main Class，main()函数所在的类）名称以及这些进程的本地虚拟机唯一ID（Local Virtual Machine Identifier，LVMID）。

```java
jps -l
```

## jstat

jstat（JVM Statistics Monitoring Tool）是用于监视虚拟机各种运行状态信息的命令行工具。它可以显示本地或者远程[插图]虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。

## jinfo

jinfo（Configuration Info for Java）的作用是实时地查看和调整虚拟机各项参数。

## jmap

Memory Map for Java 用于生成堆转储快照，一般称为 dump 文件。

## jhat

JVM Heap Analysis Tool,分析 dump 文件。

## jstack

Stack Trace for Java 生成虚拟机当前时刻的线程快照。

## HSDIS

JIT 生成代码反汇编

## JDK 可视化工具

* JConsole: java 监视与管理控制台。
* VisualVM：多合一故障处理工具。
