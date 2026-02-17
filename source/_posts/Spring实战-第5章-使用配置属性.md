---
title: 第5章 使用配置属性
date: 2026-02-17 15:10:44
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
## 本章概要

- 微调自动配置的 bean
- 将配置属性应用于应用程序组件
- 使用 Spring 配置文件（Profile）

---

## 5.1 微调自动配置

- Spring 中存在两种不同但相关的配置：
    - **Bean 装配（Bean Wiring）**：声明应用程序组件在 Spring 应用上下文中如何创建以及相互注入
    - **属性注入（Property Injection）**：在 Spring 应用上下文中设置 Bean 的属性值
- Spring Boot 的自动配置极大减少了显式 Bean 装配的工作量，但属性配置仍然需要开发者手动调整

### 5.1.1 理解 Spring 环境抽象

- **Spring 环境抽象**是所有可配置属性的统一来源，它将以下属性源聚合在一起：
    - JVM 系统属性
    - 操作系统环境变量
    - 命令行参数
    - 应用程序属性配置文件（[`application.properties`](http://application.properties) / `application.yml`）
- Spring 环境会将这些属性源拉到一起，需要属性的 Bean 可以直接从 Spring 环境中获取

> 例如，假设希望应用的 Servlet 容器使用其他端口监听请求，只需在 [`application.properties`](http://application.properties) 中设置 `server.port`：
> 

```
server.port=9090
```

或者使用 `application.yml`：

```yaml
server:
  port: 9090
```

也可以通过命令行参数设置：

```bash
java -jar tacocloud.jar --server.port=9090
```

### 5.1.2 配置数据源

- 通过配置属性来指定数据库 URL 和凭证：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/tacocloud
    username: tacodb
    password: tacopassword
    driver-class-name: com.mysql.cj.jdbc.Driver
```

- Spring Boot 会根据数据库 URL 自动推断 JDBC 驱动类，通常无需显式指定 `driver-class-name`
- 如果类路径中有连接池（如 HikariCP），Spring Boot 会自动使用它
- 可以通过 `spring.datasource.schema` 和 [`spring.datasource.data`](http://spring.datasource.data) 指定初始化 SQL 脚本
- 可以配置 JNDI 数据源：

```yaml
spring:
  datasource:
    jndi-name: java:/comp/env/jdbc/tacoCloudDS
```

> 设置 `spring.datasource.jndi-name` 时，其他数据库连接属性会被忽略。
> 

### 5.1.3 配置嵌入式服务器

- `server.port=0`：随机选取可用端口，适用于自动化集成测试
- 配置 HTTPS：需要通过 `server.ssl` 相关属性指定密钥库信息

```yaml
server:
  ssl:
    key-store: classpath:keystore.jks
    key-store-password: letmein
    key-password: letmein
```

### 5.1.4 配置日志

- Spring Boot 默认使用 **Logback** 进行日志记录
- 可在 `application.yml` 中设置日志级别：

```yaml
logging:
  level:
    root: WARN
    org.springframework.security: DEBUG
```

- 配置日志输出到文件：

```yaml
logging:
  file:
    path: /var/logs/
    name: TacoCloud.log
```

### 5.1.5 使用特殊的属性值

- 使用 `${}` 占位符引用其他配置属性的值：

```yaml
greeting:
  welcome: You are using ${spring.application.name}.
```

---

## 5.2 创建自己的配置属性

- 配置属性并非 Spring 创建的 Bean 独享，只需少量配置，就可以让自己的 Bean 使用配置属性
- 使用 **`@ConfigurationProperties`** 注解将配置属性绑定到 Bean 的属性上

### 5.2.1 定义配置属性持有者

- 在 Bean 上添加 `@ConfigurationProperties`，指定属性前缀：

```java
@Component
@ConfigurationProperties(prefix = "taco.orders")
public class OrderProps {
    private int pageSize = 20;
    // getter / setter
}
```

- 在 `application.yml` 中设置：

```yaml
taco:
  orders:
    pageSize: 10
```

- 可以使用 **`@Validated`** 注解和 JSR-303（如 `@Min`、`@Max`、`@NotNull`）对属性值进行校验：

```java
@Component
@ConfigurationProperties(prefix = "taco.orders")
@Validated
public class OrderProps {
    @Min(value = 5, message = "must be between 5 and 25")
    @Max(value = 25, message = "must be between 5 and 25")
    private int pageSize = 20;
}
```

> 将配置属性抽取到单独的 `@ConfigurationProperties` 类中是一种最佳实践——这样可以在多个 Bean 之间共享通用配置，且便于维护。
> 

### 5.2.2 声明配置属性元数据

- IDE 中可能对自定义配置属性显示警告（缺少元数据）
- 创建 **配置属性元数据** 文件消除警告并提供自动补全支持：
    - 在 `src/main/resources/META-INF/` 下创建 `additional-spring-configuration-metadata.json`

```json
{
  "properties": [
    {
      "name": "taco.orders.pageSize",
      "type": "java.lang.Integer",
      "description": "Sets the maximum number of orders to display in a list."
    }
  ]
}
```

---

## 5.3 使用 Profile 进行配置

- 当应用部署到不同环境（开发、测试、生产）时，某些配置会有所不同
- **Spring Profile** 提供了一种条件化配置的机制，使得特定配置只在指定的 Profile 激活时才生效

### 5.3.1 定义特定 Profile 的属性

- 创建以 `application-{profile名}` 命名的属性文件：
    - `application-dev.yml`：开发环境配置
    - `application-prod.yml`：生产环境配置
- 也可以在同一个 YAML 文件中使用 `---` 分隔不同 Profile 的配置：

```yaml
# 默认配置
logging:
  level:
    tacos: DEBUG
---
spring:
  profiles: prod
logging:
  level:
    tacos: WARN
server:
  port: 8443
  ssl:
    key-store: classpath:keystore.jks
    key-store-password: letmein
    key-password: letmein
```

### 5.3.2 激活 Profile

- 在 `application.yml` 中指定激活的 Profile：

```yaml
spring:
  profiles:
    active: prod
```

- 通过命令行参数激活：

```bash
java -jar tacocloud.jar --spring.profiles.active=prod
```

- 通过环境变量激活：

```bash
export SPRING_PROFILES_ACTIVE=prod
```

- 可以同时激活多个 Profile（逗号分隔）：

```yaml
spring:
  profiles:
    active: prod, audit, ha
```

### 5.3.3 使用 Profile 条件化创建 Bean

- 使用 **`@Profile`** 注解可以让某些 Bean 仅在特定 Profile 激活时才创建：

```java
@Bean
@Profile("dev")
public CommandLineRunner dataLoader(IngredientRepository repo) {
    // 仅在 dev profile 激活时加载测试数据
}
```

- 也可以使用 `!` 表示取反：

```java
@Bean
@Profile("!prod")
public CommandLineRunner dataLoader(IngredientRepository repo) {
    // 在非 prod profile 下加载测试数据
}
```

- `@Profile` 可以放在 `@Configuration` 注解的类上，使整个配置类中的所有 Bean 只在特定 Profile 下生效

---

<aside>
💡

**小结**：Spring Boot 配置属性提供了一种灵活的方式来微调自动配置和自定义 Bean 的行为。通过 `application.yml` / [`application.properties`](http://application.properties) 可以统一管理配置，使用 `@ConfigurationProperties` 将外部配置绑定到 Java 对象，使用 Profile 实现多环境差异化配置。掌握这三个核心概念，可以让 Spring 应用在不同运行环境下更加灵活和可控。

</aside>