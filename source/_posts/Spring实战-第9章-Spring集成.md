---
title: 第9章 Spring集成
date: 2026-02-17 15:10:47
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章主要介绍 **Spring Integration**，即如何在应用程序中定义集成流（Integration Flow），实现数据的实时处理以及与外部系统的集成。

---

## 9.1 声明一个简单的集成流

Spring Integration 是一个用于创建**企业集成模式（EIP）**的框架，它允许应用通过消息传递与其他应用或外部系统进行交互。

### 核心依赖

在 `pom.xml` 中添加 Spring Integration 的 starter 依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-integration</artifactId>
</dependency>
```

根据需要还可以添加特定的集成模块（如文件、邮件、JMS 等）：

```xml
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-file</artifactId>
</dependency>
```

### 集成流的三种定义方式

Spring Integration 支持三种定义集成流的方式：

- **XML 配置**：传统方式，使用 `<int:>` 命名空间
- **Java 配置**：使用 `@Configuration` 类和 `@Bean` 方法定义各组件
- **Java DSL**：使用流式 API（`IntegrationFlow`），最简洁直观

> 书中推荐使用 **Java DSL** 方式，代码更简洁且易于维护。
> 

---

## 9.2 Spring Integration 全景

### 核心组件概览

| **组件** | **说明** |
| --- | --- |
| **通道（Channel）** | 在集成流中传递消息的管道 |
| **过滤器（Filter）** | 根据条件决定消息是否继续传递 |
| **转换器（Transformer）** | 对消息进行格式或内容转换 |
| **路由器（Router）** | 根据条件将消息分发到不同的通道 |
| **切分器（Splitter）** | 将一个消息拆分成多个消息 |
| **聚合器（Aggregator）** | 将多个消息合并为一个消息（与切分器相反） |
| **服务激活器（Service Activator）** | 将消息交给某个 Java 方法处理 |
| **通道适配器（Channel Adapter）** | 将通道连接到外部系统（入站/出站） |
| **网关（Gateway）** | 通过接口将数据传入集成流 |

### 消息（Message）

消息由两部分组成：

- **Header（头信息）**：消息的元数据
- **Payload（负载）**：消息携带的实际数据

---

## 9.3 消息通道（Message Channels）

消息通道是集成流中组件之间的"管道"。Spring Integration 提供了多种通道实现：

- **DirectChannel**（默认）：点对点，发送者线程中同步调用消费者
- **QueueChannel**：消息存入队列，消费者从队列中拉取
- **PublishSubscribeChannel**：消息广播给所有订阅者
- **PriorityChannel**：类似 QueueChannel，但按优先级排序
- **RendezvousChannel**：发送者阻塞直到消费者接收消息

Java DSL 中可以通过 `.channel()` 方法指定通道：

```java
@Bean
public IntegrationFlow myFlow() {
    return IntegrationFlow.from("inputChannel")
        .channel("someChannel")
        .get();
}
```

---

## 9.4 过滤器（Filters）

过滤器用于决定哪些消息可以继续在流中传递。

```java
@Bean
public IntegrationFlow evenNumberFlow() {
    return IntegrationFlow.from("inputChannel")
        .<Integer>filter(n -> n % 2 == 0)
        .channel("evenChannel")
        .get();
}
```

---

## 9.5 转换器（Transformers）

转换器对消息的 payload 进行转换处理。

```java
@Bean
public IntegrationFlow transformFlow() {
    return IntegrationFlow.from("inputChannel")
        .<String, String>transform(String::toUpperCase)
        .channel("outputChannel")
        .get();
}
```

也可以引用一个 Spring Bean 的方法作为转换器。

---

## 9.6 路由器（Routers）

路由器根据条件将消息发送到不同的通道。

```java
@Bean
public IntegrationFlow routerFlow() {
    return IntegrationFlow.from("inputChannel")
        .<Integer, String>route(n -> n % 2 == 0 ? "evenChannel" : "oddChannel")
        .get();
}
```

---

## 9.7 切分器（Splitters）

切分器将一个消息拆分为多个消息。常见场景：

- 将包含多项内容的消息拆分为每项一条消息
- 将一个复杂的消息对象拆分为不同部分

```java
@Bean
public IntegrationFlow splitFlow() {
    return IntegrationFlow.from("inputChannel")
        .split()   // 默认拆分集合类型的 payload
        .channel("splitChannel")
        .get();
}
```

也可以使用自定义的拆分逻辑：

```java
public class OrderSplitter {
    public Collection<Object> splitOrder(PurchaseOrder order) {
        List<Object> items = new ArrayList<>();
        items.add(order.getBillingInfo());
        items.add(order.getLineItems());
        return items;
    }
}
```

---

## 9.8 服务激活器（Service Activators）

服务激活器是消息的"终点"——它接收消息并调用某个方法来处理。

```java
@Bean
public IntegrationFlow serviceActivatorFlow() {
    return IntegrationFlow.from("inputChannel")
        .<String>handle((payload, headers) -> {
            // 处理消息
            System.out.println(payload);
            return null;  // 返回 null 表示流结束
        })
        .get();
}
```

也可以将消息交给一个 Bean 的方法来处理：

```java
@Bean
public IntegrationFlow flow() {
    return IntegrationFlow.from("inputChannel")
        .handle("myService", "handleMessage")
        .get();
}
```

---

## 9.9 网关（Gateways）

网关是集成流的**入口点**。通过定义一个接口并使用 `@MessagingGateway` 注解，可以将方法调用自动转换为消息发送。

```java
@MessagingGateway(defaultRequestChannel = "inputChannel")
public interface FileWriterGateway {
    void writeToFile(
        @Header(FileHeaders.FILENAME) String filename,
        String data);
}
```

调用 `writeToFile()` 时，参数会被封装为消息并发送到 `inputChannel`。

---

## 9.10 通道适配器（Channel Adapters）

通道适配器是连接外部系统与集成流的桥梁：

- **入站通道适配器（Inbound）**：从外部系统读取数据到集成流
- **出站通道适配器（Outbound）**：将集成流的数据写入外部系统

```java
// 入站：从文件系统读取文件
@Bean
public IntegrationFlow fileReaderFlow() {
    return IntegrationFlow
        .from(Files.inboundAdapter(new File("/tmp/input"))
            .patternFilter("*.txt"))
        .channel("fileChannel")
        .get();
}
```

```java
// 出站：将消息写入文件
@Bean
public IntegrationFlow fileWriterFlow() {
    return IntegrationFlow.from("inputChannel")
        .handle(Files.outboundAdapter(new File("/tmp/output")))
        .get();
}
```

---

## 9.11 端点模块（Endpoint Modules）

Spring Integration 提供了丰富的端点模块，用于与各种外部系统集成：

| **模块** | **依赖（Artifact ID）** |
| --- | --- |
| 文件系统 | `spring-integration-file` |
| FTP / FTPS | `spring-integration-ftp` |
| Email | `spring-integration-mail` |
| JMS | `spring-integration-jms` |
| TCP / UDP | `spring-integration-ip` |
| JDBC | `spring-integration-jdbc` |
| RMI | `spring-integration-rmi` |
| RSS / Atom | `spring-integration-feed` |
| Twitter | `spring-integration-twitter` |
| Web Services | `spring-integration-ws` |

每个模块都提供了对应的通道适配器，用于入站和出站集成。

---

## 9.12 创建 Email 集成流（示例）

书中以 **Email 集成**为例，演示如何使用 Spring Integration 监听邮箱并自动处理邮件，将邮件中的订单信息解析并提交：

1. 使用 `Mail.imapInboundAdapter()` 作为入站适配器轮询邮箱
2. 使用**转换器**将邮件内容解析为订单对象
3. 使用**服务激活器**将订单提交到系统中

```java
@Bean
public IntegrationFlow tacoOrderEmailFlow(
        EmailProperties emailProps,
        EmailToOrderTransformer emailToOrderTransformer,
        OrderSubmitMessageHandler orderSubmitHandler) {
    return IntegrationFlow
        .from(Mail.imapInboundAdapter(emailProps.getImapUrl()),
            e -> e.poller(Pollers.fixedDelay(emailProps.getPollRate())))
        .transform(emailToOrderTransformer)
        .handle(orderSubmitHandler)
        .get();
}
```

---

## 本章小结

- Spring Integration 使应用能够定义**集成流**，在数据进出应用时进行处理
- 集成流可以通过 **XML**、**Java 配置** 或 **Java DSL** 三种方式定义
- **消息网关**和**通道适配器**是集成流的入口和出口
- 消息在流中可以被**转换**、**切分**、**聚合**、**路由**，以及被**服务激活器**处理
- **消息通道**连接集成流中的各个组件