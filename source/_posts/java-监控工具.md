---
title: java 监控工具
date: 2017-12-02 14:44:51
tags:
---
1. jcmd 用来打印java进程所涉及的基本类、线程、和VM信息
2. jconsole 提供jvm活动的图形化视图，包括线程的使用、类的使用和GC活动。
3. jhat 读取内存对转储
4. jmap 提供堆转储和其他内存使用信息
5. jinfo 查看jvm的系统属性，可以动态设置一些系统属性
6. jstack 转储java进程的栈信息
7. jstat 提供gc和类状态活动的信息
8. jvisualvm 监视jvm的gui工具，可视化GC、实现线程、堆内存。具有插件功能


## 基本的vm信息
1. 获取jvm运行时长

        jcmd process_id VM.uptime


2. 获取系统属性

        jcmd process_id VM.system_properties

3. 获取jvm版本

        jcmd process_id VM.version

4. 获取jvm命令行

        jcmd process_id VM.command_line

5. 获取jvm调优标记,现实命令行设置的标记一级jvm直接设置的标志，加上-all时列出jvm内部所有的标志

        jcmd process_id VM.flags [-all]

## 线程信息
1. jconsole和jvisualvm可以实时显示应用中运行中的线程数量

2. jstack 显示线程的栈信息

        jstack process_id
        jcmd process_id Thread.print

# 类信息
1. jconsole或jstat可以提供应用已使用类的个数。jstat还能提供类编译相关的信息

## 实时gc分析
几乎所有的监控工具都能报告一些gc活动的信息，jconsole可以用实时图显示堆的使用情况，jcmd可以执行gc操作，jmap可以打印堆的概括、永久带信息获取创建堆转储，jstat可以为垃圾收集器正在执行的操作生成许多视图

## 事后堆转储
jvisualvm的gui界面可以捕获堆转储，也可以用命令行jcmd或jmap生成。堆转储是堆使用情况的快照，可以用不同的工具进行分析，包括jvisualvm和jhat

