---
title: 第11章 开发反应式API
date: 2026-02-17 15:10:49
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章围绕 Spring 5 的反应式 Web 框架 **Spring WebFlux**，介绍如何编写反应式控制器、定义函数式请求处理器、测试反应式控制器、使用 WebClient 消费 REST API，以及保护反应式 Web 应用。

---

## 11.1 使用 Spring WebFlux

### 11.1.1 Spring WebFlux 简介

Spring 5 引入了一个与 Spring MVC **并行**的新 Web 框架 —— **Spring WebFlux**，它基于反应式编程模型构建，完全非阻塞。

**Spring MVC vs Spring WebFlux：**

| **对比项** | **Spring MVC** | **Spring WebFlux** |
| --- | --- | --- |
| 编程模型 | 命令式（阻塞） | 反应式（非阻塞） |
| 底层容器 | Servlet 容器（Tomcat、Jetty） | Netty、Servlet 3.1+（非阻塞模式） |
| 线程模型 | 每个请求占用一个线程 | 事件轮循（Event Loop），少量线程处理大量请求 |
| 返回类型 | 普通对象、`ResponseEntity` | `Mono<T>`、`Flux<T>` |
| 适用场景 | 传统 Web 应用 | 高并发、I/O 密集型应用 |

<aside>
💡

**事件轮循（Event Loop）机制**：WebFlux 使用少量线程（通常等于 CPU 核心数），通过事件轮循处理请求。当遇到 I/O 操作时不会阻塞线程，而是注册回调，线程可以去处理其他请求。I/O 完成后再触发回调继续处理。这使得少量线程就能支撑大量并发请求。

</aside>

**Spring WebFlux 与 Spring MVC 共享的特性：**

- 都支持基于注解的编程模型（`@RestController`、`@GetMapping` 等）
- Spring WebFlux 额外支持**函数式编程模型**（使用 `RouterFunction` 定义路由）
- 已有的 Spring MVC 知识可以直接迁移到 WebFlux

### 11.1.2 编写反应式控制器

反应式控制器的写法与 Spring MVC 非常相似，关键区别在于**返回类型使用 `Flux` 或 `Mono` 包装**：

```java
@RestController
@RequestMapping(path = "/design", produces = "application/json")
@CrossOrigin(origins = "*")
public class DesignTacoController {

    private TacoRepository tacoRepo;

    @Autowired
    public DesignTacoController(TacoRepository tacoRepo) {
        this.tacoRepo = tacoRepo;
    }

    @GetMapping("/recent")
    public Flux<Taco> recentTacos() {
        return tacoRepo.findAll().take(12);
    }

    @GetMapping("/{id}")
    public Mono<Taco> tacoById(@PathVariable("id") Long id) {
        return tacoRepo.findById(id);
    }

    @PostMapping(consumes = "application/json")
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<Taco> postTaco(@RequestBody Mono<Taco> tacoMono) {
        return tacoRepo.saveAll(tacoMono).next();
    }
}
```

**关键点：**

- `Flux<Taco>`：返回零到多个 Taco 的反应式流
- `Mono<Taco>`：返回零或一个 Taco 的反应式流
- `@RequestBody Mono<Taco>`：请求体本身也可以作为反应式类型接收，实现**端到端非阻塞**——从接收请求到保存数据全程不阻塞线程
- `take(12)`：使用 Reactor 操作符限制返回数量，替代了 MVC 中的分页查询

<aside>
📌

**输入也可以是反应式的**：在 Spring MVC 中，`@RequestBody` 接收的是普通对象（框架会阻塞等待请求体完全到达）。在 WebFlux 中，可以接收 `Mono<T>` 或 `Flux<T>`，这意味着框架不需要等待请求体完全到达就可以开始处理，实现了输入到输出的完整反应式管道。

</aside>

---

## 11.2 定义函数式请求处理器

除了基于注解的编程模型，Spring WebFlux 还提供了**函数式编程模型**，使用以下核心类型定义路由和处理逻辑：

| **类型** | **说明** |
| --- | --- |
| `RouterFunction` | 声明式地定义路由规则（替代 `@RequestMapping`） |
| `ServerRequest` | 代表 HTTP 请求（替代 `@RequestBody`、`@PathVariable` 等注解的功能） |
| `ServerResponse` | 代表 HTTP 响应（替代 `ResponseEntity`） |
| `HandlerFunction` | 接收 `ServerRequest` 并返回 `Mono<ServerResponse>` 的函数 |

**示例：用函数式方式定义路由**

```java
@Configuration
public class RouterFunctionConfig {

    @Bean
    public RouterFunction<?> routerFunction(TacoRepository tacoRepo) {
        return route(GET("/design/recent"), request -> {
            return ServerResponse.ok()
                .body(tacoRepo.findAll().take(12), Taco.class);
        })
        .andRoute(POST("/design"), request -> {
            Mono<Taco> taco = request.bodyToMono(Taco.class);
            Mono<Taco> saved = tacoRepo.saveAll(taco).next();
            return ServerResponse.created(URI.create("http://localhost:8080/design/"))
                .body(saved, Taco.class);
        });
    }
}
```

