---
layout: spring
title: Ribbon
date: 2017-09-30 09:19:07
tags: Spring Cloud Ribbon
---
Spring Cloud Ribbon 是一个基于HTTP和TCP的客户端负载均衡工具
通过@LoadBalancer给RestTemplate增加服务负载均衡支持
1. 创建一个LoadBalancerInterceptor的Bean,用于实现对客户端发起请求时进行拦截，以实现客户端负载均衡
2. 创建了一个RestTemplateCustomizer的Bean,用于给RestTemplate增加LoadBalancerInterceptor拦截器
3. 维护了一个被@LoadBalanced注解修饰的RestTemplate对象列表，并初始化，通过调用RestTemplateCustomizer的实例来给需要客户端负载均衡的RestTemplate增加LoadBalancerInterceptor拦截器