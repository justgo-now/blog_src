---
title: SpringBoot之Starter组件概览
date: 2018-12-16 18:29:34
category:
- 后端开发
tags:
- Spring
- SpringBoot
---

# 应用类 Starter


| 名称 | 描述 | POM |
| ---------- | -------------------- | ----- |
| `spring-boot-starter`                         | 核心 starter，包含自动配置支持、日志和 YAML                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
| `spring-boot-starter-activemq`                | 提供 JMS 消息支持，使用 Apache ActiveMQ                      | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
| `spring-boot-starter-amqp`                    | 提供 Spring AMQP 与 Rabbit MQ 支持                           | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
| `spring-boot-starter-aop`                     | 提供 Spring AOP 与 AspectJ 面向切面编程支持                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml) |
| `spring-boot-starter-artemis`                 | 提供 JMS 消息服务支持，使用 Apache Artemis                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml) |
| `spring-boot-starter-batch`                   | 提供 Spring Batch 支持                                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml) |
| `spring-boot-starter-cache`                   | 提供 Spring Framework 缓存支持                               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml) |
| `spring-boot-starter-cloud-connectors`        | 使用 Spring Cloud Connectors 简单连接到类似 Cloud Foundry 和 Heroku 等云平台 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml) |
| `spring-boot-starter-data-cassandra`          | 提供对 Cassandra 分布式数据库和 Spring Data Cassandra 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml) |
| `spring-boot-starter-data-cassandra-reactive` | 提供对 Cassandra 分布式数据库和 Spring Data Cassandra Reactive 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml) |
| `spring-boot-starter-data-couchbase`          | 提供对 Couchbase 面向文档数据库和 Spring Data Couchbase 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml) |
| `spring-boot-starter-data-couchbase-reactive` | 提供对 Couchbase 面向文档数据库和 Spring Data Couchbase Reactive 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase-reactive/pom.xml) |
| `spring-boot-starter-data-elasticsearch`      | 提供对 Elasticseach 搜索与分析引擎和 Spring Data Elasticsearch 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml) |
| `spring-boot-starter-data-jpa`                | 提供 Spring Data JPA 与 Hibernate 的支持                     | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
| `spring-boot-starter-data-ldap`               | 提供对 Spring Data LDAP 的支持                               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml) |
| `spring-boot-starter-data-mongodb`            | 提供对 MongoDB 面向文档数据库和 Spring Data MongoDB 的支持   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml) |
| `spring-boot-starter-data-mongodb-reactive`   | 提供对 MongoDB 面向文档数据库和 Spring Data MongoDB Reactive 的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml) |
| `spring-boot-starter-data-neo4j`              | 提供对 Neo4j 图数据库与 SPring Data Neo4j 的支持             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml) |
| `spring-boot-starter-data-redis`              | 提供对 Redis 键值数据存储、Spring Data Redis 和 Lettuce 客户端的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml) |
| `spring-boot-starter-data-redis-reactive`     | 提供对 Redis 键值数据存储、Spring Data Redis Reactive 和 Lettuce 客户端的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml) |
| `spring-boot-starter-data-rest`               | 提供使用 Spring Data REST 通过 REST 暴露 Spring Data 资源库的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml) |
| `spring-boot-starter-data-solr`               | 提供对 Apache Solr 搜索平台与 Spring Data Solr 的支持        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml) |
| `spring-boot-starter-freemarker`              | 提供使用 Freemakrer 视图构建 MVC web 应用的支持              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml) |
| `spring-boot-starter-groovy-templates`        | 提供使用 Groovy 模板视图构建 MVC web 应用的支持              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml) |
| `spring-boot-starter-hateoas`                 | 提供使用 Spring MVC 与Spring HATEOAS 构建基于超媒体的 RESTful web 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml) |
| `spring-boot-starter-integration`             | 提供对 Spring Integration 的支持                             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml) |
| `spring-boot-starter-jdbc`                    | 提供 JDBC 与 Tomcat JDBC 连接池的支持                        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml) |
| `spring-boot-starter-jersey`                  | 提供对使用 JAX-RS 与 Jersey 构建 RESTful web 应用的支持。[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-web) 的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml) |
| `spring-boot-starter-jooq`                    | 提供对使用 JOOQ 访问 SQL 数据库的支持。[`spring-boot-starter-data-jpa`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-data-jpa) 或 [`spring-boot-starter-jdbc`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-jdbc) 的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml) |
| `spring-boot-starter-json`                    | 提供了读写 json 的支持                                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml) |
| `spring-boot-starter-jta-atomikos`            | 提供 Atomikos JTA 事务支持                                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml) |
| `spring-boot-starter-jta-bitronix`            | 提供 Bitronix JTA 事务支持                                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml) |
| `spring-boot-starter-jta-narayana`            | 提供 Narayana JTA 支持                                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-narayana/pom.xml) |
| `spring-boot-starter-mail`                    | 提供使用　Java Mail 与 Spring Framework 的邮件发送支持       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml) |
| `spring-boot-starter-mustache`                | 提供使用 Mustache 视图构建 web 应用的支持                    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml) |
| `spring-boot-starter-quartz`                  | Quartz 支持                                                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml) |
| `spring-boot-starter-security`                | Spring Security 支持                                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml) |
| `spring-boot-starter-test`                    | 提供包含了 JUnit、Hamcrest 与 Mockito 类库的 Spring Boot 单元测试支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml) |
| `spring-boot-starter-thymeleaf`               | 提供使用 Thymeleaf 视图构建 MVC web 应用的支持               | [pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
| `spring-boot-starter-validation`              | 提供 Hibernate Validator 与 Java Bean Validation 的支持      | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
| `spring-boot-starter-web`                     | 提供使用 Spring MVC 构建 web（包含 RESTful）应用的支持，使用 Tomcat 作为默认嵌入式容器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml) |
| `spring-boot-starter-web-services`            | Spring Web Services 支持                                     | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml) |
| `spring-boot-starter-webflux`                 | 提供使用 Spring Framework 的 Reactive Web 支持构建 WebFlux 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml) |
| `spring-boot-starter-websocket`               | 提供使用 Spring Framework 的 WebSocket 支持构建 WebSocket 应用的支持 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |


