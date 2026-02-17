---
title: 第13章 发现服务
date: 2026-02-17 15:10:50
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章介绍 **Spring Cloud Netflix Eureka**，实现微服务架构中的服务发现机制。

---

## 13.1 微服务思维

### 单体架构 vs 微服务架构

- **单体架构（Monolith）**：整个应用作为一个可部署单元运行
    - 开发初期简单，但随规模增长变得笨重
    - 任何小改动都需要重新构建和部署整个应用
    - 难以水平扩展特定功能，只能整体扩展
- **微服务架构（Microservices）**：将应用拆分为多个小型、独立部署的服务
    - 每个服务专注于一个业务能力
    - 可独立开发、部署和扩展
    - 服务之间通过网络通信（通常是 HTTP / REST）

### 微服务的核心优势

- **独立部署**：修改一个服务不影响其他服务
- **独立扩展**：可针对高负载服务单独扩容
- **技术多样性**：不同服务可使用不同技术栈
- **故障隔离**：一个服务故障不会拖垮整个系统

### 微服务带来的挑战

- 服务实例的网络位置（host + port）是动态变化的
- 硬编码 URL 不现实 → 需要 **服务发现（Service Discovery）** 机制
- 服务发现的核心思想：服务启动时向注册中心注册自己，消费者通过注册中心查找可用实例

---

## 13.2 搭建服务注册中心

### Eureka 简介

- **Eureka** 是 Netflix 开源的服务注册与发现组件，Spring Cloud 对其进行了集成
- Eureka Server 充当服务注册中心，所有微服务向其注册并从中查找其他服务

### 创建 Eureka Server

1. 添加依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

1. 主类添加注解：

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

1. 启动后访问 Eureka 仪表盘（默认端口 8761）查看已注册的服务

### 配置 Eureka

- 默认情况下 Eureka Server 也会尝试注册自己并获取注册表，**独立运行时需禁用**：

```yaml
eureka:
  instance:
    hostname: localhost
  client:
    fetch-registry: false       # 不从其他 Eureka 获取注册表
    register-with-eureka: false # 不将自己注册到 Eureka
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
server:
  port: 8761
```

- `fetch-registry: false` — 阻止 Eureka Server 尝试从其他 Eureka 节点拉取注册信息
- `register-with-eureka: false` — 阻止 Eureka Server 将自己注册为客户端
- `defaultZone` — 指定 Eureka Server 的地址

### 自我保护模式（Self-Preservation）

- 开发环境下 Eureka 可能会显示红色警告："EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP…"
- 这是 **自我保护模式**：当 Eureka 发现大量实例心跳丢失时，会保留注册信息而不是删除
- 开发时可关闭：

```yaml
eureka:
  server:
    enable-self-preservation: false
```

### Eureka 高可用（Scaling Eureka）

- 生产环境中应部署 **多个 Eureka 实例** 实现高可用
- 多个 Eureka Server 互相注册，形成集群
- 示例：两个 Eureka 实例互相注册

```yaml
# eureka1 配置
eureka:
  client:
    service-url:
      defaultZone: http://eureka2:8762/eureka

# eureka2 配置
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:8761/eureka
```

- 客户端配置多个 Eureka 地址，任一不可用时自动切换：

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://eureka1:8761/eureka,http://eureka2:8762/eureka
```

---

## 13.3 注册与发现服务

### 将服务注册到 Eureka

1. 添加 Eureka Client 依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

1. 添加注解（Spring Cloud 较新版本中可省略，自动配置生效）：

```java
@SpringBootApplication
@EnableEurekaClient  // 或 @EnableDiscoveryClient（更通用）
public class SomeServiceApplication { ... }
```

- `@EnableEurekaClient` — 仅支持 Eureka
- `@EnableDiscoveryClient` — 支持多种服务发现实现（Eureka、Consul 等）

### 配置 Eureka Client 属性

```yaml
spring:
  application:
    name: some-service   # 服务名称，注册到 Eureka 的标识

eureka:
  client:
    service-url:
      defaultZone: http://eureka1:8761/eureka
```

- [`spring.application.name`](http://spring.application.name) 非常重要，它是其他服务查找该服务的**唯一标识**

### 消费服务（Consuming Services）

Spring Cloud 提供了多种方式来消费 Eureka 中注册的服务：

### 方式一：负载均衡的 RestTemplate

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

- `@LoadBalanced` 使 RestTemplate 具备通过**服务名**调用的能力
- 使用服务名替代主机名：

```java
// 不再硬编码 URL
// restTemplate.getForObject("http://localhost:8080/ingredients", ...)

// 使用服务名
restTemplate.getForObject("http://ingredient-service/ingredients", Ingredient[].class);
```

- Ribbon（客户端负载均衡器）会自动将服务名解析为实际的 host:port，并在多个实例间做负载均衡

### 方式二：WebClient（响应式）

```java
@Bean
@LoadBalanced
public WebClient.Builder webClientBuilder() {
    return WebClient.builder();
}
```

```java
Mono<Ingredient[]> ingredients = webClientBuilder.build()
    .get()
    .uri("http://ingredient-service/ingredients")
    .retrieve()
    .bodyToMono(Ingredient[].class);
```

### 方式三：Feign 声明式客户端

1. 添加依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

1. 启用 Feign：

```java
@SpringBootApplication
@EnableFeignClients
public class SomeServiceApplication { ... }
```

1. 定义 Feign 接口：

```java
@FeignClient("ingredient-service")  // 指定目标服务名
public interface IngredientClient {

    @GetMapping("/ingredients")
    Iterable<Ingredient> getAllIngredients();

    @GetMapping("/ingredients/{id}")
    Ingredient getIngredient(@PathVariable("id") String id);
}
```

1. 注入使用：

```java
@Autowired
private IngredientClient ingredientClient;

// 像调用本地方法一样调用远程服务
Iterable<Ingredient> ingredients = ingredientClient.getAllIngredients();
```

- Feign 会自动处理服务发现、负载均衡和 HTTP 调用

---

## 本章小结

- 微服务架构将应用拆分为独立部署的小服务，服务间通过网络通信
- **服务发现** 解决了微服务动态网络位置的问题
- **Eureka Server** 作为注册中心，维护所有服务实例的信息
- 服务通过 Eureka Client 自动注册，并通过服务名查找其他服务
- 消费服务有三种主要方式：
    - `@LoadBalanced RestTemplate` — 最简单
    - `@LoadBalanced WebClient` — 响应式场景
    - `Feign` — 声明式，最优雅
- 生产环境应部署多个 Eureka 实例保证高可用