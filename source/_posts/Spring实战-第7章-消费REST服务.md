---
title: 第7章 消费REST服务
date: 2026-02-17 15:10:46
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
## 本章概要

- 使用 `RestTemplate` 消费 REST API
- 使用 Traverson 导航超媒体 API
- 使用 `WebClient` 消费 REST API（反应式方式）

---

## 7.1 使用 RestTemplate

- Spring 提供 `RestTemplate`，简化了客户端 HTTP 请求的发送
- 如果没有 RestTemplate，我们需要手动创建 HTTP 连接、发送请求、解析响应、处理异常、关闭连接等，代码非常繁琐
- RestTemplate 提供了 **12 个核心方法**，每个方法都有多个重载形式

### 核心方法一览

| **方法** | **描述** |
| --- | --- |
| `getForObject()` | 发送 GET 请求，将响应体映射为对象 |
| `getForEntity()` | 发送 GET 请求，返回 `ResponseEntity`（包含响应体、状态码、头信息） |
| `postForObject()` | 发送 POST 请求，将响应体映射为对象 |
| `postForEntity()` | 发送 POST 请求，返回 `ResponseEntity` |
| `postForLocation()` | 发送 POST 请求，返回新创建资源的 URI |
| `put()` | 发送 PUT 请求 |
| `patchForObject()` | 发送 PATCH 请求，将响应体映射为对象 |
| `delete()` | 发送 DELETE 请求 |
| `headForHeaders()` | 发送 HEAD 请求，返回头信息 |
| `optionsForAllow()` | 发送 OPTIONS 请求，返回 Allow 头信息 |
| `exchange()` | 发送任意 HTTP 请求，返回 `ResponseEntity`，可灵活设置请求头 |
| `execute()` | 发送任意 HTTP 请求，通过回调接口处理请求和响应 |

> 除了 `execute()` 之外，这些方法大致对应了 HTTP 标准方法：GET、POST、PUT、PATCH、DELETE、HEAD、OPTIONS。
> 

### 创建 RestTemplate 实例

```java
// 方式一：直接实例化
RestTemplate rest = new RestTemplate();

// 方式二：声明为 Bean（推荐，便于注入）
@Bean
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

### GET 请求

```java
// getForObject —— 直接获取对象
public Ingredient getIngredientById(String ingredientId) {
    return rest.getForObject(
        "http://localhost:8080/ingredients/{id}",
        Ingredient.class,
        ingredientId  // 替换 {id}
    );
}

// getForEntity —— 获取 ResponseEntity（包含更多元数据）
public Ingredient getIngredientById(String ingredientId) {
    ResponseEntity<Ingredient> responseEntity =
        rest.getForEntity(
            "http://localhost:8080/ingredients/{id}",
            Ingredient.class,
            ingredientId
        );
    log.info("Fetched time: {}",
        responseEntity.getHeaders().getDate());
    return responseEntity.getBody();
}
```

- **URI 参数替换**有两种方式：
    - **可变参数（varargs）**：按顺序替换占位符
    - **Map 参数**：通过 key 匹配占位符名称

```java
// 使用 Map 替换 URI 参数
Map<String, String> urlVariables = new HashMap<>();
urlVariables.put("id", ingredientId);
return rest.getForObject(
    "http://localhost:8080/ingredients/{id}",
    Ingredient.class,
    urlVariables
);
```

### PUT 请求

```java
public void updateIngredient(Ingredient ingredient) {
    rest.put(
        "http://localhost:8080/ingredients/{id}",
        ingredient,       // 请求体
        ingredient.getId()
    );
}
```

- `put()` 方法无返回值（`void`），发送后不获取响应体

### POST 请求

```java
// postForObject —— 返回新创建的资源对象
public Ingredient createIngredient(Ingredient ingredient) {
    return rest.postForObject(
        "http://localhost:8080/ingredients",
        ingredient,
        Ingredient.class
    );
}

// postForLocation —— 返回新资源的 URI
public URI createIngredient(Ingredient ingredient) {
    return rest.postForLocation(
        "http://localhost:8080/ingredients",
        ingredient
    );
}

// postForEntity —— 返回 ResponseEntity（同时获取对象和元数据）
public Ingredient createIngredient(Ingredient ingredient) {
    ResponseEntity<Ingredient> responseEntity =
        rest.postForEntity(
            "http://localhost:8080/ingredients",
            ingredient,
            Ingredient.class
        );
    log.info("New resource created at {}",
        responseEntity.getHeaders().getLocation());
    return responseEntity.getBody();
}
```

### DELETE 请求

```java
public void deleteIngredient(Ingredient ingredient) {
    rest.delete(
        "http://localhost:8080/ingredients/{id}",
        ingredient.getId()
    );
}
```

---

## 7.2 使用 Traverson 导航 REST API

- **Traverson** 来自 Spring Data HATEOAS，用于消费 **超媒体驱动（Hypermedia）** 的 REST API
- 名称灵感来自"Traverse on"，即"在链接关系上遍历"
- Traverson 能够理解 **HAL（Hypertext Application Language）** 格式的响应，自动跟随链接

### 创建 Traverson 实例

```java
Traverson traverson = new Traverson(
    URI.create("http://localhost:8080/api"),
    MediaTypes.HAL_JSON  // 指定接受 HAL+JSON
);
```

### 跟随链接获取资源

```java
// 跟随 "ingredients" 链接，获取集合资源
ParameterizedTypeReference<Resources<Ingredient>> ingredientType =
    new ParameterizedTypeReference<Resources<Ingredient>>() {};

