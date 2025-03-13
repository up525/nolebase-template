# OpenFeign 学习笔记

## 一、基础入门

### 1.1 简介

- OpenFeign 是基于声明式的 REST 客户端，用于简化服务间远程调用。（编程式 REST 客户端（RestTemplate））

- 通过接口+注解方式定义 HTTP 请求，自动实现服务调用。

   注解驱动 

  ​	  • 指定远程地址：@FeignClient

  ​	  • 指定请求方式：@GetMapping、@PostMapping、@DeleteMapping ...

   	 • 指定携带数据：@RequestHeader、@RequestParam、@RequestBody ... 

  ​	  • 指定结果返回：响应模型

- 官网：https://docs.spring.io/spring-cloud-openfeign/reference/spring-cloud-openfeign.html#spring-cloud-feign

- 人话总结：OpenFeign是一种替代RestTemplate的工具，专门用来实现不同微服务之间实现远程调用的业务API，相比RestTemplate功能更强大，操作更简介。

![](https://i-blog.csdnimg.cn/direct/2c65e85d314946b99fcd14819801616c.png)


### 1.2 引入依赖

```xml
<!-- Spring Cloud OpenFeign 核心依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

### 1.3 开启功能

在主启动类添加注解：

```java
@EnableFeignClients // 启用 OpenFeign 客户端功能
@SpringBootApplication
public class Application { ... }
```

### 1.4 远程调用

1. **定义 Feign 客户端接口**：（客户端：发送请求 服务端：接收请求）

   1.1远程调用 - 业务API

```java
@FeignClient(value = "product-service")//声明要调用的Feign客户端，名称是application.yml中配置的名称,底层会自动去注册中心负载均衡的发送请求
public interface ProductFeignClient {//使用feign实现的专门向product服务进行调用的接口
    //mvc注解的两套逻辑
    //1.标注在Controller上，是接收这一样的请求
    //2.标注在FeignClient上，是发送这样的请求
    @GetMapping("/product/{id}")
    public Product getProductById(@PathVariable("id") Long productId);
}
//接收请求的时候是把这个id放到productId
//发送请求的时候是把productId放到id上
```

​		1.2远程调用 - 第三方API
![](https://i-blog.csdnimg.cn/direct/8998406bf209448baa22ca71342ca785.png)


tip：如何编写好OpenFeign声明式的远程调用接口？

• 业务API：直接复制对方Controller签名即可 
```java
    @GetMapping("/product/{id}")
    public Product getProductById(@PathVariable("id") Long productId);
```

• 第三方API：根据接口文档确定请求如何发

​	2.**注入使用**：

```java
@Autowired
private UserClient userClient;
public User getUser(Long id) {
    return userClient.getUserById(id); // 直接调用远程接口
}
```

### 1.5面试题：客户端负载均衡与服务端负载均衡区别？

答：根据负载均衡发生的位置来区分。

负载均衡发生在客户端就是客户端负载均衡。

负载均衡发生在服务端就是服务端负载均衡。
![](https://i-blog.csdnimg.cn/direct/879bd948469f4b82a8c212df8719eae3.png)


------

## 二、进阶配置

### 2.1 开启日志

**配置日志级别**（application.yml）：

```yaml
# 客户端
logging:
  level:
    com.atguigu.order.Feign: DEBUG # 指定客户端接口的日志级别，直接复制包名，这样其下的所有客户端就都会开启日志功能
```

**配置日志策略**（Java Config）：

```java
@Configuration
//注意一定是feign包下的Logger
import feign.Logger;

//OrderConfig
@Configuration
public class OrderConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL; // FULL/BASIC/HEADERS/NONE
    }
}
```

### 2.2 超时控制（避免服务器宕机）
商品服务卡慢导致订单服务卡慢，导致。。。链式效应进而导致整个服务的卡慢，（服务雪崩）因此我们需要一个超时控制机制。
![](https://i-blog.csdnimg.cn/direct/08c80cd5b29b46688869d7a1dc979379.png)
 connectTimeout： 打电话嘟嘟嘟，没人接（电话没接通）

​		readTimeout:   喂喂喂，没人回（电话接通了）
**openfeign：默认配置**
![](https://i-blog.csdnimg.cn/direct/1a70a1a608b34fec87b70782a19840cc.png)


```yaml
# application.yml
spring:
  cloud:
    openfeign:
      client:
        config:
          default:
            logger-level: full
            connect-timeout: 1000 # ms
            read-timeout: 2000
          product-service:
            logger-level: full
            connect-timeout: 3000
            read-timeout: 5000