# 生产类 starter


| 名称 | 描述 | POM |
| ---------- | -------------------- | ----- |
| `spring-boot-starter-actuator` | Spring Boot 的 Actuator 支持，其提供了生产就绪功能，帮助您监控和管理应用 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml) |


# 技术类 starter


| 名称 | 描述 | POM |
| ---------- | -------------------- | ----- |
| `spring-boot-starter-jetty`         | 使用 Jetty 作为嵌入式 servlet 容器。可代替 [`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml) |
| `spring-boot-starter-log4j2`        | 使用 Log4j2 作为日志组件。可代替 [`spring-boot-starter-logging`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-logging) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml) |
| `spring-boot-starter-logging`       | 使用 Logback 作为日志组件，此 starter 为默认的日志 starter   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml) |
| `spring-boot-starter-reactor-netty` | 使用 Reactor Netty 作为内嵌响应式 HTTP 服务器                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml) |
| `spring-boot-starter-tomcat`        | 使用 Tomcat 作为嵌入式 servlet 容器，此为 [`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-web) 默认的 servlet 容器 starter | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml) |
| `spring-boot-starter-undertow`      | 使用 Undertow 作为嵌入式 servlet 容器，可代替 [`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.0.0.RELEASE/reference/htmlsingle/#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.0.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml) |



**提示**

> 有关其它社区贡献的 starter 列表，请参阅 GitHub 上的 `spring-boot-starters` 模块中的 [README 文件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc)。


# 参考文献

[SpringBoot官方文档](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#using-boot-starter)  
[SpringBoot官方文档中文翻译](https://docshome.gitbooks.io/springboot/content/pages/using-spring-boot.html#using-boot-starter)  
[SpringBoot组件说明](https://blog.csdn.net/cxd275050943/article/details/74452102)  