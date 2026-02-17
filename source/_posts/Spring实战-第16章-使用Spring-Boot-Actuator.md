---
title: 第16章 使用Spring Boot Actuator
date: 2026-02-17 15:10:52
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章介绍 Spring Boot Actuator，它为 Spring Boot 应用提供**运行时监控和管理**能力。Actuator 通过一系列 HTTP 端点（以及 JMX MBean）让你窥探运行中的应用——查看配置、健康状态、指标等，甚至触发某些操作。

---

## 16.1 Actuator 概述

### 什么是 Actuator

- 在机械领域，actuator 是控制和移动机构的组件
- 在 Spring Boot 中，**Actuator** 扮演同样角色——让你**查看应用内部状态**并在一定程度上**控制应用行为**

### 启用 Actuator

添加起步依赖即可启用：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

启用后，Actuator 会自动暴露一系列 HTTP 端点，默认基础路径为 `/actuator`。

### Actuator 基础端点一览

| **HTTP 方法** | **路径** | **说明** |
| --- | --- | --- |
| GET | `/actuator/health` | 应用健康状况 |
| GET | `/actuator/info` | 应用信息 |
| GET | `/actuator/beans` | Spring 应用上下文中所有 Bean |
| GET | `/actuator/conditions` | 自动配置条件评估报告 |
| GET | `/actuator/configprops` | 配置属性及当前值 |
| GET | `/actuator/env` | 环境属性（来自各种 PropertySource） |
| GET | `/actuator/mappings` | 所有 HTTP 请求映射 |
| GET | `/actuator/loggers` | 日志记录器及其级别 |
| GET | `/actuator/metrics` | 应用指标列表 |
| GET | `/actuator/threaddump` | 线程转储 |
| GET | `/actuator/heapdump` | 堆转储（下载 hprof 文件） |
| POST | `/actuator/shutdown` | 优雅关闭应用（默认禁用） |

> 默认情况下，只有 `/health` 和 `/info` 端点对外暴露。其他端点需要通过配置显式开启。
> 

### 配置端点暴露

通过 `application.yml` 控制哪些端点对外暴露：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, info, beans, conditions, env, metrics, mappings, loggers
        # 或暴露全部端点：
        # include: "*"
        exclude: threaddump, heapdump
```

### 修改基础路径

默认基础路径为 `/actuator`，可以通过以下配置修改：

```yaml
management:
  endpoints:
    web:
      base-path: /management
```

---

## 16.2 消费 Actuator 端点

### /health — 健康检查

- 返回应用的整体健康状况
- 默认只返回一个简单的状态（`UP` 或 `DOWN`）
- 可配置显示详细信息：

```yaml
management:
  endpoint:
    health:
      show-details: always  # always | when-authorized | never
```

- 详细模式下会包含各子系统的健康指标（数据库、磁盘空间、消息代理等）
- Spring Boot 提供了多种**自动配置的健康指示器**：
    - `DataSourceHealthIndicator`：检查数据库连接
    - `DiskSpaceHealthIndicator`：检查磁盘空间
    - `MongoHealthIndicator`：检查 MongoDB 连接
    - `RabbitHealthIndicator`：检查 RabbitMQ 连接
    - 等等

### /info — 应用信息

- 默认返回空 JSON `{}`
- 可在 `application.yml` 中添加以 `info.` 开头的属性来填充内容：

```yaml
info:
  app:
    name: Taco Cloud
    description: 一个在线 Taco 订购应用
    version: 0.0.1-SNAPSHOT
```

### /beans — Bean 装配报告

- 列出 Spring 应用上下文中**所有已注册的 Bean**
- 每个 Bean 的信息包括：
    - Bean 名称
    - 类型（全限定类名）
    - 所在的应用上下文
    - 依赖的其他 Bean

### /conditions — 自动配置报告

- 列出所有自动配置条件的**评估结果**
- 分为三部分：
    - **positiveMatches**：条件匹配成功，配置已生效
    - **negativeMatches**：条件不匹配，配置未生效
    - **unconditionalClasses**：无条件配置（总是生效）

<aside>
💡

`/conditions` 端点在调试自动配置问题时非常有用——当你不确定某个自动配置为什么生效或没有生效时，查看这个端点的输出就能找到答案。

</aside>

### /env — 环境属性

- 列出**所有对应用生效的环境属性**及其来源（PropertySource）
- 属性来源包括：JVM 系统属性、操作系统环境变量、application.yml、命令行参数等
- 可查看特定属性：`/actuator/env/{propertyName}`

### /mappings — HTTP 映射

- 列出所有的 **HTTP 请求映射**
- 包括 Spring MVC 控制器映射和 Actuator 自身的端点映射
- 每条映射显示：请求方法、路径模式、处理方法（控制器类+方法名）

### /loggers — 日志管理

- 列出应用中所有**日志记录器及其配置的日志级别**
- 可查看特定 Logger：`/actuator/loggers/{loggerName}`
- **支持运行时修改日志级别**（POST 请求）：

```bash
curl -X POST http://localhost:8080/actuator/loggers/com.example \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}'
```

<aside>
📌

运行时修改日志级别是 Actuator 的一个非常实用的功能——无需重启应用即可开启 DEBUG 日志来排查问题。

</aside>

### /metrics — 应用指标

- 列出所有可用的**指标名称**
- 查看具体指标：`/actuator/metrics/{metricName}`
- 常见指标包括：
    - `jvm.memory.used`：JVM 内存使用量
    - `jvm.gc.pause`：GC 暂停时间
    - `http.server.requests`：HTTP 请求统计
    - `system.cpu.usage`：系统 CPU 使用率
    - `process.uptime`：进程运行时间
- 支持通过 `tag` 参数过滤指标：

```
/actuator/metrics/http.server.requests?tag=method:GET&tag=status:200
```

### /threaddump 和 /heapdump

- `/threaddump`：返回当前所有线程的**快照**，包括线程名、状态、堆栈跟踪和锁信息——用于诊断死锁和性能问题
- `/heapdump`：下载 **hprof 格式的堆转储文件**，可用 VisualVM 等工具分析内存泄漏

---

## 16.3 自定义 Actuator

### 自定义 /info 端点

除了在 `application.yml` 中静态配置外，还可以通过实现 `InfoContributor` 接口动态提供信息：

```java
@Component
public class TacoCountInfoContributor implements InfoContributor {

