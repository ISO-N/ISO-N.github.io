---
title: 第17章 管理Spring
date: 2026-02-17 15:10:53
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章介绍如何使用 **Spring Boot Admin** 来管理和监控 Spring Boot 应用程序。Spring Boot Admin 是由 codecentric 公司开发的开源项目，它基于 Actuator 端点，提供了一个友好的 Web UI 来可视化地管理应用。

---

## 17.1 使用 Spring Boot Admin

Spring Boot Admin 分为**服务端（Admin Server）**和**客户端（Admin Client）**两部分。服务端提供 Web UI，客户端是被监控的应用。

---

### 17.1.1 创建 Admin 服务端

创建一个新的 Spring Boot 项目，添加 Admin Server 依赖：

```xml
<dependency>
  <groupId>de.codecentric</groupId>
  <artifactId>spring-boot-admin-starter-server</artifactId>
</dependency>
```

在主应用类上添加 `@EnableAdminServer` 注解：

```java
@SpringBootApplication
@EnableAdminServer
public class AdminServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(AdminServerApplication.class, args);
    }
}
```

<aside>
💡

Admin Server 本质上是一个独立的 Spring Boot 应用，它收集并展示所有注册客户端的 Actuator 信息。默认端口为 `8080`，可通过 `server.port` 修改。

</aside>

---

### 17.1.2 注册 Admin 客户端

客户端（即你的 Spring Boot 应用）需要主动向 Admin Server 注册。添加客户端依赖：

```xml
<dependency>
  <groupId>de.codecentric</groupId>
  <artifactId>spring-boot-admin-starter-client</artifactId>
</dependency>
```

在 `application.yml` 中配置 Admin Server 的地址：

```yaml
spring:
  boot:
    admin:
      client:
        url: http://localhost:9090
```

同时需要暴露 Actuator 端点供 Admin Server 读取：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

<aside>
📌

除了手动注册客户端，还可以使用 **Spring Cloud Discovery**（如 Eureka）进行自动发现。Admin Server 会自动从服务注册中心获取所有已注册的应用实例。

</aside>

---

## 17.2 探索 Admin 服务端

Admin Server 启动后，访问其 Web UI 即可看到所有已注册的应用列表。点击某个应用可以进入详情页面，查看以下信息：

---

### 17.2.1 查看应用健康状况和基本信息

> 详情页的 **Details** 标签展示应用的健康状态、元数据以及基本信息。
> 

| **信息类别** | **说明** |
| --- | --- |
| Health | 来自 `/actuator/health`，展示 UP/DOWN 状态及各组件健康详情 |
| Info | 来自 `/actuator/info`，展示应用自定义的构建、版本等信息 |
| Metadata | 展示启动时间、JVM 版本、操作系统等运行环境元数据 |

---

### 17.2.2 监控关键指标

在 **Metrics** 标签中，可以查看应用的各种运行指标，包括：

- **JVM 内存使用**：堆内存、非堆内存的当前用量和最大值
- **系统 CPU 使用率**：进程和系统级别的 CPU 占用
- **线程数**：活跃线程、守护线程、峰值线程数
- **GC 统计**：垃圾回收次数和耗时
- **HTTP 请求统计**：请求数量、响应时间等

<aside>
💡

这些指标来自 Micrometer，对应 Actuator 的 `/actuator/metrics` 端点。你可以在下拉列表中选择具体的指标名称进行查看。

</aside>

---

### 17.2.3 查看环境属性

**Environment** 标签展示应用的所有环境属性，来自 `/actuator/env`：

