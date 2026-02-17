---
title: 第19章 部署Spring
date: 2026-02-17 15:10:55
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章是全书的最后一章，介绍如何将 Spring Boot 应用**部署到各种环境**中。Spring Boot 的灵活性意味着你有多种部署选项——从传统的应用服务器到云平台，再到容器化部署。

---

## 19.1 权衡部署选项

### 部署方式概览

Spring Boot 应用有多种部署方式，取决于目标环境：

| **部署方式** | **说明** | **适用场景** |
| --- | --- | --- |
| 可执行 JAR（默认） | 使用内嵌服务器，通过 `java -jar` 直接运行 | 云部署、Docker 容器、独立运行 |
| WAR 文件 | 部署到传统的 Java 应用服务器（Tomcat、WebSphere 等） | 已有应用服务器基础设施的企业环境 |
| Cloud Foundry | 推送到 Cloud Foundry 等 PaaS 平台 | 云原生应用、PaaS 平台 |
| Docker 容器 | 打包为 Docker 镜像并在容器中运行 | 微服务架构、Kubernetes 编排 |

<aside>
💡

Spring Boot 项目默认以**可执行 JAR** 的形式打包，内嵌 Tomcat（或 Jetty/Undertow）服务器，这也是最推荐的部署方式——简单、独立、便于容器化。

</aside>

---

## 19.2 构建和部署 WAR 文件

### 为什么需要 WAR

虽然 JAR 是首选，但某些企业环境要求必须将应用部署到已有的 **Java 应用服务器**（如 Tomcat、WebLogic、WebSphere）中，这时就需要 WAR 包。

### 改造步骤

**第一步：修改打包方式**

在 `pom.xml` 中将打包类型从 `jar` 改为 `war`：

```xml
<packaging>war</packaging>
```

**第二步：继承 SpringBootServletInitializer**

主应用类需要继承 `SpringBootServletInitializer` 并重写 `configure()` 方法：

```java
@SpringBootApplication
public class TacoCloudApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(
            SpringApplicationBuilder builder) {
        return builder.sources(TacoCloudApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(TacoCloudApplication.class, args);
    }
}
```

- `SpringBootServletInitializer` 是一个特殊的 `WebApplicationInitializer`，由 Servlet 容器在启动时调用
- `configure()` 方法告诉容器用哪个 Spring 配置类来初始化应用
- `main()` 方法保留不变——这样应用既可以作为 WAR 部署到外部服务器，也可以作为可执行 JAR 独立运行

**第三步：排除内嵌服务器（可选）**

部署到外部服务器时，可以将内嵌 Tomcat 标记为 `provided`：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

> 设为 `provided` 后，编译时仍可使用 Tomcat API，但打包时不会将 Tomcat JAR 包含进去，避免与外部服务器冲突。
> 

**第四步：构建 WAR**

```bash
mvnw package
# 或
mvn package
```

生成的 WAR 文件位于 `target/` 目录下，将其复制到应用服务器的部署目录即可。

---

## 19.3 推送到 Cloud Foundry

### Cloud Foundry 简介

- **Cloud Foundry** 是一个开源的 PaaS（Platform as a Service）平台
- 支持多种语言和框架，对 Spring Boot 应用有**开箱即用的支持**
- 提供了自动化的构建、部署、扩展和管理能力

### 使用 cf CLI 部署

安装 Cloud Foundry CLI 后，通过以下步骤部署：

```bash
# 1. 登录 Cloud Foundry
cf login -a https://api.run.pivotal.io

# 2. 推送应用（使用可执行 JAR）
cf push taco-cloud -p target/taco-cloud-0.0.1-SNAPSHOT.jar

# 3. 或者推送 WAR 文件
cf push taco-cloud -p target/taco-cloud-0.0.1-SNAPSHOT.war
```

- `cf push` 会自动检测应用类型，选择合适的 **buildpack** 进行构建
- 对于 Spring Boot JAR，CF 会自动识别并使用 Java buildpack
- CF 会为应用分配一个随机 URL，也可以通过 `--hostname` 参数指定

### 使用 manifest.yml

可以创建 `manifest.yml` 来描述部署配置，避免每次都输入参数：

```yaml
applications:
  - name: taco-cloud
    host: taco-cloud
    path: target/taco-cloud-0.0.1-SNAPSHOT.jar
    instances: 1
    memory: 1G
    env:
      SPRING_PROFILES_ACTIVE: cloud
```

