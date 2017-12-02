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
jconsole和jvisualvm可以实时
