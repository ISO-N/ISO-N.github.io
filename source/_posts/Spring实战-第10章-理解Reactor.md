---
title: 第10章 理解Reactor
date: 2026-02-17 15:10:48
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
## 10.1 理解反应式编程

- **反应式编程**是一种可以替代命令式编程的编程范式，它是一种**函数式**和**声明式**的编程模型
- 反应式编程是对**数据流**的处理，数据像水流一样经过一系列操作被处理
- 类比：想象一根水管，数据从一端流入，经过各种处理（过滤、转换、聚合等）后从另一端流出

### 反应式流（Reactive Streams）规范

反应式流规范定义了4个核心接口：

- **Publisher**：数据的发布者，负责生成数据并发送给订阅者
- **Subscriber**：数据的订阅者，接收并处理数据
- **Subscription**：发布者和订阅者之间的关系，订阅者可以通过它管理数据请求
- **Processor**：既是 Publisher 又是 Subscriber，用于在数据管道中做中间处理

### 背压（Backpressure）

- 背压是反应式流的核心特性之一
- 订阅者可以通过 `Subscription.request(n)` 告诉发布者自己能处理多少数据
- 防止快速的发布者压垮慢速的订阅者，避免系统过载

---

## 10.2 初识 Reactor

- **Project Reactor** 是 Spring 5 反应式编程模型的基础
- Reactor 实现了反应式流规范，提供了两个核心组合类型：

### Mono 和 Flux

| 类型 | 说明 | 数据量 |
| --- | --- | --- |
| **Mono** | 包含 0 或 1 个元素的反应式类型 | 0..1 |
| **Flux** | 包含 0 到 N 个元素的反应式类型 | 0..N |
- **Flux** 代表一个包含零个、一个或多个数据项的管道
- **Mono** 代表一个包含零个或一个数据项的管道（针对最多一个数据项的场景做了优化）
- 两者都实现了 Reactive Streams 的 `Publisher` 接口

### 添加 Reactor 依赖

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
</dependency>
```

测试依赖：

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <scope>test</scope>
</dependency>
```

---

## 10.3 使用常见的反应式操作

Reactor 提供了大量操作符用于组合数据流管道，可以分为以下几类：

### 10.3.1 创建操作

```java
// 从对象创建 Flux
Flux<String> fruits = Flux.just("Apple", "Orange", "Banana");

// 从集合创建
Flux<String> fruitsFromList = Flux.fromIterable(fruitList);

// 从数组创建
Flux<String> fruitsFromArray = Flux.fromArray(fruitArray);

// 创建范围 Flux
Flux<Integer> range = Flux.range(1, 5); // 1, 2, 3, 4, 5

// 创建间隔 Flux（每秒发出一个值）
Flux<Long> interval = Flux.interval(Duration.ofSeconds(1));
```

### 10.3.2 组合操作

- **`mergeWith()`** — 将两个 Flux 合并为一个，元素按发布顺序交错排列
- **`zip()`** — 将两个 Flux 合并为一个元组（Tuple），一一配对
- **`first()` / `firstWithSignal()`** — 选择第一个发布数据的 Flux

```java
// mergeWith：按时间顺序合并
Flux<String> merged = flux1.mergeWith(flux2);

// zip：成对组合
Flux<Tuple2<String, String>> zipped = Flux.zip(flux1, flux2);

// zip 使用自定义合并函数
Flux<String> zipped = Flux.zip(flux1, flux2, (a, b) -> a + " " + b);
```

### 10.3.3 转换和过滤操作

- **`skip(n)`** — 跳过前 n 个元素
- **`skip(Duration)`** — 跳过指定时间段内发出的元素
- **`take(n)`** — 只取前 n 个元素
- **`filter(Predicate)`** — 根据条件过滤元素
- **`distinct()`** — 去除重复元素

```java
Flux<String> filtered = flux
    .skip(2)
    .filter(s -> s.startsWith("A"))
    .distinct();
```

### 10.3.4 映射操作

- **`map()`** — 同步地将每个元素映射为新的元素
- **`flatMap()`** — 异步地将每个元素映射为一个 Publisher，再将结果合并

```java
// map：同步转换
Flux<String> upper = flux.map(String::toUpperCase);

// flatMap：异步转换（可搭配 subscribeOn 实现并行）
Flux<Player> players = flux
    .flatMap(n -> Mono.just(n)
        .map(p -> new Player(p))
        .subscribeOn(Schedulers.parallel())
    );
```

> **map vs flatMap**：`map` 是同步一对一转换；`flatMap` 是异步一对多转换，结果会被"展平"合并到一个 Flux 中，适合需要并发执行的场景。
> 

### 10.3.5 缓冲操作

- **`buffer(n)`** — 将 Flux 中的元素按指定大小分组为 `List` 集合
- 配合 `flatMap` + `parallel` 可以并行处理每个缓冲批次

```java
Flux<List<String>> buffered = flux.buffer(3);

// 并行处理缓冲区
flux.buffer(3)
    .flatMap(batch ->
        Flux.fromIterable(batch)
            .map(s -> s.toUpperCase())
            .subscribeOn(Schedulers.parallel())
            .log()
    );
```

- **`collectList()`** — 将 Flux 的所有元素收集为 `Mono<List<T>>`
- **`collectMap()`** — 将 Flux 收集为 `Mono<Map<K, T>>`

### 10.3.6 逻辑操作

- **`all(Predicate)`** — 所有元素是否都满足条件，返回 `Mono<Boolean>`
- **`any(Predicate)`** — 是否有任意元素满足条件，返回 `Mono<Boolean>`

```java
Mono<Boolean> allAdult = personFlux.all(p -> p.getAge() >= 18);
Mono<Boolean> hasAdmin = userFlux.any(u -> u.getRole().equals("ADMIN"));
```

---

## 10.4 本章小结

- **反应式编程**采用声明式、函数式的方式定义数据处理管道
- **Reactive Streams** 规范定义了 Publisher、Subscriber、Subscription、Processor 四个接口
- **Project Reactor** 实现了 Reactive Streams 规范，是 Spring 5 反应式特性的基础
- **Flux** 表示零到多个元素的流，**Mono** 表示零或一个元素的流
- Reactor 提供了丰富的操作符：创建、组合、转换、过滤、缓冲、逻辑判断等
- 反应式编程的核心优势：**非阻塞**、**背压支持**、更高效地利用系统资源