**关键点：**

- `route(RequestPredicate, HandlerFunction)`：定义一个路由规则
- `andRoute()`：链式添加更多路由
- `request.bodyToMono(Taco.class)`：从请求体提取反应式类型
- `ServerResponse.ok().body(...)`：构建响应
- 函数式模型将路由定义与处理逻辑**集中在一处**，不需要分散在多个控制器类的注解中

<aside>
💡

**注解模型 vs 函数式模型**：两者功能等价，选择哪种取决于个人偏好。注解模型更适合熟悉 Spring MVC 的开发者；函数式模型更贴近函数式编程风格，路由逻辑集中管理。实际项目中两者可以混用。

</aside>

---

## 11.3 测试反应式控制器

Spring 5 提供了 **`WebTestClient`**，用于测试 WebFlux 控制器，无需启动真正的服务器。

### 11.3.1 测试 GET 请求

```java
@Test
public void shouldReturnRecentTacos() {
    Taco[] tacos = { testTaco(1L), testTaco(2L), testTaco(3L) };
    Flux<Taco> tacoFlux = Flux.just(tacos);

    when(tacoRepo.findAll()).thenReturn(tacoFlux);

    WebTestClient testClient = WebTestClient
        .bindToController(new DesignTacoController(tacoRepo))
        .build();

    testClient.get().uri("/design/recent")
        .exchange()
        .expectStatus().isOk()
        .expectBody()
        .jsonPath("$").isArray()
        .jsonPath("$[0].id").isEqualTo(tacos[0].getId().toString());
}
```

**关键 API：**

- `WebTestClient.bindToController()`：绑定到特定控制器进行测试（不启动服务器）
- `.get().uri()`：构造 GET 请求
- `.exchange()`：发送请求
- `.expectStatus()`：断言响应状态码
- `.expectBody().jsonPath()`：使用 JSONPath 表达式断言响应体

### 11.3.2 测试 POST 请求

```java
@Test
public void shouldSaveATaco() {
    Mono<Taco> unsavedTacoMono = Mono.just(testTaco(null));
    Taco savedTaco = testTaco(1L);
    Flux<Taco> savedTacoFlux = Flux.just(savedTaco);

    when(tacoRepo.saveAll(any(Mono.class))).thenReturn(savedTacoFlux);

    testClient.post().uri("/design")
        .contentType(MediaType.APPLICATION_JSON)
        .body(unsavedTacoMono, Taco.class)
        .exchange()
        .expectStatus().isCreated()
        .expectBody(Taco.class)
        .isEqualTo(savedTaco);
}
```

### 11.3.3 使用实时服务器进行测试

也可以绑定到一个运行中的服务器进行集成测试：

```java
WebTestClient testClient = WebTestClient
    .bindToServer()
    .baseUrl("http://localhost:8080")
    .build();
```

- `bindToServer()`：绑定到实际运行的服务器
- 适用于完整的端到端集成测试

---

## 11.4 反应式消费 REST API

Spring WebFlux 提供了 **`WebClient`** 作为 `RestTemplate` 的反应式替代方案。`WebClient` 完全非阻塞，使用 `Mono` 和 `Flux` 作为返回类型。

### 11.4.1 获取资源

```java
// 获取单个资源
Mono<Ingredient> ingredientMono = WebClient.create()
    .get()
    .uri("{{http://localhost:8080/ingredients/{id}}}", ingredientId)
    .retrieve()
    .bodyToMono(Ingredient.class);

ingredientMono.subscribe(ingredient -> {
    // 处理数据
});

// 获取集合资源
Flux<Ingredient> ingredientFlux = WebClient.create()
    .get()
    .uri("http://localhost:8080/ingredients")
    .retrieve()
    .bodyToFlux(Ingredient.class);

ingredientFlux.subscribe(ingredient -> {
    // 处理每个元素
});
```

- `WebClient.create()`：创建 WebClient 实例
- `.retrieve()`：发送请求并获取响应
- `.bodyToMono()` / `.bodyToFlux()`：将响应体转换为反应式类型
- `.subscribe()`：触发实际的请求发送（反应式流是惰性的，不订阅就不会执行）

也可以使用**基础 URL** 简化多次调用：

```java
WebClient client = WebClient.create("http://localhost:8080");

Mono<Ingredient> ingredientMono = client
    .get()
    .uri("/ingredients/{id}", ingredientId)
    .retrieve()
    .bodyToMono(Ingredient.class);
```

### 11.4.2 发送资源

```java
Mono<Ingredient> ingredientMono = Mono.just(ingredient);

Mono<Ingredient> result = WebClient.create()
    .post()
    .uri("http://localhost:8080/ingredients")
    .body(ingredientMono, Ingredient.class)
    .retrieve()
    .bodyToMono(Ingredient.class);

result.subscribe(saved -> {
    // 处理保存后的结果
});
```