- `application.yml` / [`application.properties`](http://application.properties) 中的配置
- 系统环境变量
- JVM 系统属性
- 命令行参数

<aside>
⚠️

敏感属性（如密码、密钥）会被自动脱敏为 `******`，这与 Actuator 的默认行为一致。

</aside>

---

### 17.2.4 查看和设置日志级别

**Loggers** 标签对应 `/actuator/loggers` 端点，提供以下功能：

- 查看所有 Logger 的当前级别（TRACE、DEBUG、INFO、WARN、ERROR）
- **动态修改日志级别**，无需重启应用

```
例如：将 org.springframework.web 的日志级别从 INFO 改为 DEBUG
→ 在 Admin UI 中找到该 Logger，直接选择新级别即可生效
```

<aside>
💡

动态调整日志级别在排查生产环境问题时非常有用，可以临时开启 DEBUG 日志来获取更多信息，问题解决后再调回。

</aside>

---

### 17.2.5 监控线程

**Threads** 标签展示应用的线程信息，对应 `/actuator/threaddump`：

- 查看所有活跃线程及其状态（RUNNABLE、WAITING、TIMED_WAITING、BLOCKED）
- 查看线程的堆栈跟踪信息
- 有助于排查**死锁**和**线程阻塞**问题

---

### 17.2.6 追踪 HTTP 请求

**HTTP Traces** 标签展示最近的 HTTP 请求记录，对应 `/actuator/httptrace`：

| **字段** | **说明** |
| --- | --- |
| Timestamp | 请求发生的时间 |
| Method | HTTP 方法（GET、POST 等） |
| URI | 请求路径 |
| Status | HTTP 响应状态码 |
| Time Taken | 请求处理耗时 |

<aside>
📌

默认情况下 HTTP trace 存储在内存中，仅保留最近 100 条记录。可以通过自定义 `HttpTraceRepository` 来持久化存储。

</aside>

---

## 17.3 保护 Admin 服务端

Admin Server 展示的信息非常敏感，因此在生产环境中**必须进行安全保护**。

---

### 17.3.1 启用 Admin 服务端的登录功能

为 Admin Server 添加 Spring Security 依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

配置用户名和密码：

```yaml
spring:
  security:
    user:
      name: admin
      password: secret123
```

添加安全配置类，允许 Admin UI 的静态资源和登录页面正常访问：

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final AdminServerProperties adminServer;

    public SecurityConfig(AdminServerProperties adminServer) {
        this.adminServer = adminServer;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        SavedRequestAwareAuthenticationSuccessHandler handler =
            new SavedRequestAwareAuthenticationSuccessHandler();
        handler.setTargetUrlParameter("redirectTo");
        handler.setDefaultTargetUrl(
            adminServer.getContextPath() + "/");

        http
            .authorizeRequests()
                .antMatchers(
                    adminServer.getContextPath() + "/assets/**")
                .permitAll()
                .antMatchers(
                    adminServer.getContextPath() + "/login")
                .permitAll()
                .anyRequest().authenticated()
            .and()
                .formLogin()
                .loginPage(adminServer.getContextPath() + "/login")
                .successHandler(handler)
            .and()
                .logout()
                .logoutUrl(adminServer.getContextPath() + "/logout")
            .and()
                .httpBasic()
            .and()
                .csrf()
                .csrfTokenRepository(
                    CookieCsrfTokenRepository.withHttpOnlyFalse())
                .ignoringAntMatchers(
                    adminServer.getContextPath() + "/instances",
                    adminServer.getContextPath()
                        + "/actuator/**");
    }
}
```

<aside>
💡

配置完成后，访问 Admin Server 将会重定向到登录页面，需要输入配置的用户名和密码才能访问。

</aside>

---

### 17.3.2 向 Actuator 进行认证

当客户端的 Actuator 端点也启用了安全认证时，Admin Server 需要携带凭证才能访问。在客户端配置中提供凭证：

```yaml
spring:
  boot:
    admin:
      client:
        url: http://localhost:9090
        instance:
          metadata:
            user.name: ${spring.security.user.name}
            user.password: ${spring.security.user.password}
```

<aside>
📌

通过 `instance.metadata` 将客户端的认证信息传递给 Admin Server。Admin Server 在访问客户端的 Actuator 端点时会自动使用这些凭证。

</aside>

---

## 小结

| **主题** | **要点** |
| --- | --- |
| Spring Boot Admin | 基于 Actuator 的 Web UI 管理工具，由 codecentric 开发，非 Spring 官方项目 |
| 服务端搭建 | 添加 `spring-boot-admin-starter-server` 依赖 + `@EnableAdminServer` 注解 |
| 客户端注册 | 添加 `spring-boot-admin-starter-client` 依赖，配置 `spring.boot.admin.client.url` |
| 监控功能 | 健康状态、指标、环境属性、日志级别、线程、HTTP 追踪 |
| 安全保护 | Admin Server 端使用 Spring Security 登录保护；客户端通过 metadata 传递 Actuator 凭证 |