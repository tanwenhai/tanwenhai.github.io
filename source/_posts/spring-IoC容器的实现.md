---
title: spring IoC容器的实现
date: 2017-11-01 14:58:31
tags:
---
1. IoC控制反转 依赖对象不由自身实现，由外部注入，在具体的注入实现中，接口注入、setter注入和构造器注入是主要的注入方式
    - 对象生成或初始化时直接将数据注入到对象中
    - 通过将对象引用注入到对象数据域中
2. 在Spring IoC容器的设计中，有两个主要的容器系列，一个是实现BeanFactory接口的简单容器系列，另一个是ApplicationContext应用上下文，**应用上下文在简单容器的基础上，增加了许多面向框架的特性，同时对应用环境做了许多适配**。
    - {% asset_img BeanFactory.png BeanFactory继承图 %}
    - 在这些Spring提供的基本IoC容器的接口定义和实现的基础上，Spring通过**定义BeanDefinition来管理基于Spring的应用中的各种对象以及它们之间的相互依赖关系**，BeanDefinition抽象了对Bean的定义，对Ioc容器来说BeanDefinition就是对依赖反转模式中管理的对象依赖关系
    - {% asset_img IoC容器接口设计图.png IoC容器接口设计图 %}
3. IoC容器的初始化过程由*AbstractApplicationContext#refresh*开始,包括BeanDefinition载入、注入
    - 第一个过程是Resource定位，Resource定位指的是BeanDefinition的资源定位，它由ResourceLoader通过统一的Resouce接口来完成
    - 第二个过程是BeanDefinition的载入，载入过程是把用户定义的Bean表示成IoC容器内部的数据结构
    - 第三个过程是向Ioc容器注册这些BeanDefinition的过程，通过调用BeanDefinitionRegistry接口的实现来完成
