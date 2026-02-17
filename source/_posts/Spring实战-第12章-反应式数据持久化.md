---
title: 第12章 反应式数据持久化
date: 2026-02-17 15:10:49
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
## 12.1 理解 Spring Data 的反应式故事

- 从 Spring Data Kay 版本开始，Spring Data 支持反应式持久化
- 并非所有数据库都支持反应式，关系型数据库（如 JDBC）天生是阻塞的
- 目前原生支持反应式的 Spring Data 模块：
    - **Spring Data MongoDB**（反应式）
    - **Spring Data Cassandra**（反应式）
    - **Spring Data Redis**（反应式）

### Spring Data 反应式精髓

- 反应式 Repository 的关键区别：方法返回的不是普通对象或集合，而是 `Mono` / `Flux`
- 例如普通 Repository 返回 `List<Ingredient>`，反应式 Repository 返回 `Flux<Ingredient>`

```java
// 非反应式
Ingredient findById(String id);
Iterable<Ingredient> findAll();

// 反应式
Mono<Ingredient> findById(String id);
Flux<Ingredient> findAll();
```

### 反应式与非反应式类型的转换

- 如果必须使用不支持反应式的数据库，也可以在 **Service 层**做非反应式到反应式的桥接
- 使用 `Mono.just()` 或 `Flux.fromIterable()` 将阻塞结果包装为反应式类型

```java
// 将阻塞调用的结果包装成 Flux
Flux<Ingredient> ingredientFlux =
    Flux.fromIterable(nonReactiveRepo.findAll());
```

> ⚠️ 这种方式**并不能**让数据库调用变成非阻塞的，只是让接口统一为反应式类型。真正的非阻塞需要数据库驱动本身支持反应式。
> 

### 开发反应式 Repository

- 反应式 Repository 扩展 `ReactiveCrudRepository` 而非 `CrudRepository`

```java
public interface IngredientRepository
    extends ReactiveCrudRepository<Ingredient, String> {
}
```

---

## 12.2 使用反应式 Cassandra Repository

### 12.2.1 启用 Spring Data Cassandra

添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-cassandra-reactive</artifactId>
</dependency>
```

基本配置（`application.yml`）：

```yaml
spring:
  data:
    cassandra:
      keyspace-name: tacocloud
      schema-action: recreate
      contact-points:
        - localhost
      port: 9042
      local-datacenter: datacenter1
```

### 12.2.2 理解 Cassandra 数据建模

Cassandra 与关系型数据库的核心区别：

| 特点 | 关系型数据库 | Cassandra |
| --- | --- | --- |
| 数据结构 | 表、行、列 | 表、行、列（但含义不同） |
| 查询方式 | 灵活查询，支持 JOIN | **不支持 JOIN**，查询模式驱动表设计 |
| 数据冗余 | 追求最小化（范式化） | 鼓励**反范式化**，数据冗余是常态 |
| 分区 | 通常单节点 | 数据跨多个节点**分区存储** |

Cassandra 建模原则：

- **分区键（Partition Key）**：决定数据存储在哪个节点
- **集群键（Clustering Key）**：决定分区内数据的排序顺序
- **表的设计由查询需求驱动**，而非实体关系驱动
- 允许并鼓励数据冗余（denormalization）

### 12.2.3 映射领域类型到 Cassandra

```java
@Data
@Table("ingredients")  // 映射到 Cassandra 的 ingredients 表
public class Ingredient {

    @PrimaryKey
    private String id;

    private String name;
    private Type type;

    public enum Type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```

对于复合主键的情况，使用 `@PrimaryKeyClass`：

```java
@Data
@PrimaryKeyClass
public class TacoKey implements Serializable {

    @PrimaryKeyColumn(type = PrimaryKeyType.PARTITIONED)
    private UUID id;

    @PrimaryKeyColumn(type = PrimaryKeyType.CLUSTERED,
                       ordering = Ordering.DESCENDING)
    private Date createdAt;
}
```

```java
@Data
@Table("tacos")
public class Taco {

    @PrimaryKey
    private TacoKey key;

