---
title: SpringCloud组件概览
date: 2019-03-25 20:10:01
category: 后端开发
tags:
- Spring
- SpringCloud
layout: post
published: true
---

| 组件                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Spring Cloud Config](https://springcloud.cc/spring-cloud-config.html) | 配置管理工具包，让你可以把配置放到远程服务器，集中化管理集群配置，目前支持本地存储、Git以及Subversion。 |
| [Spring Cloud Bus](https://springcloud.cc/spring-cloud-bus.html) | 事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署。 |
| [Eureka](https://github.com/Netflix/eureka)                  | 云端服务发现，一个基于 REST 的服务，用于定位服务，以实现云端中间层服务发现和故障转移。 |
| [Hystrix](https://github.com/Netflix/hystrix)                | 熔断器，容错管理工具，旨在通过熔断机制控制服务和第三方库的节点,从而对延迟和故障提供更强大的容错能力。 |
| [Zuul](https://github.com/Netflix/zuul)                      | Zuul 是在云平台上提供动态路由,监控,弹性,安全等边缘服务的框架。Zuul 相当于是设备和 Netflix 流应用的 Web 网站后端所有请求的前门。 |
| [Archaius](https://github.com/Netflix/archaius)              | 配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能。 |
| [Consul](https://github.com/HashiCorp/consul)                | 封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成。 |
| [Spring Cloud for Cloud Foundry](https://github.com/spring-cloud/spring-cloud-cloudfoundry) | 通过Oauth2协议绑定服务到CloudFoundry，CloudFoundry是VMware推出的开源PaaS云平台。它支持多种框架、语言、运行时环境、云平台及应用服务，使开发人员能够在几秒钟内进行应用程序的部署和扩展，无需担心任何基础架构的问题 |
| [Spring Cloud Sleuth](https://github.com/spring-cloud/spring-cloud-sleuth) | 日志收集工具包，封装了Dapper和log-based追踪以及Zipkin和HTrace操作，为SpringCloud应用实现了一种分布式追踪解决方案。 |
| [Spring Cloud Data Flow](https://springcloud.cc/spring-cloud-dataflow.html) | 大数据操作工具，作为Spring XD的替代产品，它是一个混合计算模型，结合了流数据与批量数据的处理方式。 |
| [Spring Cloud Security](https://github.com/spring-cloud/spring-cloud-security) | 基于spring security的安全工具包，为你的应用程序添加安全控制。 |
| [Spring Cloud Zookeeper](https://github.com/spring-cloud/spring-cloud-zookeeper) | 操作Zookeeper的工具包，用于使用zookeeper方式的服务发现和配置管理。 |
| [Spring Cloud Stream](https://github.com/spring-cloud/spring-cloud-stream) | 数据流操作开发包，封装了与Redis,Rabbit、Kafka等发送接收消息。 |
| [Spring Cloud CLI](https://springcloud.cc/spring-cloud-cli.html) | 基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件。   |
| [Ribbon](https://github.com/Netflix/ribbon)                  | 提供云端负载均衡，有多种负载均衡策略可供选择，可配合服务发现和断路器使用。 |
| [Turbine](https://github.com/Netflix/turbine)                | Turbine是聚合服务器发送事件流数据的一个工具，用来监控集群下hystrix的metrics情况。 |
| [Feign](https://github.com/OpenFeign/feign)                  | Feign是一种声明式、模板化的HTTP客户端。                      |
| [Spring Cloud Task](https://github.com/spring-cloud/spring-cloud-task) | 提供云端计划任务管理、任务调度。                             |
| [Spring Cloud Connectors](https://springcloud.cc/spring-cloud-connectors.html) | 便于云端应用程序在各种PaaS平台连接到后端，如：数据库和消息代理服务。 |
| [Spring Cloud Cluster](https://github.com/spring-cloud/spring-cloud-cluster) | 提供Leadership选举，如：Zookeeper, Redis, Hazelcast, Consul,选举、集群的状态一致性、全局锁、tokens等常见状态模式的抽象和实现。提供在分布式系统中的集群所需要的基础功能支持， |
| [Spring Cloud Starters](https://github.com/spring-cloud/spring-cloud-stream-starters) | Spring Boot式的启动项目，为Spring Cloud提供开箱即用的依赖管理。 |
| [Turbine](https://github.com/Netflix/Turbine)                | Turbine是聚合服务器发送事件流数据的一个工具，用来监控集群下hystrix的metrics情况。 |
