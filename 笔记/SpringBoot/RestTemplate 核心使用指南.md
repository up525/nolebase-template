# RestTemplate 核心使用指南

## 一、简介与同类产品对比

### 1.1 RestTemplate 是什么

- Spring 提供的**同步 HTTP 客户端**工具类
- 基于 JDK HttpURLConnection 实现（可替换底层实现）
- 支持 RESTful 风格请求
- 提供**模版方法**简化 HTTP 操作

### 1.2 同类产品对比

| **工具**          | **特点**                             |
| ----------------- | ------------------------------------ |
| RestTemplate      | 同步阻塞，学习成本低，API 简洁       |
| WebClient         | 异步非阻塞，函数式编程，Spring5+推荐 |
| OkHttp            | 高性能，支持 SPDY/HTTP2，需额外集成  |
| Apache HttpClient | 功能全面，配置复杂                   |

## 二、SpringBoot 环境配置

### 2.1 引入依赖

```xml
<!-- Spring Boot 2.x 使用 spring-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Spring Boot 3.x 需单独引入 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

### 2.2 配置Bean

```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
            .setConnectTimeout(Duration.ofSeconds(5))
            .setReadTimeout(Duration.ofSeconds(5))
            .build();
    }
}
```

## 三、核心API详解

### 3.1 GET请求

```java
// 基本GET请求（自动反序列化）
String result = restTemplate.getForObject(
    "http://api.example.com/users/{id}", 
    String.class,  // 返回类型
    1              // 路径参数
);

// 获取完整响应
ResponseEntity<User> response = restTemplate.getForEntity(
    "http://api.example.com/users/{id}",
    User.class,
    1
);
HttpStatus statusCode = response.getStatusCode();
User user = response.getBody();
```

### 3.2 POST请求

```java
// 简单POST
User newUser = new User("John", 25);
User createdUser = restTemplate.postForObject(
    "http://api.example.com/users",
    newUser,     // 请求体
    User.class   // 响应类型
);

// 带header的POST
HttpHeaders headers = new HttpHeaders();
headers.set("Authorization", "Bearer token");
HttpEntity<User> request = new HttpEntity<>(newUser, headers);
ResponseEntity<User> response = restTemplate.postForEntity(
    "http://api.example.com/users",
    request,
    User.class
);
```

### 3.3 通用请求（exchange）

```java
// 可自定义所有请求参数
ResponseEntity<User> response = restTemplate.exchange(
    "http://api.example.com/users/{id}",
    HttpMethod.PUT,       // 指定HTTP方法
    new HttpEntity<>(updatedUser, headers),  // 包含body和headers
    User.class,           // 响应类型
    1                     // 路径参数
);
```

### 3.4 异常处理

```java
try {
    restTemplate.getForObject(url, User.class);
} catch (HttpClientErrorException e) {
    // 处理4xx错误
    System.err.println("Client Error: " + e.getStatusCode());
} catch (HttpServerErrorException e) {
    // 处理5xx错误
    System.err.println("Server Error: " + e.getStatusCode());
}

// 使用自定义错误处理器
restTemplate.setErrorHandler(new DefaultResponseErrorHandler() {
    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
        // 自定义错误处理逻辑
    }
});
```

## 四、重点总结

1. **配置要点**：
   - Spring Boot 2.x 自动配置，3.x 需手动引入
   - 推荐通过 `RestTemplateBuilder` 配置超时时间
2. **核心方法**：
   - `getForObject()`：快速获取响应体
   - `exchange()`：最灵活的请求方式
   - `postForEntity()`：需要处理响应头时使用
3. **最佳实践**：
   - 总是配置合理的超时时间
   - 推荐使用URI模板参数代替字符串拼接
   - 处理HTTP状态码异常
4. **适用场景**：
   - 快速实现同步HTTP调用
   - 简单REST接口调用
   - 需要与旧Spring项目整合
5. **注意事项**：
   - 已进入维护模式（推荐WebClient用于新项目）
   - 同步阻塞特性不适合高并发场景

> 提示：本文档基于 Spring Boot 2.7.x 版本编写，若使用 Spring Boot 3.x 请注意以下变化：
>
> 1. 需要显式引入 `spring-boot-starter-webflux`
> 2. JDK 最低要求 17
> 3. 部分API返回值类型可能有变化

