---
title: 第6章 创建REST服务
date: 2026-02-17 15:10:45
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章围绕 Taco Cloud 应用，介绍如何使用 Spring MVC 创建 REST API：编写 RESTful 控制器、启用超媒体驱动的 API（HATEOAS）、以及使用 Spring Data REST 自动生成 REST 端点。

---

## 6.1 编写 RESTful 控制器

### 从服务器获取数据

在第 2 章中，控制器主要为浏览器渲染 HTML 页面。而 REST 控制器与 MVC 控制器的区别在于：

- REST 控制器不将数据填充到模型再由视图渲染，而是**直接将数据写入响应体**
- 使用 `@RestController` 注解代替 `@Controller`

<aside>
💡

**@RestController** 是一个组合注解，包含 `@Controller` 和 `@ResponseBody`。它告诉 Spring 控制器中所有处理方法的返回值都直接写入响应体，而不是被解析为视图名称。

</aside>

**请求映射注解对照：**

| **注解** | **HTTP 方法** | **典型用法** |
| --- | --- | --- |
| `@GetMapping` | GET | 读取资源数据 |
| `@PostMapping` | POST | 创建资源 |
| `@PutMapping` | PUT | 整体更新资源 |
| `@PatchMapping` | PATCH | 部分更新资源 |
| `@DeleteMapping` | DELETE | 删除资源 |

