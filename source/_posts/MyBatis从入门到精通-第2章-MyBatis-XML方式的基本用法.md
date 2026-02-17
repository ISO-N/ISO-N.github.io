---
title: 第2章 MyBatis XML方式的基本用法
date: 2026-02-17 15:09:36
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
## 2.1 一个简单的权限控制需求

本章通过完成**权限管理**的常见业务来学习 MyBatis XML 方式的基本用法，采用 **RBAC（Role-Based Access Control，基于角色的访问控制）** 模型。

- 一个用户拥有若干角色，一个角色拥有若干权限
- 用户与角色之间、角色与权限之间，一般是**多对多**关系

### 2.1.1 创建数据库表

共 5 张表：用户表、角色表、权限表、用户角色关联表、角色权限关联表。

```sql
-- 用户表
CREATE TABLE sys_user (
    id            BIGINT       NOT NULL AUTO_INCREMENT COMMENT '用户ID',
    user_name     VARCHAR(50)  COMMENT '用户名',
    user_password VARCHAR(50)  COMMENT '密码',
    user_email    VARCHAR(50)  COMMENT '邮箱',
    user_info     TEXT         COMMENT '简介',
    head_img      BLOB         COMMENT '头像',
    create_time   DATETIME     COMMENT '创建时间',
    PRIMARY KEY (id)
);

-- 角色表
CREATE TABLE sys_role (
    id          BIGINT       NOT NULL AUTO_INCREMENT COMMENT '角色ID',
    role_name   VARCHAR(50)  COMMENT '角色名',
    enabled     INT          COMMENT '有效标志',
    create_by   BIGINT       COMMENT '创建人',
    create_time DATETIME     COMMENT '创建时间',
    PRIMARY KEY (id)
);

-- 权限表
CREATE TABLE sys_privilege (
    id             BIGINT        NOT NULL AUTO_INCREMENT COMMENT '权限ID',
    privilege_name VARCHAR(50)   COMMENT '权限名称',
    privilege_url  VARCHAR(200)  COMMENT '权限URL',
    PRIMARY KEY (id)
);

-- 用户角色关联表
CREATE TABLE sys_user_role (
    user_id BIGINT COMMENT '用户ID',
    role_id BIGINT COMMENT '角色ID'
);

-- 角色权限关联表
CREATE TABLE sys_role_privilege (
    role_id      BIGINT COMMENT '角色ID',
    privilege_id BIGINT COMMENT '权限ID'
);
```

### 2.1.2 创建实体类

在 `cn.bjut.simple.model` 包下创建 5 张表对应的实体类：`SysUser`、`SysRole`、`SysPrivilege`、`SysUserRole`、`SysRolePrivilege`。

<aside>
⚠️

**注意事项**

- **不要使用 Java 基本数据类型**（byte、int、short、long、float、double、char、boolean），应使用对应的包装类型。因为基本类型有默认值（如 `int` 默认为 0），用 `!= null` 判断时会出现隐藏 bug
- MyBatis 默认遵循**下划线转驼峰**命名：表名 `sys_user` → 类名 `SysUser`，字段 `user_name` → 属性 `userName`
</aside>

**SysUser 实体类示例：**

```java
public class SysUser {
    private Long   id;
    private String userName;
    private String userPassword;
    private String userEmail;
    private String userInfo;
    private byte[] headImg;       // byte[] 不是基本数据类型
    private Date   createTime;
    // getter 和 setter 省略
}
```

---

## 2.2 使用接口 + XML 方式

### 2.2.1 创建 Mapper.xml 和接口

- 在 `src/main/resources` 的 `cn/bjut/simple/mapper` 目录下创建 5 个 XML 映射文件
- 在 `src/main/java` 的 `cn.bjut.simple.mapper` 包下创建对应接口类

**Mapper.xml 模板：**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.bjut.simple.mapper.UserMapper">

</mapper>
```

<aside>
📌

**关键点**：`namespace` 的值必须配置成接口的**全限定名**，MyBatis 通过这个值将接口和 XML 映射文件关联起来。

</aside>

### 2.2.2 在 mybatis-config.xml 中注册

**方式一：逐个注册（不推荐）**

```xml
<mappers>
    <mapper resource="cn/bjut/simple/mapper/UserMapper.xml"/>
    <mapper resource="cn/bjut/simple/mapper/RoleMapper.xml"/>
    <!-- 新增 Mapper 需要手动添加，维护麻烦 -->
</mappers>
```

**方式二：包扫描（推荐）**

```xml
<mappers>
    <package name="cn.bjut.simple.mapper"/>
</mappers>
```

### 2.2.3 基础测试类

提取公共的 `BaseMapperTest`，后续所有测试类继承它：

```java
public class BaseMapperTest {
    private static SqlSessionFactory sqlSessionFactory;