### 11.4.3 删除资源

```java
Mono<Void> result = WebClient.create()
    .delete()
    .uri("{{http://localhost:8080/ingredients/{id}}}", ingredientId)
    .retrieve()
    .bodyToMono(Void.class);

result.subscribe();
```

### 11.4.4 处理错误

默认情况下，4xx 和 5xx 状态码会抛出 `WebClientResponseException`。可以使用 `onStatus()` 自定义错误处理：

```java
Mono<Ingredient> ingredientMono = WebClient.create()
    .get()
    .uri("{{http://localhost:8080/ingredients/{id}}}", ingredientId)
    .retrieve()
    .onStatus(HttpStatus::is4xxClientError,
        response -> Mono.just(new UnknownIngredientException()))
    .onStatus(HttpStatus::is5xxServerError,
        response -> Mono.just(new ServerErrorException()))
    .bodyToMono(Ingredient.class);
```

- `onStatus(Predicate, Function)`：根据状态码条件返回自定义异常

### 11.4.5 交换请求

如果需要访问**响应头**或 **Cookie** 等信息，可以使用 `exchangeToMono()` / `exchangeToFlux()` 代替 `retrieve()`：

```java
Mono<Ingredient> ingredientMono = WebClient.create()
    .get()
    .uri("{{http://localhost:8080/ingredients/{id}}}", ingredientId)
    .exchangeToMono(response -> {
        if (response.statusCode().equals(HttpStatus.OK)) {
            return response.bodyToMono(Ingredient.class);
        } else {
            return response.createException().flatMap(Mono::error);
        }
    });
```

- `exchangeToMono()`：提供对完整 `ClientResponse` 的访问，包括状态码、响应头、Cookie 等
- 比 `retrieve()` 更灵活，但代码也更复杂

<aside>
📌

**retrieve() vs exchangeToMono()**：大多数场景使用 `retrieve()` 就够了，它更简洁。只有在需要根据响应状态码做不同处理，或者需要访问响应头信息时，才使用 `exchangeToMono()` / `exchangeToFlux()`。

</aside>

---

## 11.5 保护反应式 Web API

Spring Security 对 WebFlux 应用提供了反应式安全支持，配置方式与 Servlet 应用类似但有一些区别。

### 11.5.1 配置反应式 Web 应用的安全性

在 WebFlux 中，安全配置不再继承 `WebSecurityConfigurerAdapter`，而是声明一个 `SecurityWebFilterChain` 类型的 Bean：

```java
@Configuration
@EnableWebFluxSecurity
public class SecurityConfig {

    @Bean
    public SecurityWebFilterChain securityWebFilterChain(
            ServerHttpSecurity http) {
        return http
            .authorizeExchange()
                .pathMatchers("/design", "/orders").hasAuthority("USER")
                .anyExchange().permitAll()
            .and()
            .build();
    }
}
```

**Spring MVC Security vs WebFlux Security 对照：**

| **Spring MVC Security** | **WebFlux Security** |
| --- | --- |
| `@EnableWebSecurity` | `@EnableWebFluxSecurity` |
| 继承 `WebSecurityConfigurerAdapter` | 声明 `SecurityWebFilterChain` Bean |
| `authorizeRequests()` | `authorizeExchange()` |
| `antMatchers()` | `pathMatchers()` |
| `HttpSecurity` | `ServerHttpSecurity` |

### 11.5.2 配置反应式的用户详情服务

在反应式安全中，使用 `ReactiveUserDetailsService` 替代 `UserDetailsService`：

```java
@Bean
public ReactiveUserDetailsService userDetailsService(
        UserRepository userRepo) {
    return new ReactiveUserDetailsService() {
        @Override
        public Mono<UserDetails> findByUsername(String username) {
            return userRepo.findByUsername(username)
                .map(user -> {
                    return user.toUserDetails();
                });
        }
    };
}
```

- `ReactiveUserDetailsService`：`findByUsername()` 返回 `Mono<UserDetails>` 而非 `UserDetails`
- 这意味着用户查找过程也是非阻塞的，与整个反应式管道保持一致

---

## 11.6 小结

- **Spring WebFlux** 是 Spring 5 引入的反应式 Web 框架，基于 Project Reactor，使用事件轮循模型实现非阻塞处理
- WebFlux 支持两种编程模型：**基于注解**（与 Spring MVC 写法相似）和**函数式**（使用 `RouterFunction`）
- 反应式控制器的返回类型使用 **`Flux`** 和 **`Mono`** 包装，输入参数也可以是反应式类型，实现端到端非阻塞
- **`WebTestClient`** 用于测试反应式控制器，支持绑定控制器或实时服务器两种方式
- **`WebClient`** 是 `RestTemplate` 的反应式替代，使用 `retrieve()` 或 `exchangeToMono()` 发送请求并以反应式类型接收响应
- WebFlux 应用的安全配置使用 `@EnableWebFluxSecurity` 和 `SecurityWebFilterChain`，用户详情服务使用 `ReactiveUserDetailsService`