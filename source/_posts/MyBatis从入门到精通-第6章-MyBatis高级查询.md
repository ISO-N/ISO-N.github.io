---
title: 第6章 MyBatis高级查询
date: 2026-02-17 15:09:40
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
## 6.1 高级结果映射

本章介绍 MyBatis 中的**高级结果映射**，包括一对一映射、一对多映射和鉴别器映射，以及**存储过程**和**类型处理器**的用法。

---

## 6.2 一对一映射

假设在 RBAC 权限系统中，一个用户只能拥有一个角色，需要在查询用户信息的同时获取用户拥有的角色。

首先在 `SysUser` 中添加 `SysRole` 引用：

```java
private SysRole sysRole;
// getter/setter
```

### 方式一：使用自动映射（别名方式）

通过 SQL 别名让 MyBatis 自动将值匹配到嵌套对象的属性上。

```java
SysUser selectUserAndRoleById(Long id);
```

```xml
<select id="selectUserAndRoleById" resultType="SysUser">
    SELECT su.id, user_name, user_password, user_email,
           su.create_time, user_info, head_img,
           sr.id          AS "sysRole.id",
           sr.role_name   AS "sysRole.roleName",
           sr.create_by   AS "sysRole.createBy",
           sr.create_time AS "sysRole.createTime"
    FROM sys_user su
    INNER JOIN sys_user_role sur ON su.id = sur.user_id
    INNER JOIN sys_role sr ON sur.role_id = sr.id
    WHERE su.id = #{id}
</select>
```

<aside>
💡

MyBatis 支持复杂的属性映射，可以多层嵌套。例如 `sysRole.roleName` 会先查找 `sysRole` 属性并创建对象，然后将值绑定到其 `roleName` 属性上。这种方式称为**关联的嵌套结果映射**。

</aside>

### 方式二：使用 resultMap 配置

使用 `<resultMap>` 显式配置列与嵌套对象属性的映射关系。

```java
SysUser selectUserAndRoleById2(Long id);
```

```xml
<resultMap id="userRoleMap" extends="BaseResultMap"
           type="SysUser">
    <result property="sysRole.id"         column="role_id"/>
    <result property="sysRole.roleName"   column="role_name"/>
    <result property="sysRole.createBy"   column="create_by"/>
    <result property="sysRole.createTime" column="role_createTime"/>
</resultMap>

<select id="selectUserAndRoleById2" resultMap="userRoleMap">
    SELECT su.id, user_name, user_password, user_email,
           su.create_time, user_info, head_img,
           sr.id          AS "role_id",
           sr.role_name,
           sr.create_by,
           sr.create_time AS "role_createTime"
    FROM sys_user su
    INNER JOIN sys_user_role sur ON su.id = sur.user_id
    INNER JOIN sys_role sr ON sur.role_id = sr.id
    WHERE su.id = #{id}
</select>
```

<aside>
⚠️

