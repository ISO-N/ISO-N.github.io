---
title: 第4章 MyBatis动态SQL
date: 2026-02-17 15:09:38
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
MyBatis 的强大特性之一便是它的**动态 SQL**。MyBatis 3 采用了功能强大的 **OGNL**（Object-Graph Navigation Language）表达式语言，在 XML 中支持以下几种标签：

- `if`
- `choose`（`when`、`otherwise`）
- `trim`（`where`、`set`）
- `foreach`
- `bind`

---

## 4.1 if 用法

`if` 标签有一个必填的 **test** 属性，值是符合 OGNL 要求的判断表达式，结果为 `true` 或 `false`。

**常用判断规则：**

- `property != null` / `property == null`：适用于**任何类型**，判断是否为空
- `property != ''` / `property == ''`：仅适用于 **String 类型**，判断是否为空字符串
- `and`（相当于 `&&`）、`or`（相当于 `||`）：多条件连接

### 4.1.1 在 WHERE 条件中使用 if

> **需求：** 高级查询——只输入用户名时模糊查询，只输入邮箱时精确匹配，同时输入则两个条件同时生效。
> 

```xml
<select id="selectByUser" resultType="SysUser">
    select id, user_name, user_password, user_email,
           user_info, head_img, create_time
    from sys_user
    where 1 = 1
    <if test="userName != null and userName != ''">
        and user_name like concat('%', #{userName}, '%')
    </if>
    <if test="userEmail != null and userEmail != ''">
        and user_email = #{userEmail}
    </if>
</select>
```

<aside>
⚠️

`where 1 = 1` 是一个默认条件，防止所有 if 都不满足时 SQL 以 `where` 结尾导致语法错误。后续 4.3 节的 `<where>` 标签可以替代这种写法。

</aside>

### 4.1.2 在 UPDATE 更新列中使用 if

> **需求：** 只更新有变化的字段，不能将原来有值的字段更新为空。方法名通常以 **Selective** 作为后缀。
> 

```xml
<update id="updateByIdSelective">
    update sys_user
    set
    <if test="userName != null and userName != ''">
        user_name = #{userName},
    </if>
    <if test="userPassword != null and userPassword != ''">
        user_password = #{userPassword},
    </if>
    <if test="userEmail != null and userEmail != ''">
        user_email = #{userEmail},
    </if>
    <if test="userInfo != null and userInfo != ''">
        user_info = #{userInfo},
    </if>
    <if test="headImg != null">
        head_img = #{headImg, jdbcType=BLOB},
    </if>
    <if test="createTime != null">
        create_time = #{createTime, jdbcType=TIMESTAMP},
    </if>
    id = #{id}
    where id = #{id}
</update>
```

<aside>
💡

**注意两点：** ①每个 if 内 SQL 末尾的**逗号**；②`where` 前的 `id = #{id}` 保证所有 if 都不满足时 SQL 仍合法（`set id=#{id} where id=#{id}`）。

</aside>

### 4.1.3 在 INSERT 动态插入列中使用 if

> **需求：** 插入数据时，参数不为空就用传入值，参数为空则使用数据库默认值。
> 

```xml
<insert id="insert2" useGeneratedKeys="true" keyProperty="id">
    insert into sys_user(
        user_name, user_password,
        <if test="userEmail != null and userEmail != ''">
            user_email,
        </if>
        user_info, head_img, create_time)
    values(
        #{userName}, #{userPassword},
        <if test="userEmail != null and userEmail != ''">
            #{userEmail},
        </if>
        #{userInfo}, #{headImg, jdbcType=BLOB},
        #{createTime, jdbcType=TIMESTAMP})
</insert>
```

<aside>
⚠️

列部分增加了 if 条件，`values` 部分也**必须增加相同的 if 条件**，保证上下完全对应。

</aside>

---

## 4.2 choose 用法

`if` 只能实现 `if` 逻辑，无法实现 `if...else`。`choose` 标签包含 `when` 和 `otherwise`，类似于 Java 中的 `switch`。

- 一个 `choose` 中**至少有一个** `when`
- `otherwise` 有 **0 个或 1 个**

> **需求：** id 有值时优先用 id 查询；id 无值但用户名有值时按用户名查询；都没有则查无结果。
> 