Resources<Ingredient> ingredientRes =
    traverson
        .follow("ingredients")   // 跟随名为 "ingredients" 的链接
        .toObject(ingredientType);

Collection<Ingredient> ingredients = ingredientRes.getContent();
```

### 多级导航

```java
// 跟随多级链接：先到 "tacos"，再到 "recents"
ParameterizedTypeReference<Resources<Taco>> tacoType =
    new ParameterizedTypeReference<Resources<Taco>>() {};

Resources<Taco> tacoRes =
    traverson
        .follow("tacos", "recents")  // 依次跟随两级链接
        .toObject(tacoType);
```

### Traverson 的局限

- Traverson 擅长 **导航和读取** 超媒体 API
- 但 **不支持写入或删除资源**（无 PUT / POST / DELETE 能力）
- 如果需要写入，需要将 Traverson 和 RestTemplate **结合使用**

```java
// 先用 Traverson 导航获取链接，再用 RestTemplate 写入
String ingredientsUrl =
    traverson
        .follow("ingredients")
        .asLink()
        .getHref();

rest.postForObject(
    ingredientsUrl,
    ingredient,
    Ingredient.class
);
```

---

## 7.3 使用 WebClient 消费 REST API

- `WebClient` 是 Spring WebFlux 提供的 **反应式** HTTP 客户端
- 与 RestTemplate 不同，WebClient 是 **非阻塞** 的，支持反应式流
- 需要引入 **Spring WebFlux** 依赖

### 创建 WebClient 实例

```java
// 方式一：直接创建
WebClient client = WebClient.create("http://localhost:8080");

// 方式二：通过 Bean 注入（推荐）
@Bean
public WebClient webClient() {
    return WebClient.create("http://localhost:8080");
}
```

### GET 请求

```java
// 获取单个资源
Mono<Ingredient> ingredient = client
    .get()
    .uri("/ingredients/{id}", ingredientId)
    .retrieve()
    .bodyToMono(Ingredient.class);

// 获取集合资源
Flux<Ingredient> ingredients = client
    .get()
    .uri("/ingredients")
    .retrieve()
    .bodyToFlux(Ingredient.class);
```

- `bodyToMono()`：将响应体转为 `Mono<T>`（单个对象）
- `bodyToFlux()`：将响应体转为 `Flux<T>`（集合/流）
- 调用 `.subscribe()` 后才真正发起请求（**惰性求值**）

```java
// 订阅后请求才会发出
ingredient.subscribe(i -> {
    // 处理响应数据
});
```

### POST 请求

```java
Mono<Ingredient> result = client
    .post()
    .uri("/ingredients")
    .body(Mono.just(ingredient), Ingredient.class)
    .retrieve()
    .bodyToMono(Ingredient.class);

result.subscribe(i -> { /* ... */ });
```

- 如果已有现成对象而非 `Mono`，可以使用 `syncBody()`（或新版本中的 `bodyValue()`）：

```java
Mono<Ingredient> result = client
    .post()
    .uri("/ingredients")
    .bodyValue(ingredient)
    .retrieve()
    .bodyToMono(Ingredient.class);
```

### PUT 请求

```java
Mono<Void> result = client
    .put()
    .uri("/ingredients/{id}", ingredient.getId())
    .bodyValue(ingredient)
    .retrieve()
    .bodyToMono(Void.class);

result.subscribe();
```

### DELETE 请求

```java
Mono<Void> result = client
    .delete()
    .uri("/ingredients/{id}", ingredientId)
    .retrieve()
    .bodyToMono(Void.class);

result.subscribe();
```

### 错误处理

```java
Mono<Ingredient> ingredient = client
    .get()
    .uri("/ingredients/{id}", ingredientId)
    .retrieve()
    .onStatus(HttpStatus::is4xxClientError,
        response -> Mono.just(new UnknownIngredientException()))
    .onStatus(HttpStatus::is5xxServerError,
        response -> Mono.just(new ServerException()))
    .bodyToMono(Ingredient.class);
```

### exchange() 方法

- `exchange()` 类似 `retrieve()`，但提供对 `ClientResponse` 的完全控制（可访问响应头、状态码等）

```java
Mono<Ingredient> ingredient = client
    .get()
    .uri("/ingredients/{id}", ingredientId)
    .exchange()
    .flatMap(cr -> {
        if (cr.headers().header("X_UNAVAILABLE").contains("true")) {
            return Mono.empty();
        }
        return Mono.just(cr);
    })
    .flatMap(cr -> cr.bodyToMono(Ingredient.class));
```

---

<aside>
💡

**小结**：Spring 提供了多种消费 REST API 的方式。**RestTemplate** 适合传统同步调用场景，简单直接；**Traverson** 专门针对超媒体 API 的导航读取；**WebClient** 是反应式的首选方案，非阻塞且更灵活。在新项目中，Spring 官方推荐优先使用 WebClient。

</aside>