**基本的 REST 控制器示例：**

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
    public Iterable<Taco> recentTacos() {
        PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
        return tacoRepo.findAll(page).getContent();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Taco> tacoById(@PathVariable("id") Long id) {
        Optional<Taco> optTaco = tacoRepo.findById(id);
        if (optTaco.isPresent()) {
            return new ResponseEntity<>(optTaco.get(), HttpStatus.OK);
        }
        return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
    }
}
```

**关键点：**

- `produces = "application/json"`：声明该控制器只处理 Accept 头包含 `application/json` 的请求
- `@CrossOrigin(origins = "*")`：允许来自任何域的跨域请求（CORS），使前端应用可以访问 API
- `@PathVariable`：从 URL 路径中提取变量
- `ResponseEntity`：可以携带 HTTP 状态码的响应对象

### 发送数据到服务器

使用 `@PostMapping` 处理创建资源的请求，`@RequestBody` 将请求体中的 JSON 绑定到对象：

```java
@PostMapping(consumes = "application/json")
@ResponseStatus(HttpStatus.CREATED)
public Taco postTaco(@RequestBody Taco taco) {
    return tacoRepo.save(taco);
}
```

- `consumes = "application/json"`：只接受 Content-Type 为 JSON 的请求
- `@ResponseStatus(HttpStatus.CREATED)`：成功时返回 **201 Created** 而非默认的 200 OK
- `@RequestBody`：表明请求体应该被转换为 Taco 对象（通过 Jackson 自动完成 JSON → 对象的反序列化）

### 在服务器上更新数据

- **PUT**：执行**整体替换**操作——如果某个字段为 null，则该字段也会被更新为 null
- **PATCH**：执行**部分更新**操作——只更新请求中包含的字段

```java
@PutMapping("/{orderId}")
public Order putOrder(@RequestBody Order order) {
    return orderRepo.save(order);
}
```

```java
@PatchMapping(path = "/{orderId}", consumes = "application/json")
public Order patchOrder(@PathVariable("orderId") Long orderId,
                        @RequestBody Order patch) {
    Order order = orderRepo.findById(orderId).get();
    // 逐字段检查并更新
    if (patch.getName() != null) {
        order.setName(patch.getName());
    }
    if (patch.getStreet() != null) {
        order.setStreet(patch.getStreet());
    }
    // ...其他字段类似
    return orderRepo.save(order);
}
```

<aside>
📌

**PUT vs PATCH**：语义上，PUT 意味着"用这个数据完全替换该资源"，PATCH 意味着"只修改我发送的这些字段"。实际开发中 PATCH 更灵活，但实现也更复杂（需要逐字段判断）。

</aside>

### 删除服务器上的数据

```java
@DeleteMapping("/{orderId}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteOrder(@PathVariable("orderId") Long orderId) {
    try {
        orderRepo.deleteById(orderId);
    } catch (EmptyResultDataAccessException e) {
        // 资源不存在，忽略
    }
}
```

- 返回 **204 No Content**：表示操作成功但无响应体
- 捕获 `EmptyResultDataAccessException`：当要删除的资源不存在时不抛异常

---

## 6.2 启用超媒体

### 什么是 HATEOAS

- **HATEOAS**（Hypermedia As The Engine Of Application State）：超媒体作为应用状态引擎
- 核心思想：API 返回的资源中应包含**指向相关资源的链接**，客户端通过这些链接导航 API，而不是硬编码 URL
- 好处：客户端与 API 的 URL 结构**解耦**，API 的 URL 变化不会影响客户端

**普通 JSON 响应 vs HATEOAS 响应对比：**

```json
// 普通 JSON（客户端需要自己拼接 URL）
{
  "id": 4,
  "name": "Veg-Out",
  "ingredients": [...]
}
```

```json
// HATEOAS（资源自带导航链接）
{
  "name": "Veg-Out",
  "ingredients": [...],
  "_links": {
    "self": { "href": "http://localhost:8080/design/4" }
  }
}
```

### Spring HATEOAS

Spring HATEOAS 项目提供了为 Spring MVC 控制器返回的资源添加链接的支持。添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

**核心类型：**

| **类型** | **说明** |
| --- | --- |
| `RepresentationModel` | 所有资源表示的基类，携带 Link 列表 |
| `EntityModel<T>` | 单个资源的包装，包含内容对象和链接 |
| `CollectionModel<T>` | 资源集合的包装 |
| `Link` | 表示一个超媒体链接（href + rel） |
| `WebMvcLinkBuilder` | 帮助构建指向 Spring MVC 控制器的链接 |

### 添加超链接

使用 `WebMvcLinkBuilder` 根据控制器方法自动生成链接，避免硬编码 URL：

```java
@GetMapping("/recent")
public CollectionModel<EntityModel<Taco>> recentTacos() {
    PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
    List<Taco> tacos = tacoRepo.findAll(page).getContent();

    CollectionModel<EntityModel<Taco>> tacoResources =
        CollectionModel.wrap(tacos);

    tacoResources.add(
        linkTo(methodOn(DesignTacoController.class).recentTacos())
            .withRel("recents")
    );

    return tacoResources;
}
```

- `linkTo()`：根据控制器类的 `@RequestMapping` 生成基础 URL
- `methodOn()`：让你调用控制器方法来推断映射路径
- `withRel("recents")`：设置链接的关系名称
- `withSelfRel()`：创建 `self` 关系的链接

### 创建资源装配器（Resource Assembler）

当需要为资源列表中的每个元素都添加链接时，可以创建**资源装配器**来避免重复代码：

```java
public class TacoModelAssembler
    extends RepresentationModelAssemblerSupport<Taco, TacoModel> {

    public TacoModelAssembler() {
        super(DesignTacoController.class, TacoModel.class);
    }

    @Override
    protected TacoModel instantiateModel(Taco taco) {
        return new TacoModel(taco);
    }

    @Override
    public TacoModel toModel(Taco taco) {
        return createModelWithId(taco.getId(), taco);
    }
}
```

- `RepresentationModelAssemblerSupport`：基类，自动为每个资源添加 `self` 链接
- `createModelWithId()`：创建包含 `self` 链接的资源对象
- 使用装配器后，控制器代码更简洁，资源链接的逻辑可复用

### 嵌套关系中的命名

- 默认情况下，Spring HATEOAS 会使用类名作为 JSON 中嵌入资源的属性名
- 使用 `@Relation` 注解可以自定义资源在嵌入式 JSON 中的名称：

```java
@Relation(value = "taco", collectionRelation = "tacos")
public class TacoModel extends RepresentationModel<TacoModel> {
    // ...
}
```

---

## 6.3 启用数据后端服务

### Spring Data REST

Spring Data REST 可以**自动为 Spring Data 仓库（Repository）创建 REST API**，无需编写任何控制器代码。

添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```