```xml
<select id="selectByIdOrUserName" resultType="SysUser">
    select id, user_name, user_password, user_email,
           user_info, head_img, create_time
    from sys_user
    where 1 = 1
    <choose>
        <when test="id != null">
            and id = #{id}
        </when>
        <when test="userName != null and userName != ''">
            and user_name = #{userName}
        </when>
        <otherwise>
            limit 0
        </otherwise>
    </choose>
</select>
```

<aside>
💡

`otherwise` 中的 `limit 0` 保证条件都不满足时不会查出所有用户，避免返回多条结果导致报错。

</aside>

---

## 4.3 where、set、trim 用法

这三个标签解决了类似的问题，`where` 和 `set` 都属于 `trim` 的一种具体用法。

### 4.3.1 where 用法

**作用：** 如果标签包含的元素有返回值，就插入一个 `WHERE`；如果内容以 `AND` 或 `OR` 开头，自动将其**剔除**。

```xml
<select id="selectByUser" resultType="SysUser">
    select id, user_name, user_password, user_email,
           user_info, head_img, create_time
    from sys_user
    <where>
        <if test="userName != null and userName != ''">
            and user_name like concat('%', #{userName}, '%')
        </if>
        <if test="userEmail != null and userEmail != ''">
            and user_email = #{userEmail}
        </if>
    </where>
</select>
```

相比 `where 1 = 1`，生成的 SQL **更干净**，不会出现多余的默认条件。

### 4.3.2 set 用法

**作用：** 如果标签包含的元素有返回值，就插入一个 `SET`；如果内容以**逗号结尾**，自动将逗号**剔除**。

```xml
<update id="updateByIdSelective">
    update sys_user
    <set>
        <if test="userName != null and userName != ''">
            user_name = #{userName},
        </if>
        <if test="userPassword != null and userPassword != ''">
            user_password = #{userPassword},
        </if>
        <if test="userEmail != null and userEmail != ''">
            user_email = #{userEmail},
        </if>
        id = #{id},
    </set>
    where id = #{id}
</update>
```

<aside>
⚠️

`set` 标签并未解决全部问题——如果所有 if 都不满足且没有 `id = #{id}` 这样的保底赋值，仍会报错。

</aside>

### 4.3.3 trim 用法

`where` 和 `set` 的底层都是通过 **TrimSqlNode** 实现的。`trim` 有以下 4 个属性：

| 属性 | 说明 |
| --- | --- |
| `prefix` | 内容不为空时，增加指定**前缀** |
| `prefixOverrides` | 内容不为空时，去掉匹配的**前缀**字符串 |
| `suffix` | 内容不为空时，增加指定**后缀** |
| `suffixOverrides` | 内容不为空时，去掉匹配的**后缀**字符串 |

**用 trim 实现 where：**

```xml
<trim prefix="WHERE" prefixOverrides="AND |OR ">
    ...
</trim>
```

**用 trim 实现 set：**

```xml
<trim prefix="SET" suffixOverrides=",">
    ...
</trim>
```

<aside>
💡

`AND`  和 `OR`  后面的**空格不能省略**，防止匹配到 `andes`、`orders` 等单词。实际 `prefixOverrides` 还包含 `AND\n`、`OR\n`、`AND\r`、`OR\r`、`AND\t`、`OR\t` 等变体。

</aside>

---

## 4.4 foreach 用法

`foreach` 可以对**数组**、**Map** 或实现了 **Iterable 接口**（List、Set 等）的对象进行遍历。

**标签属性：**

| 属性 | 说明 |
| --- | --- |
| `collection` | **必填**，要迭代的集合/数组属性名 |
| `item` | 迭代出的每一个元素的变量名 |
| `index` | 索引值（Map 时为 key） |
| `open` | 循环开始的字符串 |
| `close` | 循环结束的字符串 |
| `separator` | 每次循环之间的分隔符 |

### 4.4.1 foreach 实现 in 集合

```xml
<select id="selectByIdList" resultType="SysUser">
    select id, user_name, user_password, user_email,
           user_info, head_img, create_time
    from sys_user
    where id in
    <foreach collection="list" open="(" close=")" separator=","
             item="id" index="i">
        #{id}
    </foreach>
</select>
```

**collection 取值规则（来源于 DefaultSqlSession 的 wrapCollection 方法）：**

- 参数为 `List` → `collection="list"`
- 参数为数组 → `collection="array"`
- 使用 `@Param("xxx")` 注解 → `collection="xxx"`
- 参数为对象中的属性 → `collection="属性名"`
- 参数为 Map 且遍历其中某个值 → `collection="key名"`

