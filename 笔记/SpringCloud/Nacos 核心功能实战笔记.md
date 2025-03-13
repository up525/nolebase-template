# Nacos 核心功能实战笔记

## 一、Nacos 简介

### 1. 是什么？

- **全称**：Nacos = **Na**ming and **Co**nfiguration **S**ervice  
- **定位**：阿里巴巴开源的 **动态服务发现、配置管理、服务管理平台**  
- **核心功能**：服务注册与发现 + 统一配置管理 + 服务健康监测  
- **适用场景**：微服务架构、云原生应用（如 Spring Cloud、Dubbo）  

### 2. 同类产品对比

| 产品          | 主要功能                       | 优势                                  | 局限性                       | 典型场景                     |
| ------------- | ------------------------------ | ------------------------------------- | ---------------------------- | ---------------------------- |
| **Nacos**     | 服务发现 + 配置管理            | 功能全面，支持动态配置、AP/CP模式切换 | 学习成本略高                 | 需要统一管理服务与配置的场景 |
| **Eureka**    | 服务发现                       | 简单轻量，AP模型保证高可用            | 无配置管理功能，已停止维护   | 纯服务发现场景               |
| **Consul**    | 服务发现 + 配置管理 + 健康检查 | 支持多数据中心，安全性强              | 配置管理功能较弱             | 复杂分布式系统               |
| **Zookeeper** | 服务发现 + 分布式协调          | 强一致性（CP模型）                    | 配置管理需自行实现，运维复杂 | 强一致性要求的场景           |

---

## 二、Nacos 核心功能详解

### 1. 服务注册与发现

- **工作原理**：  

  1. 服务启动时向Nacos注册自己的元数据（IP、端口、健康状态）  
  2. 消费者通过Nacos查询可用服务实例列表  

3. 内置心跳机制自动剔除故障节点  

- **示例场景**：  

  ```java
  // 订单服务注册到Nacos
  @SpringBootApplication
  @EnableDiscoveryClient  // 关键注解
  public class OrderServiceApplication { ... }
  
  // 用户服务调用订单服务
  @Autowired
  private RestTemplate restTemplate;
  
  public String getOrder() {
    // 直接使用服务名调用（需配合@LoadBalanced）
    return restTemplate.getForObject("http://order-service/orders", String.class);
  }
  ```

### 2. 动态配置管理

- **核心特性**：
  - **配置热更新**：修改Nacos控制台配置 → 应用实时生效（无需重启）
  - **多环境支持**：通过 **Namespace** 隔离开发/测试/生产环境
  - **灰度发布**：支持按IP或分组推送特定配置

------

### 3. 服务健康监测

- **机制**：

  - 客户端每5秒发送心跳包到Nacos服务器
  - 超过15秒未收到心跳标记为"不健康"
  - 30秒未恢复则从服务列表移除

- **扩展能力**：

  ```java
  // 自定义健康检查规则
  @Component
  public class CustomHealthChecker implements HealthIndicator {
    @Override
    public Health health() {
      // 检查数据库连接等自定义逻辑
      return Health.up().withDetail("db", "connected").build();
    }
  }
  ```

------

## 三、为什么选择 Nacos？

### 1. 核心优势

- **一站式解决方案**：同时解决服务发现与配置管理问题
- **灵活性**：支持AP（高可用）和CP（强一致）模式动态切换
- **生态整合**：无缝对接Spring Cloud、Dubbo、K8s
- **易用性**：提供可视化控制台，降低运维复杂度

### 2. 典型应用场景

1. **微服务架构**：统一管理上百个服务的注册与配置
2. **配置灰度发布**：只对部分服务实例推送新配置
3. **多环境管理**：通过Namespace快速切换开发/测试/生产环境
4. **服务熔断**：结合Sentinel实现流量控制

------

## 附：快速命令备忘

```bash
# 启动Nacos服务器（单机模式）
startup.cmd -m standalone
```

## 四、注册中心实战

