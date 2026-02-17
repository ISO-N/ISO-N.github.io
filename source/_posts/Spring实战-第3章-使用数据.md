---
title: 第3章 使用数据
date: 2026-02-17 15:10:42
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章主要介绍如何为 Taco Cloud 应用添加**数据持久化**功能，涵盖三种方式：JDBC、Spring Data JDBC 和 Spring Data JPA。

---

## 3.1 使用 JdbcTemplate 读取和写入数据

### 为什么不用原生 JDBC？

原生 JDBC 代码存在大量**样板代码**（创建连接、创建语句、处理结果集、关闭资源、异常处理等），业务逻辑被淹没在基础设施代码中。

### JdbcTemplate 的优势

- Spring 的 `JdbcTemplate` 封装了所有样板代码
- 开发者只需关注**查询 SQL** 和**结果映射**
- 通过 `spring-boot-starter-jdbc` 引入依赖

### 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 定义领域对象

以 Taco Cloud 为例，定义 `Ingredient`、`Taco`、`TacoOrder` 等领域类。

### 定义 Repository 接口

```java
public interface IngredientRepository {
    Iterable<Ingredient> findAll();
    Optional<Ingredient> findById(String id);
    Ingredient save(Ingredient ingredient);
}
```

### 使用 JdbcTemplate 实现 Repository

```java
@Repository
public class JdbcIngredientRepository implements IngredientRepository {

    private JdbcTemplate jdbcTemplate;

    @Autowired
    public JdbcIngredientRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public Iterable<Ingredient> findAll() {
        return jdbcTemplate.query(
            "select id, name, type from Ingredient",
            this::mapRowToIngredient);
    }

    @Override
    public Optional<Ingredient> findById(String id) {
        List<Ingredient> results = jdbcTemplate.query(
            "select id, name, type from Ingredient where id=?",
            this::mapRowToIngredient, id);
        return results.size() == 0 ?
            Optional.empty() : Optional.of(results.get(0));
    }

    private Ingredient mapRowToIngredient(ResultSet row, int rowNum)
            throws SQLException {
        return new Ingredient(
            row.getString("id"),
            row.getString("name"),
            Ingredient.Type.valueOf(row.getString("type")));
    }

    @Override
    public Ingredient save(Ingredient ingredient) {
        jdbcTemplate.update(
            "insert into Ingredient (id, name, type) values (?, ?, ?)",
            ingredient.getId(), ingredient.getName(),
            ingredient.getType().toString());
        return ingredient;
    }
}
```

### 关键注解与概念

- `@Repository`：将类声明为 Repository 组件，Spring 自动发现并注册为 Bean
- `@Autowired`：通过构造器注入 `JdbcTemplate`
- `query()`：执行查询，通过 `RowMapper` 将结果集映射为对象
- `update()`：执行插入/更新/删除操作

### 定义数据库 Schema

在 `src/main/resources/schema.sql` 中定义建表语句，Spring Boot 启动时自动执行：

```sql
create table if not exists Ingredient (
    id varchar(4) not null primary key,
    name varchar(25) not null,
    type varchar(10) not null
);

create table if not exists Taco_Order ( ... );
create table if not exists Taco ( ... );
create table if not exists Ingredient_Ref ( ... );
```

### 插入复杂数据

对于 `TacoOrder` 这种包含嵌套对象的复杂数据：

- 需要先保存 Order，获取生成的 ID
- 再保存关联的 Taco，建立外键关系
- 可使用 `PreparedStatementCreatorFactory` 和 `KeyHolder` 获取自增 ID

---

## 3.2 使用 Spring Data JDBC

### 什么是 Spring Data？

Spring Data 是一个大型项目，包含多个子项目（Spring Data JPA、Spring Data MongoDB、Spring Data Redis 等），核心能力是**自动生成 Repository 实现**。

