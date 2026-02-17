---
title: 附录 类型处理器（TypeHandler）
date: 2026-02-17 15:09:44
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
本附录列出了 MyBatis 系统提供的全部**类型处理器**，这些处理器在 MyBatis 初始化时自动注册，负责 **Java 类型**与 **JDBC 类型**之间的双向转换。

---

## TypeHandler 接口

所有类型处理器都实现了 `org.apache.ibatis.type.TypeHandler<T>` 接口：

```java
public interface TypeHandler<T> {
    // 设置参数：将 Java 类型转换为 JDBC 类型
    void setParameter(PreparedStatement ps, int i,
                      T parameter, JdbcType jdbcType) throws SQLException;

    // 获取结果：将 JDBC 类型转换为 Java 类型（按列名）
    T getResult(ResultSet rs, String columnName) throws SQLException;

    // 获取结果：将 JDBC 类型转换为 Java 类型（按列索引）
    T getResult(ResultSet rs, int columnIndex) throws SQLException;

    // 获取结果：存储过程（按列索引）
    T getResult(CallableStatement cs, int columnIndex) throws SQLException;
}
```

<aside>
💡

实际开发中一般继承 `BaseTypeHandler<T>` 抽象类，它实现了 `TypeHandler` 接口并提供了通用的 null 值处理逻辑，只需覆写四个 `getNullableResult` / `setNonNullParameter` 方法即可。

</aside>

---

## 默认类型处理器一览

| **类型处理器** | **Java 类型** | **JDBC 类型** |
| --- | --- | --- |
| BooleanTypeHandler | `Boolean` / `boolean` | 任何兼容的 BOOLEAN |
| ByteTypeHandler | `Byte` / `byte` | 任何兼容的 NUMERIC 或 BYTE |
| ShortTypeHandler | `Short` / `short` | 任何兼容的 NUMERIC 或 SMALLINT |
| IntegerTypeHandler | `Integer` / `int` | 任何兼容的 NUMERIC 或 INTEGER |
| LongTypeHandler | `Long` / `long` | 任何兼容的 NUMERIC 或 BIGINT |
| FloatTypeHandler | `Float` / `float` | 任何兼容的 NUMERIC 或 FLOAT |
| DoubleTypeHandler | `Double` / `double` | 任何兼容的 NUMERIC 或 DOUBLE |
| BigDecimalTypeHandler | `BigDecimal` | 任何兼容的 NUMERIC 或 DECIMAL |
| StringTypeHandler | `String` | CHAR、VARCHAR |
| ClobReaderTypeHandler | `Reader` | — |
| ClobTypeHandler | `String` | CLOB、LONGVARCHAR |
| NStringTypeHandler | `String` | NVARCHAR、NCHAR |
| NClobTypeHandler | `String` | NCLOB |
| BlobInputStreamTypeHandler | `InputStream` | — |
| ByteArrayTypeHandler | `byte[]` | 任何兼容的字节流类型 |
| BlobTypeHandler | `byte[]` | BLOB、LONGVARBINARY |
| DateTypeHandler | [`java.util.Date`](http://java.util.Date) | TIMESTAMP |
| DateOnlyTypeHandler | [`java.util.Date`](http://java.util.Date) | DATE |
| TimeOnlyTypeHandler | [`java.util.Date`](http://java.util.Date) | TIME |
| SqlDateTypeHandler | [`java.sql.Date`](http://java.sql.Date) | DATE |
| SqlTimeTypeHandler | `java.sql.Time` | TIME |
| SqlTimestampTypeHandler | `java.sql.Timestamp` | TIMESTAMP |
| ObjectTypeHandler | `Object` | OTHER 或未指定 |
| EnumTypeHandler | `Enum` | VARCHAR（存储枚举 `name()`） |
| EnumOrdinalTypeHandler | `Enum` | 任何兼容的 NUMERIC 或 DOUBLE（存储枚举 `ordinal()`） |

### JSR-310（Java 8 日期时间 API）类型处理器

从 MyBatis **3.4.5** 开始，默认支持 JSR-310 日期和时间 API：

| **类型处理器** | **Java 类型** | **JDBC 类型** |
| --- | --- | --- |
| InstantTypeHandler | `java.time.Instant` | TIMESTAMP |
| LocalDateTimeTypeHandler | `java.time.LocalDateTime` | TIMESTAMP |
| LocalDateTypeHandler | `java.time.LocalDate` | DATE |
| LocalTimeTypeHandler | `java.time.LocalTime` | TIME |
| OffsetDateTimeTypeHandler | `java.time.OffsetDateTime` | TIMESTAMP |
| OffsetTimeTypeHandler | `java.time.OffsetTime` | TIME |
| ZonedDateTimeTypeHandler | `java.time.ZonedDateTime` | TIMESTAMP |
| YearTypeHandler | `java.time.Year` | INTEGER |
| MonthTypeHandler | `java.time.Month` | INTEGER |
| YearMonthTypeHandler | `java.time.YearMonth` | VARCHAR 或 LONGVARCHAR |
| JapaneseDateTypeHandler | `java.time.chrono.JapaneseDate` | DATE |

---

## 配置类型处理器

### 方式一：在 mybatis-config.xml 中逐个注册

```xml
<typeHandlers>
    <typeHandler handler="cn.bjut.simple.type.EnabledTypeHandler"
                 javaType="cn.bjut.simple.type.Enabled"/>
</typeHandlers>
```

### 方式二：按包名自动扫描注册

```xml
<typeHandlers>
    <package name="cn.bjut.simple.type"/>
</typeHandlers>
```

<aside>
💡

使用包扫描方式时，需要在自定义 TypeHandler 类上通过注解来指定 Java 类型和 JDBC 类型：

- `@MappedTypes`：指定关联的 Java 类型
- `@MappedJdbcTypes`：指定关联的 JDBC 类型
</aside>

### 方式三：在映射文件中直接指定

可以在 `resultMap` 或 SQL 参数中直接指定使用的 TypeHandler，不需要全局注册：

```xml
<!-- 在 resultMap 中指定 -->
<result column="enabled" property="enabled"
        typeHandler="cn.bjut.simple.type.EnabledTypeHandler"/>

<!-- 在 SQL 参数中指定 -->
#{enabled, typeHandler=cn.bjut.simple.type.EnabledTypeHandler}
```

<aside>
⚠️

**注意事项**

- 全局注册（方式一、二）对所有匹配的 Java 类型生效
- 映射文件中指定（方式三）只对当前位置生效，**优先级高于全局配置**
- MyBatis 不会通过检测数据库元信息来决定使用哪种类型，需要在参数和结果映射中指明字段类型，以绑定到正确的处理器
</aside>

---

## 与第 6 章的关联

本附录所列的类型处理器是第 6 章 6.6 节内容的扩展。第 6 章以枚举类型 `Enabled` 为例，介绍了 `EnumTypeHandler`、`EnumOrdinalTypeHandler` 和自定义 `EnabledTypeHandler` 的实现，完整代码可参考 [第6章 MyBatis高级查询](%E7%AC%AC6%E7%AB%A0%20MyBatis%E9%AB%98%E7%BA%A7%E6%9F%A5%E8%AF%A2%2002f1f83d4adf4a4cb6474d907df75a04.md)。