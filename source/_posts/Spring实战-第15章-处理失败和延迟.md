---
title: 第15章 处理失败和延迟
date: 2026-02-17 15:10:52
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章属于 **Part 4: 云原生Spring** 的最后一章，主要介绍如何使用 **断路器模式（Circuit Breaker Pattern）** 来处理微服务架构中的失败和延迟问题，核心工具为 **Netflix Hystrix**。

---

## 15.1 理解断路器

### 为什么需要断路器？

在微服务架构中，服务之间存在大量远程调用。当某个下游服务**变慢或不可用**时，调用方的线程会被阻塞等待响应，最终可能导致：

- 线程资源耗尽
- **级联故障（Cascading Failure）**：一个服务的失败逐级传播到整个系统
- 用户体验严重下降

### 断路器的三种状态

断路器的工作原理类似于电路中的保险丝：

| 状态 | 说明 | 行为 |
| --- | --- | --- |
| **Closed（关闭）** | 正常状态 | 请求正常通过，失败次数被记录 |
| **Open（打开）** | 故障状态 | 请求直接被拦截，执行 fallback 方法，不再调用下游服务 |
| **Half-Open（半开）** | 恢复探测 | 允许少量请求通过以检测下游是否恢复，决定回到 Closed 还是 Open |

> 断路器模式的核心思想：**快速失败（fail fast）**，避免无意义的等待，同时提供优雅降级。
> 

---

## 15.2 声明断路器

### 添加依赖

在项目中引入 Spring Cloud Netflix Hystrix：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

### 启用 Hystrix

在主应用类上添加 `@EnableHystrix` 注解：

```java
@SpringBootApplication
@EnableHystrix
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### 使用 @HystrixCommand 声明断路器

在可能失败的方法上添加 `@HystrixCommand`，并指定 **fallback 方法**：

```java
@HystrixCommand(fallbackMethod = "getDefaultIngredients")
public Iterable<Ingredient> getAllIngredients() {
    // 远程服务调用
    return restTemplate.getForObject("http://ingredient-service/ingredients", 
                                      Ingredient[].class);
}

// fallback 方法：签名必须与原方法一致
public Iterable<Ingredient> getDefaultIngredients() {
    // 返回默认值或缓存数据
    return Arrays.asList(new Ingredient("DEFAULT", "默认配料", Type.WRAP));
}
```

<aside>
💡

fallback 方法的**参数和返回类型**必须与原方法完全一致。fallback 本身也可以再加 `@HystrixCommand`，形成 **fallback 链**。

</aside>

### 缓解延迟（Mitigating Latency）

Hystrix 默认对方法调用设置 **1秒超时**。如果方法执行超过 1 秒，断路器将中断调用并执行 fallback。

可以通过 `commandProperties` 自定义超时时间：

```java
@HystrixCommand(
    fallbackMethod = "getDefaultIngredients",
    commandProperties = {
        @HystrixProperty(
            name = "execution.isolation.thread.timeoutInMilliseconds",
            value = "500"  // 超时设为 500ms
        )
    }
)
public Iterable<Ingredient> getAllIngredients() { ... }
```

### 管理断路器阈值（Managing Circuit Breaker Thresholds）

Hystrix 通过以下参数决定何时打开断路器：

| 属性 | 说明 | 默认值 |
| --- | --- | --- |
| `circuitBreaker.requestVolumeThreshold` | 在滚动时间窗口内触发断路器的最小请求数 | 20 |
| `circuitBreaker.errorThresholdPercentage` | 触发断路器打开的失败比例 | 50% |
| `metrics.rollingStats.timeInMilliseconds` | 滚动时间窗口大小 | 10000ms（10秒） |
| `circuitBreaker.sleepWindowInMilliseconds` | 断路器打开后多久进入半开状态 | 5000ms（5秒） |

配置示例：

```java
@HystrixCommand(
    fallbackMethod = "getDefaultIngredients",
    commandProperties = {
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "30"),
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "25"),
        @HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "20000"),
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "60000")
    }
)
```

> 上面的配置表示：在 20 秒的窗口内，如果至少有 30 个请求且 25% 以上失败，则打开断路器；打开后 60 秒进入半开状态。
> 

---

## 15.3 监控失败

### Hystrix Dashboard 简介

Hystrix 提供了一个实时的 **监控仪表盘**，用于可视化每个断路器的运行状态。

**添加依赖：**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

**启用 Dashboard：**

```java
@SpringBootApplication
@EnableHystrixDashboard
public class HystrixDashboardApplication {
    public static void main(String[] args) {
        SpringApplication.run(HystrixDashboardApplication.class, args);
    }
}
```

访问 [`http://localhost:port/hystrix`](http://localhost:port/hystrix) 即可打开仪表盘界面。

### Dashboard 监控的关键指标

- **成功/失败/超时/拒绝**的请求数量
- 断路器当前状态（Closed / Open / Half-Open）
- **错误百分比**
- 请求频率（每秒请求数）
- **延迟百分位数**（p50、p99 等）
- 线程池使用情况

### 理解 Hystrix 线程池

Hystrix 默认使用 **线程池隔离** 策略：

- 每个 `@HystrixCommand` 方法在**独立的线程池**中执行，而非调用方线程
- 这提供了**舱壁隔离（Bulkhead Isolation）**：一个服务的延迟不会耗尽其他服务的线程
- 默认每个线程池大小为 **10 个线程**

<aside>
⚠️

线程池隔离会带来额外的线程上下文切换开销。对于延迟极低的本地调用，可以考虑使用**信号量隔离（Semaphore Isolation）**：

```java
@HystrixProperty(
    name = "execution.isolation.strategy",
    value = "SEMAPHORE"
)
```

</aside>

---

## 15.4 聚合多个 Hystrix 流

在微服务架构中，通常有多个服务各自暴露 Hystrix 流。**Turbine** 可以将多个服务的 Hystrix 流聚合为一个统一的流，便于集中监控。

### 添加 Turbine 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
</dependency>
```

### 启用 Turbine

```java
@SpringBootApplication
@EnableTurbine
public class TurbineServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(TurbineServerApplication.class, args);
    }
}
```

### 配置聚合的服务

在 `application.yml` 中指定需要聚合的服务：

```yaml
turbine:
  app-config: ingredient-service,taco-service  # 要监控的服务名
  cluster-name-expression: "'default'"          # 集群名称
```

配置好后，在 Hystrix Dashboard 中输入 Turbine 的流地址（如 [`http://turbine-host:port/turbine.stream`](http://turbine-host:port/turbine.stream)），即可看到所有服务的断路器聚合视图。

---

## 本章小结

<aside>
📌

- **断路器模式**通过快速失败和 fallback 机制防止级联故障
- **@HystrixCommand** 是声明断路器的核心注解，通过 `fallbackMethod` 提供降级逻辑
- Hystrix 支持**超时控制**和**阈值配置**来精细管理断路器行为
- **Hystrix Dashboard** 提供实时可视化监控
- **线程池隔离**实现舱壁模式，防止故障传播
- **Turbine** 聚合多个微服务的 Hystrix 流，实现集中监控
</aside>