### 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
```

### 定义 Repository 接口

只需继承 `CrudRepository` 接口，Spring Data 自动提供实现：

```java
public interface IngredientRepository extends CrudRepository<Ingredient, String> {
    // 无需编写任何方法实现！
}
```

`CrudRepository` 提供的常用方法：

- `save()`：保存实体
- `findById()`：按 ID 查找
- `findAll()`：查找所有
- `count()`：统计数量
- `delete()`：删除实体
- `existsById()`：判断是否存在

### 领域类注解

```java
@Table   // 标注实体对应的表（可省略，默认用类名）
@Id      // 标注主键字段
@Column  // 标注列映射（可省略，默认用字段名）
```

示例：

```java
@Table
public class TacoOrder implements Serializable {
    @Id
    private Long id;
    // ...其他字段
    private List<Taco> tacos = new ArrayList<>();
}
```

### 与 JdbcTemplate 对比

|  | JdbcTemplate | Spring Data JDBC |
| --- | --- | --- |
| SQL | 手动编写 | 自动生成 |
| 映射 | 手动 RowMapper | 自动映射 |
| 代码量 | 较多 | 极少 |
| 灵活性 | 高 | 中等 |

---

## 3.3 使用 Spring Data JPA 持久化数据

### JPA 简介

**JPA（Java Persistence API）** 是 Java 的 ORM 标准规范，Hibernate 是最常用的实现。Spring Data JPA 在 JPA 之上提供了更高级的抽象。

### 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

> 引入此依赖后替换 `spring-boot-starter-jdbc` 或 `spring-boot-starter-data-jdbc`。
> 

### 将领域对象标注为 JPA 实体

```java
@Data
@Entity
public class Ingredient {
    @Id
    private String id;
    private String name;

    @Enumerated(EnumType.STRING)
    private Type type;

    public enum Type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```

```java
@Data
@Entity
public class TacoOrder implements Serializable {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    // ...delivery and payment fields

    @OneToMany(cascade = CascadeType.ALL)
    private List<Taco> tacos = new ArrayList<>();
}
```

```java
@Data
@Entity
public class Taco {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;
    private Date createdAt;

    @ManyToMany
    @Size(min = 1, message = "至少选择1种配料")
    private List<Ingredient> ingredients = new ArrayList<>();

    @PrePersist
    void createdAt() {
        this.createdAt = new Date();
    }
}
```

### JPA 核心注解

| 注解 | 作用 |
| --- | --- |
| `@Entity` | 声明为 JPA 实体 |
| `@Id` | 标注主键 |
| `@GeneratedValue` | 主键自动生成策略 |
| `@Table` | 指定映射的表名 |
| `@Column` | 指定映射的列名 |
| `@OneToMany` | 一对多关系 |
| `@ManyToMany` | 多对多关系 |
| `@ManyToOne` | 多对一关系 |
| `@PrePersist` | 持久化前的回调方法 |
| `@Enumerated` | 枚举类型映射方式 |

### 声明 JPA Repository

```java
public interface IngredientRepository extends CrudRepository<Ingredient, String> {
}

public interface OrderRepository extends CrudRepository<TacoOrder, Long> {
    List<TacoOrder> findByDeliveryZip(String deliveryZip);
}
```

### 自定义查询方法

Spring Data JPA 根据**方法命名规则**自动生成查询：

```java
// 根据 zip 和日期范围查找，按 deliveryName 排序
List<TacoOrder> findByDeliveryZipAndPlacedAtBetweenOrderByDeliveryName(
    String deliveryZip, Date startDate, Date endDate);
```

命名规则关键字：

| 关键字 | 含义 | 示例 |
| --- | --- | --- |
| `findBy` | 查询 | `findByName(String name)` |
| `And` / `Or` | 与 / 或 | `findByNameAndType(...)` |
| `Between` | 范围 | `findByDateBetween(Date s, Date e)` |
| `LessThan` / `GreaterThan` | 比较 | `findByAgeLessThan(int age)` |
| `IsNull` / `IsNotNull` | 空判断 | `findByNameIsNull()` |
| `Like` / `Contains` | 模糊查询 | `findByNameContains(String s)` |
| `OrderBy` | 排序 | `findByZipOrderByName(...)` |
| `IgnoreCase` | 忽略大小写 | `findByNameIgnoreCase(String n)` |

### 使用 @Query 自定义查询

当方法命名太复杂时，使用 `@Query` 注解：

```java
@Query("SELECT o FROM TacoOrder o WHERE o.deliveryCity = 'Seattle'")
List<TacoOrder> readOrdersDeliveredInSeattle();
```

---

## 本章小结

<aside>
📌

- Spring 提供了 `JdbcTemplate` 简化 JDBC 操作，消除样板代码
- Spring Data JDBC 和 Spring Data JPA 通过继承 `CrudRepository` 自动生成 Repository 实现，进一步减少代码
- **JdbcTemplate** 适合需要完全控制 SQL 的场景
- **Spring Data JDBC** 适合简单直接的数据访问，不需要 JPA 的复杂特性
- **Spring Data JPA** 适合需要复杂对象关系映射（ORM）的场景
- Spring Data JPA 支持通过**方法命名规则**和 `@Query` 注解自动生成查询
</aside>