    @Column("ingredients")  // 将 List 存储为 Cassandra 集合列
    private List<IngredientUDT> ingredients;
}
```

### 用户自定义类型（UDT）

- Cassandra 不支持 JOIN，所以关联数据通常以 **UDT（User Defined Type）**的形式内嵌
- 使用 `@UserDefinedType` 注解标记

```java
@Data
@UserDefinedType("ingredient")
public class IngredientUDT {
    private String name;
    private Ingredient.Type type;
}
```

### 12.2.4 编写反应式 Cassandra Repository

```java
public interface IngredientRepository
    extends ReactiveCrudRepository<Ingredient, String> {
}
```

```java
public interface TacoRepository
    extends ReactiveCrudRepository<Taco, UUID> {
}
```

`ReactiveCrudRepository` 提供的常用方法：

- `Flux<T> findAll()` — 查询所有
- `Mono<T> findById(ID id)` — 按 ID 查询
- `Mono<T> save(T entity)` — 保存
- `Mono<Void> deleteById(ID id)` — 按 ID 删除
- `Mono<Long> count()` — 计数

也可以自定义查询方法：

```java
Flux<Taco> findByKeyId(UUID id);
```

---

## 12.3 编写反应式 MongoDB Repository

### 12.3.1 启用 Spring Data MongoDB

添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
</dependency>
```

基本配置（`application.yml`）：

```yaml
spring:
  data:
    mongodb:
      host: localhost
      port: 27017
      database: tacocloud
```

> 开发时可以使用内嵌 MongoDB（Flapdoodle）进行测试，添加 `de.flapdoodle.embed.mongo` 依赖即可。
> 

### 12.3.2 将领域类型映射为文档

```java
@Data
@Document(collection = "ingredients")  // 映射到 MongoDB 的 ingredients 集合
public class Ingredient {

    @Id
    private String id;

    private String name;
    private Type type;

    public enum Type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```

```java
@Data
@Document
public class Taco {

    @Id
    private String id;

    private String name;
    private Date createdAt;
    private List<Ingredient> ingredients;
}
```

```java
@Data
@Document
public class TacoOrder {

    @Id
    private String id;

    private Date placedAt;
    private User user;
    private List<Taco> tacos = new ArrayList<>();

    // delivery 和 payment 信息直接作为文档字段
    private String deliveryName;
    private String deliveryStreet;
    private String deliveryCity;
    private String ccNumber;
}
```

MongoDB 与 Cassandra/JPA 映射注解对比：

| 用途 | JPA | Cassandra | MongoDB |
| --- | --- | --- | --- |
| 实体/文档标记 | `@Entity` | `@Table` | `@Document` |
| 主键 | `@Id` (javax.persistence) | `@PrimaryKey` | `@Id` (spring data) |
| 列/字段映射 | `@Column` | `@Column` | `@Field`（可选） |

### 12.3.3 编写反应式 MongoDB Repository 接口

```java
public interface IngredientRepository
    extends ReactiveCrudRepository<Ingredient, String> {
}
```

```java
public interface TacoRepository
    extends ReactiveCrudRepository<Taco, String> {

    // 自定义查询：查找最近创建的 12 个 Taco
    Flux<Taco> findByOrderByCreatedAtDesc();
}
```

```java
public interface OrderRepository
    extends ReactiveCrudRepository<TacoOrder, String> {
}
```

```java
public interface UserRepository
    extends ReactiveCrudRepository<User, String> {

    Mono<User> findByUsername(String username);
}
```

---

## 12.4 本章小结

- Spring Data 支持对 **Cassandra**、**MongoDB**、**Redis** 等 NoSQL 数据库的反应式持久化
- 反应式 Repository 扩展 **`ReactiveCrudRepository`**，返回 `Mono` 和 `Flux` 类型
- **Cassandra** 建模需要理解分区键、集群键和反范式化设计，使用 UDT 替代 JOIN
- **MongoDB** 将领域对象映射为文档，使用 `@Document`、`@Id` 等注解
- 对于不支持反应式的数据库（如关系型数据库），可以在 Service 层用 `Mono.just()` 和 `Flux.fromIterable()` 做桥接，但底层仍然是阻塞调用
- 选择反应式持久化的关键前提：**数据库驱动本身必须支持非阻塞 I/O**