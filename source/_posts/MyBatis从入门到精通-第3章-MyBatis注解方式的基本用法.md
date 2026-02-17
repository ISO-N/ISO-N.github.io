---
title: 第3章 MyBatis注解方式的基本用法
date: 2026-02-17 15:09:37
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
MyBatis 注解方式是将 SQL 语句直接写在接口方法上，适合需求简单的场景。优点是效率高、代码简洁；缺点是 SQL 变化时需要重新编译代码。最基本的注解有 **@Select**、**@Insert**、**@Update**、**@Delete** 四种。

---

## 3.1 @Select 注解

```java
@Select({"select id, role_name roleName, enabled, create_by createBy, create_time createTime",
         "from sys_role",
         "where id = #{id}"})
SysRole selectById(Long id);
```

注解方式同样需要解决 **表字段与 Java 属性的映射问题**，有三种方式：

1. SQL 中使用 **别名**（如上例 `role_name roleName`）
2. 使用 **mapUnderscoreToCamelCase** 全局配置自动映射
3. 使用 **@Results** 注解手动指定映射

### 3.1.1 使用 mapUnderscoreToCamelCase 配置

在 mybatis-config.xml 中开启下划线转驼峰的自动映射：

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

开启后 SQL 中无需手动指定别名：

```java
@Select("select id, role_name, enabled, create_by, create_time from sys_role where id = #{id}")
SysRole selectById2(Long id);
```

### 3.1.2 使用 resultMap 方式

使用 **@Results** 注解（对应 XML 中的 `<resultMap>`）手动映射：

```java
@Results(id = "roleResultMap", value = {
    @Result(property = "id", column = "id", id = true),
    @Result(property = "roleName", column = "role_name"),
    @Result(property = "enabled", column = "enabled"),
    @Result(property = "createBy", column = "create_by"),
    @Result(property = "createTime", column = "create_time")
})
@Select("select id, role_name, enabled, create_by, create_time from sys_role where id = #{id}")
SysRole selectById2(Long id);
```

- `@Result` 对应 XML 中的 `<result>` 元素，设置 `id = true` 表示主键（对应 `<id>`）
- 从 **MyBatis 3.3.1** 开始，`@Results` 增加了 `id` 属性，设置后可通过 **@ResultMap** 引用复用：

```java
@ResultMap("roleResultMap")
@Select("select * from sys_role")
List<SysRole> selectAll();
```

> **注意：** `@ResultMap` 也可以引用 XML 中 `<resultMap>` 的 id 属性值。
> 

---

## 3.2 @Insert 注解

### 3.2.1 不需要返回主键

```java
@Insert({"insert into sys_role(id, role_name, enabled, create_by, create_time)",
         "values(#{id}, #{roleName}, #{enabled}, #{createBy}, #{createTime, jdbcType=TIMESTAMP})"})
int insert(SysRole sysRole);
```

### 3.2.2 返回自增主键

使用 **@Options** 注解设置 `useGeneratedKeys` 和 `keyProperty`：

```java
@Insert({"insert into sys_role(role_name, enabled, create_by, create_time)",
         "values(#{roleName}, #{enabled}, #{createBy}, #{createTime, jdbcType=TIMESTAMP})"})
@Options(useGeneratedKeys = true, keyProperty = "id")
int insert2(SysRole sysRole);
```

- 与 insert 相比，SQL 中去掉了 id 列
- 多个列时可使用 `keyColumn` 属性

### 3.2.3 返回非自增主键

使用 **@SelectKey** 注解（对应 XML 中的 `<selectKey>`）：

```java
@Insert({"insert into sys_role(role_name, enabled, create_by, create_time)",
         "values(#{roleName}, #{enabled}, #{createBy}, #{createTime, jdbcType=TIMESTAMP})"})
@SelectKey(statement = "SELECT LAST_INSERT_ID()",
           keyProperty = "id",
           resultType = Long.class,
           before = false)
int insert3(SysRole sysRole);
```

- `before = false` → 等同于 XML 中 `order="AFTER"`
- `before = true` → 等同于 XML 中 `order="BEFORE"`

---

## 3.3 @Update 和 @Delete 注解

```java
@Update({"update sys_role",
         "set role_name = #{roleName},",
         "enabled = #{enabled},",
         "create_by = #{createBy},",
         "create_time = #{createTime, jdbcType=TIMESTAMP}",
         "where id = #{id}"})
int updateById(SysRole sysRole);

@Delete("delete from sys_role where id = #{id}")
int deleteById(Long id);
```

---

## 3.4 Provider 注解

MyBatis 提供了 **4 种 Provider 注解**，用于通过 Java 方法动态生成 SQL：

- `@SelectProvider`
- `@InsertProvider`
- `@UpdateProvider`
- `@DeleteProvider`

### 基本用法

在接口中使用 `@SelectProvider`：

```java
@SelectProvider(type = PrivilegeProvider.class, method = "selectById")
SysPrivilege selectById(Long id);
```

Provider 类的实现：

```java
public class PrivilegeProvider {

    // 方式一：使用 SQL 构建器
    public String selectById(final Long id) {
        return new SQL() {{
            SELECT("id, privilege_name, privilege_url");
            FROM("sys_privilege");
            WHERE("id = #{id}");
        }}.toString();
    }

    // 方式二：直接返回 SQL 字符串
    public String selectAll() {
        return "select * from sys_privilege";
    }
}
```

### Provider 注解的要求

- **type**：包含指定方法的类，该类必须有空的构造方法
- **method**：方法名，返回值必须是 `String` 类型

> SQL 较长或需要拼接时推荐使用 `new SQL()` 方式，简单 SQL 直接返回字符串即可。
> 

---

## 3.5 本章小结

| **注解** | **用途** | **对应 XML** |
| --- | --- | --- |
| @Select | 查询 | `<select>` |
| @Insert | 插入 | `<insert>` |
| @Update | 更新 | `<update>` |
| @Delete | 删除 | `<delete>` |
| @Results / @Result | 结果映射 | `<resultMap>` / `<result>` |
| @ResultMap | 引用已有映射 | `resultMap` 属性 |
| @Options | 返回自增主键 | `useGeneratedKeys` 属性 |
| @SelectKey | 返回非自增主键 | `<selectKey>` |
| @SelectProvider 等 | 动态生成 SQL | 无直接对应 |
- 注解方式适合 **简单 SQL**，代码更简洁
- 复杂 SQL 仍推荐使用 **XML 方式**
- 两种方式可以 **混合使用**，根据实际需求灵活选择