<aside>
💡

只需添加 Spring Data REST 依赖，它就会**自动为所有 Spring Data Repository 暴露 REST 端点**，并且自动支持 HATEOAS 风格的超媒体链接。

</aside>

### 设置基础路径

为避免与自定义控制器冲突，通常为 Spring Data REST 设置统一的基础路径：

```yaml
spring:
  data:
    rest:
      base-path: /api
```

这样所有自动生成的端点都会以 `/api` 开头，例如 `/api/tacos`、`/api/orders`。

### 调整资源路径和关系名称

Spring Data REST 默认根据实体类名的复数形式生成路径。可以使用 `@RestResource` 注解自定义：

```java
@Data
@Entity
@RestResource(rel = "tacos", path = "tacos")
public class Taco {
    // ...
}
```

### 自动生成的端点

Spring Data REST 自动提供以下操作：

| **HTTP 方法** | **路径示例** | **操作** |
| --- | --- | --- |
| GET | `/api/tacos` | 获取所有 Taco（分页） |
| GET | `/api/tacos/{id}` | 获取单个 Taco |
| POST | `/api/tacos` | 创建新 Taco |
| PUT | `/api/tacos/{id}` | 整体更新 Taco |
| PATCH | `/api/tacos/{id}` | 部分更新 Taco |
| DELETE | `/api/tacos/{id}` | 删除 Taco |

### 分页和排序

Spring Data REST 自动支持分页和排序参数：

- `page`：页码（从 0 开始）
- `size`：每页大小
- `sort`：排序字段和方向

```
GET /api/tacos?size=5&page=0&sort=createdAt,desc
```

- 响应中会自动包含分页导航链接（`first`、`last`、`next`、`prev`）

### 添加自定义端点

如果自动生成的端点无法满足需求，可以编写自定义控制器进行补充。但要注意：

- 自定义控制器的端点**不会自动包含 Spring Data REST 的基础路径**
- 使用 `@RepositoryRestController` 注解可以让自定义端点继承基础路径

```java
@RepositoryRestController
public class RecentTacosController {

    private TacoRepository tacoRepo;

    public RecentTacosController(TacoRepository tacoRepo) {
        this.tacoRepo = tacoRepo;
    }

    @GetMapping(path = "/tacos/recent", produces = "application/hal+json")
    public ResponseEntity<CollectionModel<TacoModel>> recentTacos() {
        PageRequest page = PageRequest.of(0, 12, Sort.by("createdAt").descending());
        List<Taco> tacos = tacoRepo.findAll(page).getContent();

        CollectionModel<TacoModel> tacoResources =
            new TacoModelAssembler().toCollectionModel(tacos);

        return new ResponseEntity<>(tacoResources, HttpStatus.OK);
    }
}
```

<aside>
📌

`@RepositoryRestController` 不同于 `@RestController`：它不会自动给方法添加 `@ResponseBody` 语义，因此需要返回 `ResponseEntity` 或在方法上添加 `@ResponseBody`。

</aside>

---

## 6.4 小结

- **`@RestController`** 结合 `@GetMapping` / `@PostMapping` 等注解可以快速创建 RESTful 端点
- 控制器方法的返回值会通过 **Jackson** 自动序列化为 JSON 写入响应体
- 使用 `@ResponseStatus` 自定义 HTTP 状态码，使用 `ResponseEntity` 获得更灵活的响应控制
- **Spring HATEOAS** 为资源添加超媒体链接，使 API 具有自描述性，客户端与 URL 结构解耦
- **Spring Data REST** 可自动为 Spring Data Repository 暴露完整的 RESTful CRUD 端点，支持分页、排序和 HATEOAS
- 使用 `@RepositoryRestController` 可以在自动生成的端点基础上添加自定义逻辑