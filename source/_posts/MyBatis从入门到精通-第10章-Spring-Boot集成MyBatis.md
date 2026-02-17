---
title: 第10章 Spring Boot集成MyBatis
date: 2026-02-17 15:09:43
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
## 10.1 基本的Spring Boot项目

Spring Boot 是 Spring 社区提供的一个全新框架，旨在简化 Spring 应用的搭建和开发过程。它采用**约定优于配置**的理念，开箱即用。

### 创建项目

通过 Spring Initializr（[https://start.spring.io）可以快速创建](https://start.spring.io）可以快速创建) Spring Boot 项目，选择以下基本依赖：

- **Spring Web**：提供 Web 开发支持
- **MySQL Driver**：MySQL 数据库驱动

生成的项目基本结构如下：

```
mybatis-spring-boot/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com.example/
│   │   │       └── Application.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── static/
│   │       └── templates/
│   └── test/
├── pom.xml
```

### pom.xml 核心配置

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.x.RELEASE</version>
</parent>

<dependencies>
    <!-- Spring Boot Web 依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Spring Boot 测试依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

### 入口类

Spring Boot 应用通过 `@SpringBootApplication` 注解标识入口类：

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

- `@SpringBootApplication` 是一个组合注解，包含 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan`
- Spring Boot 会自动扫描入口类所在包及其子包下的组件

---

## 10.2 集成MyBatis

### 添加 MyBatis Starter 依赖

在 `pom.xml` 中加入 `mybatis-spring-boot-starter` 和数据库驱动依赖：

```xml
<!-- MyBatis Spring Boot Starter -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>

<!-- MySQL 驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

<aside>
💡

**mybatis-spring-boot-starter** 是 MyBatis 官方提供的 Spring Boot 集成启动器，它会自动完成以下工作：

- 自动检测已有的 `DataSource`
- 使用 `DataSource` 创建并注册一个 `SqlSessionFactoryBean` 实例
- 创建并注册一个 `SqlSessionTemplate` 实例
- 自动扫描 Mapper 接口并注册到 Spring 容器
</aside>

### 配置数据源

在 [`application.properties`](http://application.properties) 中配置数据库连接信息：

```
# 数据源配置
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

### 配置 MyBatis

在 [`application.properties`](http://application.properties) 中配置 MyBatis 相关属性：

```
# MyBatis 配置
mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.example.model
```

使用 `@MapperScan` 注解指定 Mapper 接口的扫描路径：

```java
@SpringBootApplication
@MapperScan("com.example.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

> 也可以在每个 Mapper 接口上添加 `@Mapper` 注解，但使用 `@MapperScan` 更方便，不需要逐个标注。
> 

---

## 10.3 MyBatis Starter 配置介绍

`mybatis-spring-boot-starter` 提供了丰富的配置项，均以 `mybatis.` 为前缀，可在 [`application.properties`](http://application.properties) 或 `application.yml` 中配置。

### 常用配置项

| **配置项** | **说明** | **示例** |
| --- | --- | --- |
| `mybatis.config-location` | 指定 MyBatis 主配置文件路径 | `classpath:mybatis-config.xml` |
| `mybatis.mapper-locations` | 指定 Mapper XML 文件路径，支持通配符 | `classpath:mapper/*.xml` |
| `mybatis.type-aliases-package` | 指定类型别名包，自动注册该包下的类为别名 | `com.example.model` |
| `mybatis.type-handlers-package` | 指定类型处理器包 | `com.example.handler` |
| `mybatis.configuration.*` | 直接设置 MyBatis Configuration 对象的属性 | 见下方说明 |

### 通过 configuration 属性直接配置

可以直接在 [`application.properties`](http://application.properties) 中设置 MyBatis 的 `Configuration` 属性，无需额外的 `mybatis-config.xml`：

```
# 开启驼峰命名映射
mybatis.configuration.map-underscore-to-camel-case=true
# 开启延迟加载
mybatis.configuration.lazy-loading-enabled=true
# 设置默认的执行器类型
mybatis.configuration.default-executor-type=reuse
```

<aside>
⚠️

`mybatis.config-location` 和 `mybatis.configuration.*` **不能同时使用**。如果同时配置了两者，Spring Boot 启动时会抛出异常。选择其一即可：

- 使用 `mybatis-config.xml` 文件：通过 `mybatis.config-location` 指定
- 直接在 [`application.properties`](http://application.properties) 中配置：通过 `mybatis.configuration.*` 属性设置
</aside>

---

## 10.4 简单示例

本节以书中前面章节的 `simple` 项目为基础，演示在 Spring Boot 中使用 MyBatis。

### 10.4.1 引入 simple 依赖

将之前章节中的 `simple` 项目作为依赖引入，以复用已有的 Mapper 接口和 XML 文件：

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>simple</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

在 [`application.properties`](http://application.properties) 中配置 Mapper 扫描和 XML 路径：

```
# 扫描 simple 项目中的 Mapper XML
mybatis.mapper-locations=classpath:mapper/*.xml
# 类型别名包
mybatis.type-aliases-package=tk.mybatis.simple.model
```

同时修改 `@MapperScan` 指向 `simple` 项目的 Mapper 包：

```java
@SpringBootApplication
@MapperScan("tk.mybatis.simple.mapper")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

<aside>
💡

通过这种依赖引入的方式，可以直接复用之前章节中已经编写好的所有 Mapper 接口、XML 映射文件和实体类，避免重复编码。

</aside>

### 10.4.2 开发业务（Service）层

创建 Service 类，注入 Mapper 接口，添加 `@Service` 注解：

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    public List<SysUser> findAll() {
        return userMapper.selectAll();
    }

    public SysUser findById(Long id) {
        return userMapper.selectById(id);
    }
}
```

- `@Service` 将该类注册为 Spring 的 Bean
- `@Autowired` 自动注入 MyBatis 扫描并注册的 Mapper 代理对象
- 在 Spring Boot 中，事务管理可通过 `@Transactional` 注解实现

### 10.4.3 开发控制（Controller）层

创建 Controller 类，注入 Service，通过 RESTful 接口暴露服务：

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @RequestMapping("/users")
    public List<SysUser> findAll() {
        return userService.findAll();
    }

    @RequestMapping("/users/{id}")
    public SysUser findById(@PathVariable("id") Long id) {
        return userService.findById(id);
    }
}
```

- `@RestController` 是 `@Controller` + `@ResponseBody` 的组合注解，返回值自动序列化为 JSON
- `@RequestMapping` 映射 HTTP 请求路径
- `@PathVariable` 获取 URL 中的路径变量

### 10.4.4 运行应用查看效果

启动 Spring Boot 应用后，可以通过浏览器或工具访问接口：

- 查询所有用户：[`http://localhost:8080/users`](http://localhost:8080/users)
- 根据 ID 查询用户：[`http://localhost:8080/users/1`](http://localhost:8080/users/1)

> Spring Boot 内嵌了 Tomcat 服务器，无需额外部署 WAR 包，运行 `main` 方法即可启动应用。
> 

---

## 10.5 本章小结

| **对比项** | **Spring 集成 MyBatis（第9章）** | **Spring Boot 集成 MyBatis（第10章）** |
| --- | --- | --- |
| 配置方式 | XML 配置（`applicationContext.xml` 等） | [`application.properties`](http://application.properties) / `application.yml` |
| 依赖管理 | 手动管理各依赖版本 | Starter 自动管理依赖版本 |
| 自动配置 | 需手动配置 `SqlSessionFactory`、`MapperScannerConfigurer` 等 | Starter 自动配置，开箱即用 |
| 服务器部署 | 需外部 Servlet 容器（如 Tomcat） | 内嵌 Tomcat，`main` 方法直接启动 |
| 项目结构 | WAR 包结构，包含 `web.xml` | JAR 包结构，无需 `web.xml` |

<aside>
📝

**核心要点回顾**

- `mybatis-spring-boot-starter` 大幅简化了 MyBatis 与 Spring 的集成过程
- 通过 `@MapperScan` 或 `@Mapper` 注解注册 Mapper 接口
- 所有配置均可在 [`application.properties`](http://application.properties) 中完成，无需 XML 配置文件
- 注意 `mybatis.config-location` 和 `mybatis.configuration.*` 不可同时使用
- Spring Boot 内嵌服务器使开发和测试更加便捷
</aside>