### 4.4.2 foreach 实现批量插入

> 批量插入是 SQL-92 特性，支持的数据库：MySQL、DB2、SQL Server 2008+、SQLite 3.7.11+、H2 等。
> 

```xml
<insert id="insertUserList" useGeneratedKeys="true" keyProperty="id">
    insert into sys_user(user_name, user_password, user_email,
                         user_info, head_img, create_time)
    values
    <foreach collection="list" item="user" separator=",">
        (#{user.userName}, #{user.userPassword}, #{user.userEmail},
         #{user.userInfo}, #{user.headImg, jdbcType=BLOB},
         #{user.createTime, jdbcType=TIMESTAMP})
    </foreach>
</insert>
```

<aside>
💡

`item="user"` 指定了遍历变量名，取内部属性时用 `user.属性名`。批量返回主键需要 MyBatis 3.3.1 及以上版本。

</aside>

### 4.4.3 foreach 实现动态 UPDATE

通过 Map 参数遍历键值对实现动态更新：

```xml
<update id="updateByMap">
    update sys_user
    <set>
        <foreach collection="_parameter" item="val" index="key"
                 separator=",">
            ${key} = #{val}
        </foreach>
    </set>
    where id = #{id}
</update>
```

---

## 4.5 bind 用法

**作用：** 使用 OGNL 表达式创建一个变量并绑定到上下文中。

> **问题引入：** `concat()` 函数在 MySQL 中支持多个参数，但在 Oracle 中**仅支持两个**，更换数据库时可能出错。
> 

```xml
<if test="userName != null and userName != ''">
    <bind name="nameLike" value="'%' + userName + '%'"/>
    and user_name like #{nameLike}
</if>
```

<aside>
✅

使用 `bind` 既可以方便**更换数据库**，也能**防止 SQL 注入**。

</aside>

---

## 4.6 多数据库支持

在 MyBatis 核心配置文件中配置 `databaseIdProvider`，可以根据不同数据库执行不同 SQL：

```xml
<databaseIdProvider type="DB_VENDOR">
    <property name="SQL Server" value="sqlserver"/>
    <property name="DB2" value="db2"/>
    <property name="MySQL" value="mysql"/>
    <property name="Oracle" value="oracle"/>
</databaseIdProvider>
```

**使用方式一：整条 SQL 匹配**

```xml
<select id="selectByUser" databaseId="mysql" resultType="SysUser">
    select * from sys_user where
    user_name like concat('%', #{userName}, '%')
</select>

<select id="selectByUser" databaseId="oracle" resultType="SysUser">
    select * from sys_user where
    user_name like '%' || #{userName} || '%'
</select>
```

**使用方式二：通过 if + `_databaseId` 局部匹配**

```xml
<select id="selectByUser" resultType="SysUser">
    select * from sys_user
    <where>
        <if test="_databaseId == 'mysql'">
            user_name like concat('%', #{userName}, '%')
        </if>
        <if test="_databaseId == 'oracle'">
            user_name like '%' || #{userName} || '%'
        </if>
    </where>
</select>
```

<aside>
💡

如果同时存在带 `databaseId` 和不带 `databaseId` 的同名语句，不带的会被**忽略**。

</aside>

---

## 4.7 OGNL 用法

MyBatis 动态 SQL 中常用的 OGNL 表达式：

| 表达式 | 说明 |
| --- | --- |
| `e1 or e2` | 或 |
| `e1 and e2` | 与 |
| `!e` / `not e` | 非 |
| `e1 == e2` / `e1 eq e2` | 等于 |
| `e1 != e2` / `e1 neq e2` | 不等于 |
| `e1 lt e2` / `e1 gt e2` | 小于 / 大于 |
| `e1 lte e2` / `e1 gte e2` | 小于等于 / 大于等于 |
| `e1 + e2`、`e1 - e2`、`e1 * e2`、`e1 / e2`、`e1 % e2` | 算术运算 |
| [`e.property`](http://e.property) | 对象属性值（支持多层嵌套） |
| `e.method(args)` | 调用对象方法 |
| `e1[e2]` | 按索引取值（List、Map、数组） |
| `@class@method(args)` | 调用类的**静态方法** |
| `@class@field` | 访问类的**静态字段** |