    @BeforeClass
    public static void init() {
        try {
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public SqlSession getSqlSession() {
        return sqlSessionFactory.openSession();
    }
}
```

---

## 2.3 SELECT 用法

### 查询单条数据 — selectById

**接口方法：**

```java
SysUser selectById(Long id);
```

**XML 映射（使用 resultMap）：**

```xml
<resultMap id="userMap" type="cn.bjut.simple.model.SysUser">
    <id     property="id"           column="id"/>
    <result property="userName"     column="user_name"/>
    <result property="userPassword" column="user_password"/>
    <result property="userEmail"    column="user_email"/>
    <result property="userInfo"     column="user_info"/>
    <result property="headImg"      column="head_img"/>
    <result property="createTime"   column="create_time"/>
</resultMap>

<select id="selectById" resultMap="userMap">
    SELECT * FROM sys_user WHERE id = #{id}
</select>
```

### 查询多条数据 — selectAll

**接口方法：**

```java
List<SysUser> selectAll();
```

**XML 映射（使用 resultType + 别名）：**

```xml
<select id="selectAll" resultType="cn.bjut.simple.model.SysUser">
    SELECT id,
           user_name     userName,
           user_password userPassword,
           user_email    userEmail,
           user_info     userInfo,
           head_img      headImg,
           create_time   createTime
    FROM sys_user
</select>
```

### resultMap vs resultType

| 对比项 | resultMap | resultType |
| --- | --- | --- |
| **映射方式** | 通过 `<resultMap>` 标签显式配置列与属性的映射 | 直接指定返回类型，依赖列名与属性名一致（或别名） |
| **列名不一致时** | 在 resultMap 中配置 column → property | 需要在 SQL 中为列设置别名 |
| **适用场景** | 复杂映射、嵌套对象、继承映射 | 简单查询，列名与属性名一致或可通过别名匹配 |

<aside>
💡

**自动驼峰映射**：在 mybatis-config.xml 中配置 `mapUnderscoreToCamelCase` 为 `true`，可自动将下划线命名的列映射到驼峰命名的属性，无需手动写别名或 resultMap。

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```

</aside>

### resultMap 标签详解

**resultMap 属性：**

- `id`：唯一标识符，在 `<select>` 中通过 `resultMap` 属性引用
- `type`：映射到的 Java 实体类全限定名
- `autoMapping`：可选，覆盖全局的 autoMappingBehavior 配置
- `extends`：可选，继承其他 resultMap

**resultMap 子标签：**

- `<id>`：标记主键字段（帮助提升整体性能）
- `<result>`：普通字段映射
- `<constructor>`：使用构造方法注入结果（含 `<idArg>` 和 `<arg>`）
- `<association>`：复杂类型关联（多对一 / 一对一）
- `<collection>`：复杂类型集合（一对多）
- `<discriminator>`：根据结果值决定使用哪个映射

**id / result 标签属性：**

- `column`：数据库列名或别名
- `property`：Java 对象属性名，支持嵌套如 `address.street`
- `javaType`：Java 类型全限定名或别名
- `jdbcType`：JDBC 类型，主要用于插入/更新/删除时可能为空的列

### 多表关联查询

**需求：** 根据用户 id 查询其拥有的所有角色

```java
List<SysRole> selectRolesByUserId(Long userId);
```

```xml
<select id="selectRolesByUserId" resultType="SysRole">
    SELECT r.id, r.role_name roleName, r.enabled,
           r.create_by createBy, r.create_time createTime
    FROM sys_user u
    INNER JOIN sys_user_role ur ON u.id = ur.user_id
    INNER JOIN sys_role r ON ur.role_id = r.id
    WHERE u.id = #{userId}
</select>
```

### 关联查询需要额外字段时的 3 种处理方式

当查询结果除了 `sys_role` 的字段，还需要 `sys_user` 的部分字段（如 `user_name`）时：

1. **直接添加属性**：在 `SysRole` 中添加 `userName` 属性（简单但破坏实体类纯净性）
2. **新建扩展类（继承 + 基本字段）**：适合少量额外字段

```java
public class SysRoleExtend extends SysRole {
    private String userName;
    // getter/setter
}
```

1. **新建扩展类（继承 + 实体对象）**（推荐）：适合需要大量额外字段的场景

```java
public class SysRoleExtend extends SysRole {
    private SysUser user;
    // getter/setter
}
```

此时 SQL 中用 `user.userName`、`user.userEmail` 形式设置别名，MyBatis 会自动赋值到嵌套对象的属性。

---

## 2.4 INSERT 用法

### 简单的 insert

```java
int insert(SysUser sysUser);
```

```xml
<insert id="insert">
    INSERT INTO sys_user (id, user_name, user_password, user_email,
                          user_info, head_img, create_time)
    VALUES (#{id}, #{userName}, #{userPassword}, #{userEmail},
            #{userInfo}, #{headImg, jdbcType=BLOB},
            #{createTime, jdbcType=TIMESTAMP})
</insert>
```

<aside>
⚠️

- 为了防止类型错误，对于特殊类型（如 BLOB、TIMESTAMP 等），建议**指定 `jdbcType`**
- `insert` 返回值是 `int`，表示**受影响的行数**，而非自增主键值
</aside>

### 获取自增主键 — useGeneratedKeys

```xml
<insert id="insert2" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO sys_user (user_name, user_password, user_email,
                          user_info, head_img, create_time)
    VALUES (#{userName}, #{userPassword}, #{userEmail},
            #{userInfo}, #{headImg, jdbcType=BLOB},
            #{createTime, jdbcType=TIMESTAMP})
</insert>
```

- `useGeneratedKeys="true"`：使用 JDBC 的 `getGeneratedKeys` 方法获取自增主键
- `keyProperty="id"`：将获取到的主键值赋给参数对象的 `id` 属性
- 执行 insert 后，通过 `sysUser.getId()` 即可获取新生成的主键

### 获取非自增主键 — selectKey

适用于 Oracle 等不支持自增主键的数据库，或需要在插入前生成主键的场景：

```xml
<insert id="insert3">
    <selectKey keyColumn="id" resultType="long"
               keyProperty="id" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>
    INSERT INTO sys_user (user_name, user_password, user_email,
                          user_info, head_img, create_time)
    VALUES (#{userName}, #{userPassword}, #{userEmail},
            #{userInfo}, #{headImg, jdbcType=BLOB},
            #{createTime, jdbcType=TIMESTAMP})
</insert>
```

- `order="BEFORE"`：在 INSERT 之前执行（如 Oracle 序列）
- `order="AFTER"`：在 INSERT 之后执行（如 MySQL `LAST_INSERT_ID()`）

---

## 2.5 UPDATE 用法

```java
int updateById(SysUser sysUser);
```

```xml
<update id="updateById">
    UPDATE sys_user
    SET user_name     = #{userName},
        user_password = #{userPassword},
        user_email    = #{userEmail},
        user_info     = #{userInfo},
        head_img      = #{headImg, jdbcType=BLOB},
        create_time   = #{createTime, jdbcType=TIMESTAMP}
    WHERE id = #{id}
</update>
```

- 返回值 `int` 为受影响的行数
- 与 `<insert>` 类似，特殊类型需要指定 `jdbcType`

---

## 2.6 DELETE 用法

```java
int deleteById(Long id);
```

```xml
<delete id="deleteById">
    DELETE FROM sys_user WHERE id = #{id}
</delete>
```

- 参数只有一个基本类型时，`#{}` 中的名称可以任意
- 返回值为受影响的行数

---

## 2.7 多个接口参数的用法

前面的方法都只有一个参数（基本类型或 JavaBean），实际开发中经常需要**多个参数**。

### 方式一：使用 `@Param` 注解（推荐）

```java
List<SysRole> selectRolesByUserIdAndEnabled(
    @Param("userId") Long userId,
    @Param("enabled") Integer enabled);
```

```xml
<select id="selectRolesByUserIdAndEnabled" resultType="SysRole">
    SELECT r.id, r.role_name roleName, r.enabled,
           r.create_by createBy, r.create_time createTime
    FROM sys_user u
    INNER JOIN sys_user_role ur ON u.id = ur.user_id
    INNER JOIN sys_role r ON ur.role_id = r.id
    WHERE u.id = #{userId} AND r.enabled = #{enabled}
</select>
```

<aside>
💡

`@Param` 注解中的值对应 XML 中 `#{}` 里的参数名。给参数加上该注解后，MyBatis 会自动将多个参数放入一个 Map 中，注解的值作为 key。

</aside>

### 方式二：使用 JavaBean 封装参数

当参数较多时，可以封装为一个对象传入：

```java
List<SysRole> selectRolesByCondition(RoleQuery query);
```

### 方式三：使用 Map 传参（不推荐）

```java
List<SysRole> selectByMap(Map<String, Object> params);
```

灵活但缺少类型安全和可读性，不建议在正式项目中使用。

---

## 本章核心知识总结

| 标签 / 概念 | 说明 |
| --- | --- |
| **`<select>`** | 查询语句，通过 `resultMap` 或 `resultType` 设置返回值类型 |
| **`<insert>`** | 插入语句，可通过 `useGeneratedKeys` 或 `<selectKey>` 获取主键 |
| **`<update>`** | 更新语句，返回受影响的行数 |
| **`<delete>`** | 删除语句，返回受影响的行数 |
| **`<resultMap>`** | 显式定义数据库列与 Java 属性的映射关系，支持嵌套和继承 |
| **`#{}`** | 预编译参数占位符，防止 SQL 注入，对应 JDBC 的 `?` |
| **`@Param`** | 多参数时为每个参数指定名称，XML 中通过该名称引用 |
| **`namespace`** | XML 命名空间，必须与接口全限定名一致 |
| **mapUnderscoreToCamelCase** | 全局配置，自动将下划线列名映射到驼峰属性名 |