### 远程调用 - 基本流程
![](https://i-blog.csdnimg.cn/direct/514654c9c35a42939ad79198fcd03006.png)



### 1. 基础配置

**步骤：**

1. **添加依赖**  

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

2.**配置Nacos地址**

![](https://i-blog.csdnimg.cn/direct/f0729d1bf49b41c390599b3f29551538.png)


```properties
# Nacos服务器中显示的名称
spring.application.name=service-order
# 该模块启动占用的端口
server.port=8000
```

```yaml
# application.yml
server:
  port: 8000
spring:
  application:
    name: order-service

spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848  # Nacos服务地址
```

3.**开启服务发现**
在启动类添加注解：

```java
@SpringBootApplication
@EnableDiscoveryClient  // 开启服务注册与发现
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```

查看注册中心效果 访问 http://localhost:8848/nacos  

集群模式启动测试 单机情况下通过改变端口模拟微服务集群

------

### 2. 扩展功能

**1. 获取服务实例列表**
通过 `DiscoveryClient` 查询服务实例：（Spring提供的，NacosDiscoveryClient是DiscoveryClient的实现）

```java
@Autowired
private DiscoveryClient discoveryClient;

public void listServices() {
    List<String> services = discoveryClient.getServices();  // 获取所有服务名
    List<ServiceInstance> instances = discoveryClient.getInstances("order-service");  // 获取指定服务的实例
}
```

**2. 负载均衡调用**
引入依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

使用 `LoadBalancerClient` 选择实例：

```java
//负载均衡基础版
@Configuration
public class OrderConfig {
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

    //负载均衡发送请求
    public Product getProductFromRemoteWithLoadBalancer(Long productId){
        //1.获取服务实例
        ServiceInstance choose = loadBalancerClient.choose("service-product");
        //2.获取服务实例的uri
        String url = "http://"+choose.getHost()+":"+choose.getPort()+"/product/"+productId;
        log.info("请求地址：{}",url);
        //3.发送请求
        Product product = restTemplate.getForObject(url, Product.class);
        return product;
    }
```

```java
//进阶基于注解的负载均衡
@Configuration
public class OrderConfig {

    @LoadBalanced//使用注解，自动实现负载均衡
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
private Product getProductFromRemqteWithLoadBalanceAnnotation(Long productId) {
    String url="http://service-product/product/"+productId;
	//给远程发送请求；service-product会被动态替换
	Product product = restTemplate.getForObject(url,Product.class);
	return product;
}
```



3.**远程调用 - 面试题**

思考：注册中心宕机，远程调用还能成功吗？
![](https://i-blog.csdnimg.cn/direct/7d30abf1ae1d4ef299dca452277b8de0.png)



1.调用过：远程调用虽然不在依赖中心，但是可以通过。

2.没调用过：相当于第一次发起远程调用，不能通过。

------

## 五、配置中心实战

### 1. 配置中心 - 基本使用

**步骤：**

1. **添加依赖**

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

​	2.**配置Nacos地址**

```yaml
spring.application.name=service-product # 在Nacos中展示的模块名称
server.port=9000 # 项目启动的默认端口号

spring.cloud.nacos.server-addr=127.0.0.1:8848 # Nacos占用的端口号
spring.config.import=nacos:service-order.properties  # 导入Nacos中的配置
```

补充：

```properties
# nacos禁用导入检查
spring.nacos.config.import-check.enabled=false  
```

2.1**动态读取配置**
使用 `@Value` + `@RefreshScope` 实现动态刷新(在Nacos中更新，就会立即在项目中更新)：

```java
@RestController
@RefreshScope  // 支持配置热更新
public class UserController {
    @Value("${app.maxRetry:3}")  // 默认值3
    private int maxRetry;
}
```

​	2.2**批量绑定配置**
使用 `@ConfigurationProperties`：

```java
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@ConfigurationProperties(prefix = "order")//配置批量绑定在nacos下，可以无需@RefreshScope注解就能实现自动刷新
@Component
@Data
public class OrderProperties {
    String timeout;
    String autoConfirm;
}
```

> 优点：
>
> ​	1.代码更简洁，使用的时候直接注入即可。
>
> ​	2.使用**批量绑定配置**，spring的底层会自动实现动态刷新，不用再加`@RefreshScope`注解。

3.**在Nacos中配置数据集：** 

例如：service-order.properties

```properties
# Nacos中的配置service-order.properties
order.timeout=10min
order.auto-confirm=7d
```



 4.**如何在NacosConfigManager 监听配置变化？**

​	1、项目启动就监听配置文件变化
​	2、发生变化后拿到变化值
​	3、发送邮件

```java
@EnableDiscoveryClient//开启服务发现功能
@SpringBootApplication
public class OrderMainApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderMainApplication.class, args);
        }
        //1、项目启动就监听配置文件变化
        //2、发生变化后拿到变化值
        //3、发送邮件
        @Bean
        ApplicationRunner applicationRunner(NacosConfigManager nacosConfigManager){
            return (ApplicationArguments args )-> {
                ConfigService configService = nacosConfigManager.getConfigService();
                configService.addListener("service-order.properties", "DEFAULT_GROUP", new Listener() {
                    @Override
                    public void receiveConfigInfo(String configInfo) {
                        System.out.println("配置文件发生变化："+configInfo);
                        System.out.println("邮件通知");
                    }

                    @Override
                    public Executor getExecutor() {
                        return Executors.newFixedThreadPool(4);
                    }
                });
                System.out.println("启动成功");
            };
     }
}
```



5.**思考：** Nacos中的数据集和application.properties 有相同的配置项，哪个生效？

答：Nacos中的数据集生效，因为在spring中导入优先级遵循：先导入优先，外部优先。

------

### 2. 配置中心 -  数据隔离

**需求描述**

​	 • 项目有多套环境：dev，test，prod 

​	 • 每个微服务，同一种配置，在每套环境的值都不一样。

 		• 如：database.properties 

​	 	• 如：common.properties

​	 • 项目可以通过切换环境，加载本环境的配置

​	 • **难点 **

​		• 区分多套环境 

​		• 区分多种微服务

​		• 区分多种配置 

​		• 按需加载配置



**1.在Nacos中 配置优先级**

- **Namespace**：区分环境（如 dev/test/prod）
- **Group**：区分微服务（如product-service/order-service）
- **Data ID**：具体配置文件（如 commom.properties）

**示例：**

![](https://i-blog.csdnimg.cn/direct/afe3b2333d5d4c669a317780fcce2102.png)
![](https://i-blog.csdnimg.cn/direct/6bd0ecf727a249228af4e781b471f71c.png)
![](https://i-blog.csdnimg.cn/direct/724032b1e8934c6fb4624d4171f1fd2a.png)






2. **在Nacos中创建配置**

![](https://i-blog.csdnimg.cn/direct/aace86fe46ae46eab0eed77a82d6b45f.png)
![](https://i-blog.csdnimg.cn/direct/d7f7dbe65879476fac68c20c6f40f7c0.png)




```yaml
# 例在Nacos中创建配置：
Data ID: commom.properties
Group: order  
Namespace: dev
配置内容：order.timeout=1min
		order.auto-confirm=1h
```

3.在项目中按需加载配置

```java
spring:
  application:
    name: order-service
  cloud:
    nacos:
      server-addr: 127.0.0.1:8848
      config:
        namespace: dev
  config:
    import:
      - nacos:common.properties?group=order
```

## 三、总结

| 功能         | 核心步骤                                                     | 扩展能力                                                     |
| :----------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **注册中心** | 1. 引入依赖 2. 配置地址 3. 启用 `@EnableDiscoveryClient`     | 1. 服务实例查询（`DiscoveryClient`） 2. 负载均衡调用（`LoadBalancerClient`+`RestTemplate`） |
| **配置中心** | 1. 引入依赖 2. 配置地址+导入数据集 3. 使用 `@Value`+`@RefreshScope`实现绑定属性以及动态刷新 | 1. 多环境隔离（Namespace/Group/Data ID） 2. 批量绑定（`@ConfigurationProperties`） |