注意为 [`sr.id`](http://sr.id) 和 `sr.create_time` 起别名，避免与 `sys_user` 表的同名列冲突。

</aside>

### 方式三：使用 association 标签关联

`<association>` 标签用于一对一的关联配置，可以关联 Java 类（`javaType`）或直接引用已有的 `resultMap`。

**association 标签属性：**

- `property`：对应实体类中的属性名（必填）
- `javaType`：属性对应的 Java 类型
- `resultMap`：引用已有的 resultMap（优先级高于 javaType 中的映射）
- `columnPrefix`：查询列的前缀，子标签中可省略该前缀

**关联 resultMap 示例（推荐）：**

```xml
<resultMap id="userRoleMap2" extends="BaseResultMap"
           type="SysUser">
    <association property="sysRole" columnPrefix="role_"
                 resultMap="cn.bjut.simple.mapper.RoleMapper.BaseResultMap"/>
</resultMap>
```

<aside>
📌

**跨文件引用 resultMap**：`association` 中的 `resultMap` 必须加上所引用 resultMap 所在文件的 `namespace` 全路径。

配置 `columnPrefix` 后，SQL 中从表的列名需要加上对应前缀，如 `role_name` 变为 `role_role_name`。

</aside>

### 方式四：association 嵌套查询

前三种方式通过一条复杂 SQL 一次查出结果，第四种方式使用**多次简单查询**分别获取数据。

**association 嵌套查询常用属性：**

- `select`：另一个映射查询的 id，MyBatis 会额外执行该查询获取嵌套对象
- `column`：将主查询中列的结果作为嵌套查询的参数。多参数写法：`column="{prop1=col1, prop2=col2}"`
- `fetchType`：数据加载方式，可选 `lazy`（延迟加载）和 `eager`（积极加载），会覆盖全局 `lazyLoadingEnabled` 配置

```java
SysUser selectUserAndRoleById3(Long id);
```

```xml
<resultMap id="userRoleMapSelect" extends="BaseResultMap"
           type="SysUser">
    <association property="sysRole"
                 column="{id=role_id}"
                 select="cn.bjut.simple.mapper.RoleMapper.selectByPrimaryKey"/>
</resultMap>

<select id="selectUserAndRoleById3" resultMap="userRoleMapSelect">
    SELECT su.id, user_name, user_password, user_email,
           su.create_time, user_info, head_img, role_id
    FROM sys_user su
    INNER JOIN sys_user_role sur ON su.id = sur.user_id
    WHERE su.id = #{id}
</select>
```

### 延迟加载

使用嵌套查询时，如果返回 N 条数据就会执行 N 次嵌套 SQL，这就是 **N+1 问题**。延迟加载可以解决此问题——只有当调用 `getRole()` 时才执行嵌套查询。

**开启全局延迟加载配置：**

```xml
<settings>
    <setting name="aggressiveLazyLoading" value="false"/>
    <setting name="lazyLoadingEnabled"    value="true"/>
</settings>
```

**在 association 中指定延迟加载：**

```xml
<association property="sysRole" fetchType="lazy"
             column="{id=role_id}"
             select="cn.bjut.simple.mapper.RoleMapper.selectByPrimaryKey"/>
```

<aside>
⚠️

**注意事项**

- 不要在调用 `getRole()` 前使用 `toString()` 打印对象，这会触发延迟加载
- `lazyLoadTriggerMethods` 配置项指定哪些方法触发延迟加载，默认值为 `equals, clone, hashCode, toString`
- 在 Spring 集成时，延迟加载属性只能在 **Service 层**调用。当结果从 Service 返回到 Controller 层后，SqlSession 已关闭，再获取延迟加载属性会抛出异常
</aside>

### 一对一映射四种方式对比

| 方式 | 实现方式 | 优点 | 缺点 |
| --- | --- | --- | --- |
| **SQL 别名自动映射** | SQL 中为列起嵌套别名 | 简单直接 | SQL 别名复杂，不可复用 |
| **resultMap 配置** | resultMap 中直接配置嵌套属性 | 映射清晰 | 配置量较大 |
| **association 关联 resultMap** | association + columnPrefix + 已有 resultMap | 可复用已有 resultMap | SQL 列名需加前缀 |
| **association 嵌套查询** | association + select 属性执行额外查询 | SQL 简单，支持延迟加载 | N+1 问题（可通过延迟加载缓解） |

---

## 6.3 一对多映射

一对多映射使用 `<collection>` 标签进行配置，有两种方式。

首先在 `SysUser` 中添加角色列表：

```java
private List<SysRole> roleList;
// getter/setter
```

### 方式一：collection 嵌套结果映射

通过一次 SQL 查询将所有结果查出，然后通过结果映射将数据分配到不同对象中。

```java
List<SysUser> selectAllUserAndRoles();
```

```xml
<resultMap id="userRoleListMap" extends="BaseResultMap"
           type="SysUser">
    <collection property="roleList" columnPrefix="role_"
                resultMap="cn.bjut.simple.mapper.RoleMapper.BaseResultMap"/>
</resultMap>

<select id="selectAllUserAndRoles" resultMap="userRoleListMap">
    SELECT su.id, user_name, user_password, user_email,
           su.create_time, user_info, head_img,
           sr.id          AS "role_id",
           sr.role_name      "role_role_name",
           sr.create_by      "role_create_by",
           sr.enabled        "role_enabled",
           sr.create_time AS "role_createTime"
    FROM sys_user su
    INNER JOIN sys_user_role sur ON su.id = sur.user_id
    INNER JOIN sys_role sr ON sur.role_id = sr.id
</select>
```

<aside>
📌

**id 标签的重要性**

MyBatis 通过 resultMap 中的 `<id>` 标签判断结果是否属于同一条数据：

- 配置了 `<id>`：只比较 id 字段，效率高（N 条数据只比较 N 次）
- 未配置 `<id>`（只有 `<result>`）：比较所有字段，效率低（N 条数据 × M 个字段 = N×M 次比较）
- `<id>` 配置的列在查询结果中为 NULL：会导致嵌套集合中只保留一条数据

**结论：必须配置 `<id>` 标签。**

</aside>

### 多层嵌套示例

除了用户对应多个角色，每个角色还对应多个权限。可以进行**多层嵌套**。

在 `SysRole` 中添加权限列表：

```java
private List<SysPrivilege> privilegeList;
// getter/setter
```

**RoleMapper.xml 中添加角色-权限映射：**

```xml
<resultMap id="rolePrivilegeListMap" extends="BaseResultMap"
           type="SysRole">
    <collection property="privilegeList" columnPrefix="privilege_"
                resultMap="cn.bjut.simple.mapper.PrivilegeMapper.BaseResultMap"/>
</resultMap>
```

**UserMapper.xml 中修改用户-角色映射，引用新的 rolePrivilegeListMap：**

```xml
<resultMap id="userRoleListMap2" extends="BaseResultMap"
           type="SysUser">
    <collection property="roleList" columnPrefix="role_"
                resultMap="cn.bjut.simple.mapper.RoleMapper.rolePrivilegeListMap"/>
</resultMap>

<select id="selectAllUserAndRolesAndPrivileges"
        resultMap="userRoleListMap2">
    SELECT su.id, user_name, user_password, user_email,
           su.create_time, user_info, head_img,
           sr.id             "role_id",
           sr.role_name      "role_role_name",
           sr.create_by      "role_create_by",
           sr.enabled        "role_enabled",
           sr.create_time    "role_createTime",
           sp.id             "role_privilege_id",
           sp.privilege_name "role_privilege_privilege_name",
           sp.privilege_url  "role_privilege_privilege_url"
    FROM sys_user su
    INNER JOIN sys_user_role sur ON su.id = sur.user_id
    INNER JOIN sys_role sr ON sur.role_id = sr.id
    INNER JOIN sys_role_privilege srp ON sr.id = srp.role_id
    INNER JOIN sys_privilege sp ON srp.privilege_id = sp.id
</select>
```

<aside>
⚠️

**columnPrefix 是叠加的**：由于 `userRoleListMap2` 中 collection 的前缀是 `role_`，而 `rolePrivilegeListMap` 中 collection 的前缀是 `privilege_`，所以 SQL 中权限相关列的别名前缀为 `role_privilege_`，例如 `sp.privilege_name` 的别名为 `role_privilege_privilege_name`。

</aside>

### 方式二：collection 嵌套查询

与 association 嵌套查询类似，通过多次简单 SQL 分别获取数据，支持延迟加载。

```java
SysUser selectUserAllRolesAndPrivilegesById(Long id);
```

**UserMapper.xml：**

```xml
<resultMap id="userRoleListMapSelect" extends="BaseResultMap"
           type="SysUser">
    <collection property="roleList" fetchType="lazy"
                column="{user_id=id}"
                select="cn.bjut.simple.mapper.RoleMapper.selectRolesByUserId"/>
</resultMap>

<select id="selectUserAllRolesAndPrivilegesById"
        resultMap="userRoleListMapSelect">
    SELECT id, user_name, user_password, user_email,
           create_time, user_info, head_img
    FROM sys_user
    WHERE id = #{id}
</select>
```

**RoleMapper.xml：**

```xml
<resultMap id="rolePrivilegeListMapSelect" extends="BaseResultMap"
           type="SysRole">
    <collection property="privilegeList" fetchType="lazy"
                column="{role_id=id}"
                select="cn.bjut.simple.mapper.PrivilegeMapper.selectByRoleId"/>
</resultMap>

<select id="selectRolesByUserId" resultMap="rolePrivilegeListMapSelect">
    SELECT id, role_name, enabled, create_by, create_time
    FROM sys_role
    INNER JOIN sys_user_role sur ON sys_role.id = sur.role_id
    WHERE user_id = #{user_id}
</select>
```

**PrivilegeMapper.xml：**

```xml
<select id="selectByRoleId" resultType="SysPrivilege">
    SELECT id, privilege_name, privilege_url
    FROM sys_privilege
    RIGHT JOIN sys_role_privilege srp ON sys_privilege.id = srp.privilege_id
    WHERE role_id = #{role_id}
</select>
```

<aside>
💡

嵌套查询配合全局延迟加载后，只有在调用 `getRoleList()` 或 `getPrivilegeList()` 时才会执行对应的嵌套 SQL，大幅减少不必要的查询。

</aside>

---

## 6.4 鉴别器映射

`<discriminator>` 标签类似 Java 的 `switch` 语句，根据某个列的值决定使用不同的映射配置。

**discriminator 属性：**

- `javaType`：指定列的 Java 类型，保证使用相同的类型比较值
- `column`：要进行鉴别比较的列

**case 标签属性：**

- `value`：与 `column` 列的值进行匹配
- `resultType`：匹配时使用的结果类型
- `resultMap`：匹配时使用的 resultMap（优先级高于 `resultType`）

### 示例

需求：查询用户的角色和权限时，当角色的 `enabled = 1` 时查出该角色的权限，`enabled = 0` 时不查询权限。

```xml
<resultMap id="rolePrivilegeListMapChoose" type="SysRole">
    <discriminator javaType="int" column="enabled">
        <case value="1"
              resultMap="rolePrivilegeListMapSelect"/>
        <case value="0"
              resultMap="cn.bjut.simple.mapper.RoleMapper.BaseResultMap"/>
    </discriminator>
</resultMap>
```

- 当 `enabled = 1` 时，使用 `rolePrivilegeListMapSelect`，会执行嵌套查询获取权限列表
- 当 `enabled = 0` 时，使用 `BaseResultMap`，只映射角色基本信息，不查询权限

---

## 6.5 存储过程

存储过程是一组为了完成特定功能的 SQL 语句集，存储在数据库中，经过第一次编译后再次调用不需要重新编译。

### 6.5.1 根据用户 id 查询用户信息

**创建存储过程：**

```sql
DROP PROCEDURE IF EXISTS `select_user_by_id`;
DELIMITER ;;
CREATE PROCEDURE `select_user_by_id`(
    IN  userId       BIGINT,
    OUT userName     VARCHAR(50),
    OUT userPassword VARCHAR(50),
    OUT userEmail    VARCHAR(50),
    OUT userInfo     TEXT,
    OUT headImg      BLOB,
    OUT createTime   DATETIME
)
BEGIN
    SELECT user_name, user_password, user_email,
           user_info, head_img, create_time
    INTO userName, userPassword, userEmail,
         userInfo, headImg, createTime
    FROM sys_user WHERE id = userId;
END
;;
DELIMITER ;
```

**MyBatis 调用：**

```java
void selectUserById(SysUser sysUser);
```

```xml
<select id="selectUserById" statementType="CALLABLE"
        useCache="false">
    {call select_user_by_id(
        #{id, mode=IN},
        #{userName, mode=OUT, jdbcType=VARCHAR},
        #{userPassword, mode=OUT, jdbcType=VARCHAR},
        #{userEmail, mode=OUT, jdbcType=VARCHAR},
        #{userInfo, mode=OUT, jdbcType=VARCHAR},
        #{headImg, mode=OUT, jdbcType=BLOB, javaType=_byte[]},
        #{createTime, mode=OUT, jdbcType=TIMESTAMP}
    )}
</select>
```

<aside>
⚠️

**存储过程注意事项**

- `statementType="CALLABLE"` 表示调用存储过程
- **OUT 模式的参数必须指定 `jdbcType`**（IN 模式下 MyBatis 有默认值，OUT 模式没有）
- `headImg` 额外指定了 `javaType=_byte[]`，因为 MyBatis 中 BLOB 默认对应的 Java 类型是 `Byte`
- `useCache="false"` 不支持 MyBatis 二级缓存
- 返回值为 `void`，结果会直接赋值到传入的 `sysUser` 对象属性中
- `jdbcType`、`mode` 的值所有字母**必须大写**
</aside>

### 6.5.2 分页查询存储过程

```sql
DROP PROCEDURE IF EXISTS `select_user_page`;
DELIMITER ;;
CREATE PROCEDURE `select_user_page`(
    IN  userName VARCHAR(50),
    IN  _offset  BIGINT,
    IN  _limit   BIGINT,
    OUT total    BIGINT
)
BEGIN
    SELECT COUNT(*) INTO total
    FROM sys_user
    WHERE user_name LIKE CONCAT('%', userName, '%');

    SELECT *
    FROM sys_user
    WHERE user_name LIKE CONCAT('%', userName, '%')
    LIMIT _offset, _limit;
END
;;
DELIMITER ;
```

```java
List<SysUser> selectUserPage(Map<String, Object> params);
```

```xml
<select id="selectUserPage" statementType="CALLABLE"
        useCache="false" resultMap="BaseResultMap">
    {call select_user_page(
        #{userName, mode=IN},
        #{_offset, mode=IN},
        #{_limit, mode=IN},
        #{total, mode=OUT, jdbcType=BIGINT}
    )}
</select>
```

<aside>
💡

此存储过程不仅通过 OUT 参数返回 `total`，还返回了查询结果集，所以需要配置 `resultMap` 和方法返回值 `List<SysUser>`。

</aside>

### 6.5.3 保存和删除用户的存储过程

**保存用户及角色关联：**

```sql
DROP PROCEDURE IF EXISTS `insert_user_and_roles`;
DELIMITER ;;
CREATE PROCEDURE `insert_user_and_roles`(
    OUT userId       BIGINT,
    IN  userName     VARCHAR(50),
    IN  userPassword VARCHAR(50),
    IN  userEmail    VARCHAR(50),
    IN  userInfo     TEXT,
    IN  headImg      BLOB,
    OUT createTime   DATETIME,
    IN  roleIds      VARCHAR(200)
)
BEGIN
    SET createTime = NOW();
    INSERT INTO sys_user(user_name, user_password, user_email,
                         user_info, head_img, create_time)
    VALUES (userName, userPassword, userEmail,
            userInfo, headImg, createTime);
    SELECT LAST_INSERT_ID() INTO userId;
    -- roleIds 格式为 "1,2,3"
    SET roleIds = CONCAT(',', roleIds, ',');
    INSERT INTO sys_user_role(user_id, role_id)
    SELECT userId, id FROM sys_role
    WHERE INSTR(roleIds, CONCAT(',', id, ',')) > 0;
END
;;
DELIMITER ;
```

```java
int insertUserAndRoles(@Param("user") SysUser user,
                       @Param("roleIds") String roleIds);
```

```xml
<insert id="insertUserAndRoles" statementType="CALLABLE">
    {call insert_user_and_roles(
        #{user.id, mode=OUT, jdbcType=BIGINT},
        #{user.userName, mode=IN},
        #{user.userPassword, mode=IN},
        #{user.userEmail, mode=IN},
        #{user.userInfo, mode=IN},
        #{user.headImg, mode=IN, jdbcType=BLOB},
        #{user.createTime, mode=OUT, jdbcType=TIMESTAMP},
        #{roleIds, mode=IN}
    )}
</insert>
```

**删除用户及角色关联：**

```sql
DROP PROCEDURE IF EXISTS `delete_user_by_id`;
DELIMITER ;;
CREATE PROCEDURE `delete_user_by_id`(IN userId BIGINT)
BEGIN
    DELETE FROM sys_user_role WHERE user_id = userId;
    DELETE FROM sys_user WHERE id = userId;
END
;;
DELIMITER ;
```

```xml
<delete id="deleteUserById" statementType="CALLABLE">
    {call delete_user_by_id(#{id, mode=IN})}
</delete>
```

---

## 6.6 类型处理器（TypeHandler）

### 枚举类型的处理

以角色的 `enabled` 字段为例，1 表示启用，0 表示禁用，适合用枚举类型表示：

```java
public enum Enabled {
    disabled(0),
    enabled(1);

    private final int value;

    Enabled(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}
```

### MyBatis 内置的枚举处理器

| 处理器 | 转换方式 | 示例 |
| --- | --- | --- |
| **EnumTypeHandler**（默认） | 将枚举转换为字符串字面值（`name()`） | `Enabled.disabled` → `"disabled"` |
| **EnumOrdinalTypeHandler** | 将枚举转换为索引值（`ordinal()`） | `Enabled.disabled` → `0` |

**在 mybatis-config.xml 中配置枚举处理器：**

```xml
<typeHandlers>
    <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler"
                 javaType="cn.bjut.simple.type.Enabled"/>
</typeHandlers>
```

<aside>
⚠️

`EnumOrdinalTypeHandler` 使用的是枚举的 **声明顺序索引**（`ordinal()`），而不是自定义的 `value` 值。如果枚举声明顺序与自定义值不一致，会导致映射错误。

</aside>

### 自定义 TypeHandler

当内置处理器无法满足需求时（如需要使用自定义的 `value` 值），需要自定义 TypeHandler：

```java
public class EnabledTypeHandler extends BaseTypeHandler<Enabled> {
    private final Map<Integer, Enabled> enabledMap = new HashMap<>();

    public EnabledTypeHandler() {
        for (Enabled enabled : Enabled.values()) {
            enabledMap.put(enabled.getValue(), enabled);
        }
    }

    // 设置参数时的类型转换
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i,
            Enabled parameter, JdbcType jdbcType) throws SQLException {
        ps.setInt(i, parameter.getValue());
    }

    // 通过列名从结果集取值并转换
    @Override
    public Enabled getNullableResult(ResultSet rs,
            String columnName) throws SQLException {
        Integer value = rs.getObject(columnName, Integer.class);
        return enabledMap.get(value);
    }

    // 通过列索引从结果集取值并转换
    @Override
    public Enabled getNullableResult(ResultSet rs,
            int columnIndex) throws SQLException {
        Integer value = rs.getObject(columnIndex, Integer.class);
        return enabledMap.get(value);
    }

    // 存储过程的类型转换
    @Override
    public Enabled getNullableResult(CallableStatement cs,
            int columnIndex) throws SQLException {
        Integer value = cs.getObject(columnIndex, Integer.class);
        return enabledMap.get(value);
    }
}
```

**注册自定义处理器：**

```xml
<typeHandlers>
    <typeHandler handler="cn.bjut.simple.type.EnabledTypeHandler"
                 javaType="cn.bjut.simple.type.Enabled"/>
</typeHandlers>
```

---

## 本章核心知识总结

| 标签 / 概念 | 说明 |
| --- | --- |
| **`<association>`** | 一对一关联映射，支持嵌套结果映射和嵌套查询两种方式 |
| **`<collection>`** | 一对多集合映射，支持嵌套结果映射和嵌套查询两种方式 |
| **`<discriminator>`** | 鉴别器映射，根据列值选择不同的映射配置，类似 switch 语句 |
| **columnPrefix** | 列前缀，多层嵌套时前缀会叠加 |
| **`<id>` 标签** | 标记主键列，在嵌套映射中用于判断数据是否相同，必须配置以保证正确性和性能 |
| **延迟加载** | 配合嵌套查询使用，解决 N+1 问题，只在需要时执行嵌套 SQL |
| **存储过程** | 通过 `statementType="CALLABLE"` 调用，OUT 参数必须指定 `jdbcType` |
| **TypeHandler** | 处理 Java 类型与数据库类型之间的转换，可自定义实现 |