```

### 2.3 重试机制

远程调用超时失败后，还可以进行多次尝试，如果某次成功返回ok，如 果多次依然失败则结束调用，返回错误。(一次调用失败我不甘心，我想多试几次。)

![](https://i-blog.csdnimg.cn/direct/75cf36d921d146609df3ebe41839f214.png)


```yaml
# application.yml
spring:
  cloud:
    openfeign:
      client:
        config:
          default:
            retryable: true # 启用重试
            maxAttempts: 3 # 最大重试次数
```

或

```java
@Bean
Retryer retryer(){
    return new Retryer.Default();
}
//spring会自动把retryer注入到容器，注入之后底层会默认这个配置（源码）
public Default() {
    this(100L, TimeUnit.SECONDS.toMillis(1L), 5);//请求超时默认发5次，最多1s
}
```

### 2.4 Fallback 兜底返回

1.引入 sentinel 

```xml
<dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
 </dependency>
```

2.开启熔断

```yaml
feign:
  sentinel:
    enabled: true
```

3.编写 fallback 函数

```java
@FeignClient(value = "service-product",fallback = ProductFeignClientFallback.class) // feign客户端
public interface ProductFeignClient {
   //mvc注解的两套使用逻辑
   //1、标注在Controller上，是接收这样的请求
   //2、标注在FeignClient上，是发送这样的请求
   @GetMapping("/product/{id}")
   Product getProductById(@PathVariable("id") Long id);
}
```

```java
// Feign包下创建一个FallBack包（兜底回调）
// FallBack包在创建ProductFeignClientFallback，并且继承ProductFeignClient(接口)
@Component//只有把这个兜底回调放在容器中才能实现兜底回调
public class ProductFeignClientFallback implements ProductFeignClient {
	//这个函数不一定会回调，只有出现超时情况才会调用这个函数
    public Product getProductById(Long id) {
        System.out.println("兜底回调....");
        Product product = new Product();
        product.setId(id);
        product.setPrice(new BigDecimal("0"));
        product.setProductName("未知商品");
        product.setNum(0);
        return product;
    }
}
```

------

## 三、拦截器用法
![](https://i-blog.csdnimg.cn/direct/384c973e57f4416bb7c9c2b47ec9187e.png)


1.请求拦截器

2.响应拦截器（用的不多）

**自定义请求拦截器**：

```java
import feign.RequestInterceptor;

@Component//如果放入IOC容器就会自动注册拦截器，可以不做后面的注册
public class XTokenRequestInterceptor implements RequestInterceptor {
    /**
    *请求拦截器
    *template 请求模板
    */
    @Override
    public void apply(RequestTemplate template) {
        System.out.println("XTokenRequestInterceptor.......");
        template.header("X-Token", "UUID.randomUUID.toString()"); // 添加请求头，用作身份验证
    }
}
```

**注册拦截器**（配置类中）：

```java
@Configuration
public class FeignConfig {
    @Bean
    public AuthInterceptor authInterceptor() {
        return new AuthInterceptor();
    }
}
```

------

## 四、重点总结

| 核心内容           | 关键点                                                       |
| :----------------- | :----------------------------------------------------------- |
| **远程调用客户端** | 通过 `@FeignClient` 定义接口，使用 `@GetMapping` 等注解声明 HTTP 方法。 |
| **超时控制**       | 配置 `connectTimeout` 和 `readTimeout`，支持全局和按服务配置。 |
| **重试机制**       | 启用 `retryable` 并设置 `maxAttempts`，增强服务调用容错性。  |
| **Fallback 兜底**  | 实现 Fallback 类处理服务降级，防止级联故障。                 |
| **拦截器**         | 通过 `RequestInterceptor` 添加统一请求头或认证信息。         |

**注意事项**：

1. 生产环境建议配置合理的超时时间和重试策略，避免雪崩效应。
2. 拦截器可用于统一认证、日志跟踪等场景。