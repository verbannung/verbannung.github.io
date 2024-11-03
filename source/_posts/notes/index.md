---
title: springcloud学习之hystrix
date: 2024-11-03 23:34:09
tags:
    - "notes"
    - "springcloud"
    - "hystrix"
---
springcloud hystrix 使用方法和原理说明

# hystrix是什么
 Hystrix是Netflix开源的一款容错框架，包含常用的容错方法：线程池隔离、信号量隔离、熔断、降级回退。在高并发访问下，系统所依赖的服务的稳定性对系统的影响非常大，依赖有很多不可控的因素，比如网络连接变慢，资源突然繁忙，暂时不可用，服务脱机等。我们要构建稳定、可靠的分布式系统，就必须要有这样一套容错方法。

换句话说，hystrix是解决服务器压力过大或者某个服务崩溃造成雪崩效应的一个容错方案。

# hystrix的主要功能