    private TacoRepository tacoRepo;

    public TacoCountInfoContributor(TacoRepository tacoRepo) {
        this.tacoRepo = tacoRepo;
    }

    @Override
    public void contribute(Info.Builder builder) {
        long tacoCount = tacoRepo.count();
        Map<String, Object> tacoMap = new HashMap<>();
        tacoMap.put("count", tacoCount);
        builder.withDetail("taco-stats", tacoMap);
    }
}
```

访问 `/actuator/info` 时会包含：

```json
{
  "taco-stats": {
    "count": 44
  }
}
```

### 自定义健康指示器

实现 `HealthIndicator` 接口来创建自定义健康检查：

```java
@Component
public class WackoHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        int hour = Calendar.getInstance().get(Calendar.HOUR_OF_DAY);
        if (hour > 12) {
            return Health.up()
                .withDetail("reason", "Everything is fine.")
                .build();
        } else {
            return Health.down()
                .withDetail("reason", "It's too early.")
                .build();
        }
    }
}
```

- `Health.up()`：构建状态为 UP 的健康信息
- `Health.down()`：构建状态为 DOWN 的健康信息
- `withDetail()`：添加额外的详情键值对

### 注册自定义指标

使用 Micrometer 的 `MeterRegistry` 注册自定义指标：

```java
@Component
public class TacoMetrics extends AbstractRepositoryEventListener<Taco> {

    private MeterRegistry meterRegistry;

    public TacoMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    @Override
    protected void onAfterCreate(Taco taco) {
        List<Ingredient> ingredients = taco.getIngredients();
        for (Ingredient ingredient : ingredients) {
            meterRegistry.counter("tacocloud",
                "ingredient", ingredient.getId()).increment();
        }
    }
}
```

- `MeterRegistry`：Micrometer 的核心接口，用于注册和管理指标
- `counter()`：创建或获取一个计数器
- 注册后可通过 `/actuator/metrics/tacocloud` 查看

### 创建自定义端点

使用 `@Endpoint` 注解创建全新的 Actuator 端点：

```java
@Component
@Endpoint(id = "notes", enableByDefault = true)
public class NotesEndpoint {

    private List<Note> notes = new ArrayList<>();

    @ReadOperation
    public List<Note> notes() {
        return notes;
    }

    @WriteOperation
    public List<Note> addNote(String text) {
        notes.add(new Note(text));
        return notes;
    }

    @DeleteOperation
    public List<Note> deleteNote(int index) {
        if (index < notes.size()) {
            notes.remove(index);
        }
        return notes;
    }
}
```

| **注解** | **HTTP 方法** | **说明** |
| --- | --- | --- |
| `@ReadOperation` | GET | 读取数据 |
| `@WriteOperation` | POST | 写入数据 |
| `@DeleteOperation` | DELETE | 删除数据 |

> 自定义端点通过 `/actuator/notes` 访问，同时也会自动作为 JMX MBean 暴露。
> 

---

## 16.4 保护 Actuator

### 为什么要保护

- Actuator 端点暴露了应用的**敏感内部信息**（环境变量、Bean 列表、配置属性等）
- 在生产环境中，**必须对 Actuator 端点进行安全保护**

### 使用 Spring Security 保护端点

结合 Spring Security，可以像保护其他 URL 一样保护 Actuator 端点：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .requestMatcher(EndpointRequest.toAnyEndpoint())
            .authorizeRequests()
                .anyRequest().hasRole("ADMIN")
            .and()
            .httpBasic();
    }
}
```

**关键类：**

- `EndpointRequest`：Spring Boot 提供的请求匹配器，专门用于匹配 Actuator 端点
    - `toAnyEndpoint()`：匹配所有 Actuator 端点
    - `to(HealthEndpoint.class)`：匹配特定端点
    - `toAnyEndpoint().excluding(HealthEndpoint.class)`：排除特定端点

可以更细粒度地控制访问：

```java
http
    .requestMatcher(EndpointRequest.toAnyEndpoint())
    .authorizeRequests()
        .requestMatchers(EndpointRequest.to(HealthEndpoint.class, InfoEndpoint.class))
            .permitAll()
        .anyRequest()
            .hasRole("ADMIN")
    .and()
    .httpBasic();
```

<aside>
📌

建议在生产环境中：`/health` 和 `/info` 可以公开访问（用于负载均衡器健康检查等），其余端点应限制为管理员角色才能访问。

</aside>

---

## 16.5 小结

- **Spring Boot Actuator** 提供了一组 HTTP 端点，用于在运行时监控和管理 Spring Boot 应用
- 只需添加 `spring-boot-starter-actuator` 依赖即可启用，通过配置控制暴露哪些端点
- 常用端点：`/health`（健康检查）、`/info`（应用信息）、`/env`（环境属性）、`/metrics`（指标）、`/loggers`（日志级别管理）
- 可以通过实现 `InfoContributor`、`HealthIndicator` 接口或使用 `MeterRegistry` 来**自定义 Actuator 信息**
- 使用 `@Endpoint` 注解可以创建**全新的自定义端点**
- 生产环境中应结合 **Spring Security** 保护 Actuator 端点，防止敏感信息泄露