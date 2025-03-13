# Spring Cloud Gateway 笔记

## 简介

Spring Cloud Gateway 是基于 Spring 5、Spring Boot 2 和 Project Reactor 的 API 网关，提供动态路由、安全、监控和弹性等功能。  
**核心特性**：异步非阻塞模型、高性能、支持动态配置、丰富的断言（Predicate）和过滤器（Filter）。

官网：https://spring.io/projects/spring-cloud-gateway

### 与其他网关对比

| 产品                     | 特点                                                         |
| ------------------------ | ------------------------------------------------------------ |
| **Zuul 1.x**             | 基于 Servlet 2.5，阻塞式 I/O，性能较低，社区已逐步淘汰       |
| **Zuul 2.x**             | 支持非阻塞，但生态不完善                                     |
| **Kong**                 | 基于 Nginx + OpenResty，适合复杂场景，但依赖数据库，配置复杂 |
| **Spring Cloud Gateway** | 轻量级、无缝集成 Spring 生态，性能优异，适合微服务场景       |

---

## 1 基础入门

### 1.1 功能

![image.png](https://i-blog.csdnimg.cn/img_convert/3329998b6ecab760b8b6842f099b3117.png)

- 动态路由：根据请求路径、Header 等条件路由到不同服务。
- 请求过滤：修改请求/响应内容（如添加 Header、限流）。
- 负载均衡：集成 Ribbon 实现服务负载均衡。

---

### 1.2 HelloWorld

> `/api/order/**`路由给订单
>
> `/api/product/**`路由给商品
>
> 测试负载均衡

#### 1.2.1 创建项目

1. - 引入 `spring-cloud-starter-gateway`、
   - 引入`spring-cloud-starter-alibaba-nacos-discovery`

   ```xml
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-gateway</artifactId>
   </dependency>
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

#### 1.2.2 改造微服务

确保目标微服务已注册到注册中心（如 Nacos/Eureka），并暴露接口（如 `/api/user/{id}`）。

- 为 service-order、service-prduct 添加 `/api`基础路径

#### 1.2.3 配置网关

在 `application.yml` 中配置路由规则：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: order
        uri: lb://service-order  # lb表示负载均衡
        predicates:
         - Path=/api/order/**   # 路径匹配
      - id: product
        uri: lb://service-product
        predicates:
         - Path=/api/product/**
        filters:
            - StripPrefix=1      # 去掉路径前缀（/api/user -> /user）           
```

------

### 1.3 原理

![image.png](https://i-blog.csdnimg.cn/img_convert/3eb23c299e5ee395e16dbeeaf562bfb8.png)

- **核心组件**：`Route`（路由规则）、`Predicate`（匹配条件）、`Filter`（处理逻辑）。
- **执行流程**：客户端请求 → 匹配 Predicate → 执行 Filter 链 → 转发到目标服务。

------

## 2 Predicate - 断言

```yml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - After=2017-01-20T17:42:47.789-07:00[America/Denver]
```

断言用于定义请求匹配条件，常用类型：

| 断言类型   | 示例配置                      | 说明           |
| :--------- | :---------------------------- | :------------- |
| **Path**   | `- Path=/order/**`            | 路径匹配       |
| **Method** | `- Method=GET,POST`           | HTTP 方法匹配  |
| **Header** | `- Header=X-Request-Id, \\d+` | 请求头正则匹配 |
| **Query**  | `- Query=name, zhangsan`      | 请求参数匹配   |

------

## 3 Filter - 过滤器

过滤器用于修改请求/响应，分为 `GatewayFilter`（单路由）和 `GlobalFilter`（全局）。

### 常用内置过滤器

```yaml
filters:
  - AddRequestHeader=X-Request-Color, blue  # 添加请求头
  - RewritePath=/api/(?<segment>.*), /$\{segment}  # 重写路径
  - Retry=3  # 失败重试3次
```

------

## 4 CORS - 跨域处理

在 `application.yml` 中全局配置跨域：

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "https://docs.spring.io"
            allowedMethods:
            - GET
```

局部跨域::

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cors_route
        uri: https://example.org
        predicates:
        - Path=/service/**
        metadata:
          cors:
            allowedOrigins: '*'
            allowedMethods:
              - GET
              - POST
            allowedHeaders: '*'
            maxAge: 30
```

------

## 5 GlobalFilter

自定义全局过滤器：

```java
@Bean
public GlobalFilter customFilter() {
    return new CustomGlobalFilter();
}

public class CustomGlobalFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("custom global filter");
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

------

## 总结

### 核心要点

1. **路由配置**：通过 `Predicate` 定义匹配条件，`Filter` 定义处理逻辑。
2. **断言类型**：掌握 `Path`、`Header`、`Query` 等常用断言。
3. **过滤器**：内置过滤器快速实现功能，`GlobalFilter` 自定义全局逻辑。
4. **跨域配置**：通过 YAML 或代码全局解决跨域问题。
5. **性能优势**：基于 WebFlux 的异步非阻塞模型，适合高并发场景。

### 代码复用技巧

- 将通用路由配置抽象为 `yml` 片段，方便多环境复用。
- 自定义 `GlobalFilter` 封装日志等公共逻辑。