---
title: 第1章 Spring起步
date: 2026-02-17 15:10:41
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
## 本章概要

- Spring 和 Spring Boot 核心要点
- 初始化 Spring 项目
- Spring 生态概览

---

## 1.1 什么是 Spring

- Spring 的核心是一个 **容器（Container）**，也称为 **Spring 应用上下文（Spring Application Context）**，负责创建和管理应用组件（Bean）
- 组件之间通过 **依赖注入（Dependency Injection, DI）** 进行装配
- Spring 容器使用 **控制反转（Inversion of Control, IoC）** 原则：对象的创建和生命周期管理由框架完成，而非由应用代码自身控制

### 组件装配方式

- **XML 配置**：早期方式，使用 XML 文件描述 Bean 及其依赖关系
- **Java 配置（JavaConfig）**：使用 `@Configuration` 和 `@Bean` 注解，更加类型安全
- **自动配置（Automatic Configuration）**：Spring Boot 的核心能力，根据类路径、已有 Bean 和属性设置自动推断并配置组件

> Spring Boot 的自动配置大大减少了显式配置的代码量，让开发者能够专注于业务逻辑。
> 

## 1.2 初始化 Spring 应用

### 使用 Spring Initializr

- 可通过以下方式创建项目：
    - **Spring Initializr 网站**：[https://start.spring.io](https://start.spring.io)
    - **IDE 集成**（如 IntelliJ IDEA、VS Code 的 Spring 插件）
    - **Spring Boot CLI**
    - **curl 命令行**

### 项目结构

- `src/main/java`：应用源码
- `src/main/resources`：非 Java 资源（如配置文件）
    - [`application.properties`](http://application.properties) 或 `application.yml`：应用配置
    - `static/`：静态资源（CSS、JS、图片等）
    - `templates/`：模板文件（如 Thymeleaf）
- `src/test/java`：测试代码

### 关键文件

- **引导类（Bootstrap class）**：带有 `@SpringBootApplication` 注解的主类
    - `@SpringBootApplication` 是一个组合注解，包含：
        - `@SpringBootConfiguration`：标识该类为配置类
        - `@EnableAutoConfiguration`：启用 Spring Boot 自动配置
        - `@ComponentScan`：启用组件扫描，自动发现 `@Component`、`@Service`、`@Controller` 等注解的类
- **pom.xml / build.gradle**：构建规范，声明依赖和插件

### 一个最简示例

```java
@SpringBootApplication
public class TacoCloudApplication {
    public static void main(String[] args) {
        SpringApplication.run(TacoCloudApplication.class, args);
    }
}
```

## 1.3 编写 Spring 应用

### 处理 Web 请求

- **控制器（Controller）**：处理 HTTP 请求并返回响应
- 使用 `@Controller` 注解标识控制器类
- 使用 `@GetMapping` 处理 GET 请求（类似的还有 `@PostMapping`、`@PutMapping` 等）

```java
@Controller
public class HomeController {
    @GetMapping("/")
    public String home() {
        return "home"; // 返回视图名称
    }
}
```

### 视图（View）

- 控制器方法返回的字符串是 **逻辑视图名**
- Spring 通过视图解析器（如 Thymeleaf）将逻辑视图名映射到具体模板

### 测试控制器

```java
@SpringBootTest
@AutoConfigureMockMvc
public class HomeControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void testHomePage() throws Exception {
        mockMvc.perform(get("/"))
            .andExpect(status().isOk())
            .andExpect(view().name("home"));
    }
}
```

## 1.4 俯瞰 Spring 生态

### Spring 核心框架

- 提供核心容器和 DI 框架
- Spring MVC（Web 框架）
- 数据持久化支持（JdbcTemplate 等）
- 响应式编程（Spring WebFlux）

### Spring Boot

- 自动配置、起步依赖（Starter Dependencies）
- 内嵌服务器（Tomcat、Jetty、Undertow）
- Actuator：运行时监控
- 简化的环境配置和属性管理

### Spring Data

- 将数据仓库抽象为简单的 Java 接口
- 支持 JPA、MongoDB、Redis、Cassandra 等多种数据存储

### Spring Security

- 认证（Authentication）与授权（Authorization）
- 支持 OAuth2、LDAP 等

### Spring Integration & Spring Batch

- **Spring Integration**：实时集成，基于消息驱动
- **Spring Batch**：批处理集成

### Spring Cloud

- 微服务相关支持（配置管理、服务发现、断路器等）

---

<aside>
💡

**小结**：Spring 的目标是简化 Java 开发。Spring Boot 通过自动配置和起步依赖进一步降低了使用门槛，让开发者可以快速搭建一个可运行的应用，专注于编写业务代码。

</aside>