然后只需运行：

```bash
cf push
```

### 绑定服务

Cloud Foundry 通过**服务绑定（Service Binding）**为应用提供数据库、消息队列等后端服务：

```bash
# 创建 MySQL 服务实例
cf create-service p-mysql 100mb taco-cloud-db

# 将服务绑定到应用
cf bind-service taco-cloud taco-cloud-db

# 重启应用以使绑定生效
cf restage taco-cloud
```

<aside>
📌

绑定服务后，Cloud Foundry 会自动通过环境变量（`VCAP_SERVICES`）向应用注入数据库连接信息，Spring Boot 的 **Cloud Connector** 会自动识别并配置数据源——无需手动修改配置文件。

</aside>

---

## 19.4 在 Docker 容器中运行

### 为什么使用 Docker

- **一致性**：开发、测试、生产环境完全一致——"在我机器上能跑"不再是问题
- **隔离性**：每个容器独立运行，互不影响
- **可移植性**：Docker 镜像可以在任何支持 Docker 的环境中运行
- **编排支持**：配合 Kubernetes 等编排工具实现自动化部署、扩缩容

### 创建 Dockerfile

在项目根目录创建 `Dockerfile`：

```docker
FROM openjdk:11-jre-slim
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

- `FROM`：基于 OpenJDK 11 精简版镜像
- `ARG`：定义构建参数，指向打包好的 JAR 文件
- `COPY`：将 JAR 复制到镜像中
- `ENTRYPOINT`：定义容器启动命令

### 构建和运行 Docker 镜像

```bash
# 先构建 JAR
mvnw package

# 构建 Docker 镜像
docker build -t tacocloud/taco-cloud .

# 运行容器
docker run -p 8080:8080 tacocloud/taco-cloud
```

### 使用 Spring Boot 的分层 JAR

Spring Boot 支持将 JAR 拆分为多个层（layers），利用 Docker 的**层缓存机制**加速镜像构建：

```docker
FROM openjdk:11-jre-slim as builder
WORKDIR application
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

FROM openjdk:11-jre-slim
WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

<aside>
💡

分层构建的好处：当代码变更但依赖没变时，Docker 只需重新构建**应用层**，依赖层直接使用缓存，大幅加速构建速度。

</aside>

### 使用 Spring Boot Maven/Gradle 插件构建镜像

Spring Boot 2.3+ 提供了无需编写 Dockerfile 即可构建 OCI 镜像的能力：

```bash
# Maven
mvnw spring-boot:build-image

# Gradle
gradlew bootBuildImage
```

- 使用 **Cloud Native Buildpacks** 自动构建优化的 Docker 镜像
- 无需手动编写 Dockerfile
- 生成的镜像自动分层、优化

---

## 19.5 以终为始

<aside>
📖

这是全书的最后一节，作者做了一个总结性的回顾：从第 1 章的 Spring 起步到第 19 章的部署，整个 Taco Cloud 应用的开发旅程走到了终点——但这也是新旅程的起点。

</aside>

### 全书回顾

- **Part 1（基础）**：Spring 起步、开发 Web 应用、使用数据、安全保护、配置属性
- **Part 2（集成）**：创建/消费 REST 服务、发送异步消息、Spring Integration
- **Part 3（反应式）**：Reactor、反应式 API、反应式数据持久化
- **Part 4（云原生）**：服务发现、配置管理、处理失败和延迟
- **Part 5（部署和管理）**：Actuator、Spring Boot Admin、JMX 监控、部署

---

## 19.6 小结

- Spring Boot 应用可以部署为 **可执行 JAR**（推荐）、**WAR**（传统应用服务器）或 **Docker 容器**
- 构建 WAR 时，需要继承 `SpringBootServletInitializer` 并重写 `configure()` 方法
- **Cloud Foundry** 等 PaaS 平台对 Spring Boot 有开箱即用的支持，使用 `cf push` 即可快速部署
- **Docker** 容器化是当前最主流的部署方式，Spring Boot 提供了分层 JAR 和内置 buildpack 支持
- Spring Boot Maven/Gradle 插件的 `build-image` 命令可以**无需 Dockerfile** 直接构建优化的容器镜像
- 无论选择哪种部署方式，Spring Boot 的核心理念不变：**让开发者专注于业务逻辑，而不是基础设施**