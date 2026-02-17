---
title: 第5章 MyBatis代码生成器
date: 2026-02-17 15:09:39
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
MyBatis Generator（MBG）是 MyBatis 官方提供的**代码生成器**，可以根据数据库表自动生成对应的**实体类、Mapper 接口、Mapper XML 文件和 Example 对象**，几乎包含了全部的单表操作方法，大幅减少重复的 CRUD 样板代码。

<aside>
💡

MBG 解决的是简单的单表 CRUD 操作，**联合查询和存储过程**仍需手写 SQL。

</aside>

---

## 5.1 XML 配置详解

MBG 的配置以 XML 形式编写，文件头和根节点如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 配置内容 -->
</generatorConfiguration>
```

`generatorConfiguration` 根节点下有 **3 个子标签**，必须严格按以下顺序配置：

| 标签 | 数量 | 说明 |
| --- | --- | --- |
| `properties` | 0 或 1 个 | 引入外部属性文件，可用 `${property}` 引用属性值（如 JDBC 信息） |
| `classPathEntry` | 0 或多个 | 通过 `location` 属性指定 JDBC 驱动 jar 路径 |
| `context` | 至少 1 个 | **核心标签**，配置代码生成的详细规则 |

### properties 标签

- 包含 `resource` 和 `url` 两个属性，**只能使用其中一个**
- `resource`：指定 classpath 下的属性文件，如 `com/myproject/[generatorConfig.properties](http://generatorConfig.properties)`
- `url`：指定文件系统路径，如 [`file:///C:/myfolder/generatorConfig.properties`](file:///C:/myfolder/generatorConfig.properties)

### classPathEntry 标签

最常见的用法是指定数据库驱动路径：

```xml
<classPathEntry location="E:\mysql\mysql-connector-java-5.1.29.jar"/>
```

### context 标签

`context` 标签是重点，常见配置如下：

```xml
<context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
```

- `id`：唯一标识，多个 context 时用于区分
- `targetRuntime`：生成代码的运行时环境
    - **MyBatis3**：生成包含 Example 相关对象和方法的完整代码
    - **MyBatis3Simple**：不生成 Example 相关代码，更简洁
- `defaultModelType`：推荐使用 **flat**，保证一个表对应一个实体类

---

### 5.1.1 property 标签

配置 context 的全局属性，常用于设置**分隔符**和**编码**：

```xml
<property name="autoDelimitKeywords" value="true"/>
<property name="beginningDelimiter" value="`"/>
<property name="endingDelimiter" value="`"/>
<property name="javaFileEncoding" value="UTF-8"/>
```

<aside>
📌

**分隔符的作用**：当 SQL 中有数据库关键字时，使用反单引号括住可避免错误。MBG 内部维护了关键字列表（见 `org.mybatis.generator.internal.db.SqlReservedWords` 类），会自动为匹配的字段或表添加分隔符。

</aside>

### 5.1.2 plugin 标签

- 可配置 **0 个或多个**，用于扩展或修改 MBG 生成的代码
- 插件按配置顺序依次执行
- 常见内置插件：
    - **CachePlugin**（缓存插件）：在 XML 映射文件中增加 `<cache>` 标签，仅 `targetRuntime="MyBatis3"` 有效
    - **SerializablePlugin**（序列化插件）：让实体类实现 `Serializable` 接口
    - **RowBoundsPlugin**（分页插件）
    - **ToStringPlugin**：生成 `toString()` 方法

### 5.1.3 commentGenerator 标签

- 配置如何生成注释信息，最多配置 **1 个**
- MBG 默认生成的注释实用价值不大，且带时间戳会导致版本控制频繁提交
- 推荐屏蔽注释的配置：

```xml
<commentGenerator>
    <property name="suppressDate" value="true"/>
    <property name="addRemarkComments" value="true"/>
</commentGenerator>
```

- `suppressDate`：阻止生成日期时间戳
- `addRemarkComments`：添加数据库表的备注信息作为注释

### 5.1.4 jdbcConnection 标签

- **必选标签**，且只能有一个，用于指定 MBG 连接的数据库信息

```xml
<jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
    connectionURL="jdbc:mysql://localhost:3306/mybatis?serverTimezone=Asia/Shanghai"
    userId="root"
    password="">
</jdbcConnection>
```

### 5.1.5 javaTypeResolver 标签

- 最多配置 **1 个**，用于指定 JDBC 类型和 Java 类型的转换规则
- 提供可选属性 `type`，一般使用默认实现即可（DEFAULT）
- 常用 property：`forceBigDecimals`，设置为 `true` 时将 DECIMAL/NUMERIC 类型强制转为 `BigDecimal`

### 5.1.6 javaModelGenerator 标签

- **必选标签**，且最多配置 1 个，用于配置实体类生成规则

```xml
<javaModelGenerator targetPackage="test.model" targetProject="src\main\java">
    <property name="trimStrings" value="true"/>
    <property name="rootClass" value="tk.mybatis.simple.model.BaseEntity"/>
</javaModelGenerator>
```

- `targetPackage`：生成实体类的目标包名
- `targetProject`：生成文件的目标项目路径
- `trimStrings`：setter 方法中是否对字符串类型进行 `trim()` 操作
- `rootClass`：设置所有实体类的公共基类

### 5.1.7 sqlMapGenerator 标签

- 可选标签，最多配置 1 个，用于配置 **Mapper.xml 文件**的生成

```xml
<sqlMapGenerator targetPackage="test.xml" targetProject="src\main\resources"/>
```

- 当 `targetRuntime` 为 `MyBatis3` 且 `javaClientGenerator` 需要 XML 时，此标签必须配置
- 可选 property `enableSubPackages`：为 `true` 时按 catalog 和 schema 生成子包

### 5.1.8 javaClientGenerator 标签

- 可选标签，最多配置 1 个，用于配置 **Mapper 接口**的生成
- 不配置则不会生成 Mapper 接口

```xml
<javaClientGenerator type="XMLMAPPER" targetPackage="test.dao"
    targetProject="src\main\java"/>
```

- `type` 常用值：
    - **XMLMAPPER**：接口方法的实现在 XML 中
    - **ANNOTATEDMAPPER**：使用注解方式

### 5.1.9 table 标签

- **至少配置 1 个**，可配置多个。只有配置过的表才会生成代码
- 必选属性 `tableName`，可用 SQL 通配符 `%` 匹配多个表

```xml
<table tableName="%">
    <generatedKey column="id" sqlStatement="MySql"/>
</table>
```

table 标签包含以下**子标签**：

#### 5.1.9.1 generatedKey 标签

- 指定自动生成主键的属性（identity 字段或 sequences 序列）
- MBG 会在 insert 的 SQL 映射中插入 `<selectKey>` 标签
- 最多配置 **1 个**
- 必选属性：
    - `column`：主键列名
    - `sqlStatement`：返回新值的 SQL 语句，支持预定义值如 `MySql`、`SqlServer`、`DB2` 等

```xml
<generatedKey column="id" sqlStatement="MySql"/>
```

#### 5.1.9.2 columnRenamingRule 标签

- 最多配置 **1 个**，用于在生成列之前对列进行**重命名**
- 适用于去除统一前缀的场景，例如列名 `CUST_BUSINESS_NAME`、`CUST_CITY` 等：

```xml
<columnRenamingRule searchString="^CUST_" replaceString=""/>
```

- 内部使用 `java.util.regex.Matcher.replaceAll` 实现
- 当 `columnOverride` 匹配某列时，`columnRenamingRule` 会被忽略

#### 5.1.9.3 columnOverride 标签

- 可配置 **0 个或多个**，用于将默认计算的属性值更改为指定值
- 必选属性 `column`（列名）
- 可选属性：
    - `property`：Java 属性名
    - `javaType`：完全限定 Java 类型
    - `jdbcType`：JDBC 类型（如 INTEGER、VARCHAR 等）
    - `typeHandler`：自定义类型处理器的全限定类名
    - `delimitedColumnName`：是否在列名上增加分隔符

#### 5.1.9.4 ignoreColumn 标签

- 可配置 **0 个或多个**，用于**屏蔽不需要生成的列**
- 必选属性 `column`（要忽略的列名）
- 可选属性 `delimitedColumnName`：匹配时是否区分大小写（默认 `false`，不区分）

---

## 5.2 一个配置参考示例

在 `src/main/resources/generator` 目录下创建 `generatorConfig.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <properties resource="jdbc.properties"/>

    <!-- targetRuntime="MyBatis3Simple" 不会生成 Example 相关代码 -->
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <property name="autoDelimitKeywords" value="true"/>
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>
        <property name="javaFileEncoding" value="UTF-8"/>

        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="addRemarkComments" value="true"/>
        </commentGenerator>

        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
            connectionURL="jdbc:mysql://localhost:3306/mybatis"
            userId="${jdbc.username}"
            password="${jdbc.password}">
        </jdbcConnection>

        <!-- 实体类 -->
        <javaModelGenerator targetPackage="test.model"
            targetProject="src\main\java">
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- XML 文件 -->
        <sqlMapGenerator targetPackage="test.xml"
            targetProject="src\main\resources"/>

        <!-- Mapper 接口 -->
        <javaClientGenerator type="XMLMAPPER"
            targetPackage="test.dao"
            targetProject="src\main\java"/>

        <!-- 生成所有表 -->
        <table tableName="%">
            <generatedKey column="id" sqlStatement="MySql"/>
        </table>
    </context>
</generatorConfiguration>
```

---

## 5.3 运行 MyBatis Generator

### 5.3.1 使用 Java 编写代码运行（推荐）

- 这种方式出现问题最少，配置最容易，**推荐使用**
- 缺点：与当前项目绑定在一起
- `generatorConfig.xml` 配置的特殊类，只要在当前项目或 classpath 中即可直接使用

```java
public class Generator {
    public static void main(String[] args) throws Exception {
        List<String> warnings = new ArrayList<>();
        boolean overwrite = true;
        // 读取 MBG 配置文件
        InputStream is = Generator.class.getResourceAsStream(
            "/generator/generatorConfig.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is);
        is.close();

        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator generator = new MyBatisGenerator(
            config, callback, warnings);
        generator.generate(null);
        // 输出警告信息
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }
}
```

### 5.3.2 使用 Maven Plugin 运行

在 `pom.xml` 中添加 MBG 的 Maven 插件，然后通过 Maven 命令运行：

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.7</version>
    <configuration>
        <configurationFile>
            src/main/resources/generator/generatorConfig.xml
        </configurationFile>
        <overwrite>true</overwrite>
        <verbose>true</verbose>
    </configuration>
</plugin>
```

运行命令：`mvn mybatis-generator:generate`

### 5.3.3 使用命令行运行

```
java -jar mybatis-generator-core-x.x.x.jar -configfile generatorConfig.xml -overwrite
```

### 5.3.4 使用 Eclipse 插件运行

- 下载 MBG 的 Eclipse 插件后，通过 Help → Install New Software 安装
- Eclipse 插件方式的特殊之处：JDBC 驱动需要通过 `classPathEntry` 配置
- 右键 `generatorConfig.xml` → Run As → Run MyBatis Generator 即可运行

---

## 5.4 Example 介绍

当 `context` 的 `targetRuntime` 配置为 **MyBatis3** 时，MBG 会生成与 **Example** 相关的对象和方法。Example 用于**构造动态查询条件**，支持复杂的单表操作。

<aside>
⚡

如果 `targetRuntime` 设置为 `MyBatis3Simple`，则**不会**生成 Example 相关代码。

</aside>

### Example 生成的核心方法

| 方法 | 说明 |
| --- | --- |
| `selectByExample` | 根据条件查询，返回列表 |
| `updateByExample` | 根据条件更新，**空属性会被更新为 null** |
| `updateByExampleSelective` | 根据条件更新，**空属性不会更新**（推荐） |
| `deleteByExample` | 根据条件删除 |
| `countByExample` | 根据条件统计数量 |

### 基本用法示例

```java
@Test
public void testExample() {
    SqlSession sqlSession = getSqlSession();
    try {
        CountryMapper countryMapper = sqlSession.getMapper(CountryMapper.class);
        // 创建 Example 对象
        CountryExample example = new CountryExample();
        // 设置排序规则
        example.setOrderByClause("id desc, countryname asc");
        // 设置去重
        example.setDistinct(true);

        // 创建条件（只能有一个 createCriteria）
        CountryExample.Criteria criteria = example.createCriteria();
        criteria.andIdGreaterThanOrEqualTo(1);   // id >= 1
        criteria.andIdLessThan(4);                // id < 4
        criteria.andCountrycodeLike("%U%");       // countrycode like '%U%'

        // or 条件（可以有多个 or）
        CountryExample.Criteria or = example.or();
        or.andCountrynameEqualTo("中国");          // countryname = '中国'

        // 执行查询
        List<Country> countryList = countryMapper.selectByExample(example);
        printCountryList(countryList);
    } finally {
        sqlSession.close();
    }
}
```

生成的 SQL：

```sql
SELECT DISTINCT id, countryname, countrycode
FROM country
WHERE (id >= ? AND id < ? AND countrycode LIKE ?)
   OR (countryname = ?)
ORDER BY id DESC, countryname ASC
```

<aside>
⚠️

**注意事项**

- `like` 查询必须**自己写通配符** `%`，MBG 不会自动添加
- 多个 `or()` 会生成 `OR (...) OR (...)` 这样的 SQL
- 如果没有 `or()`，则只有 `createCriteria()` 中的条件
</aside>

### updateByExampleSelective 示例

```java
@Test
public void testUpdateByExampleSelective() {
    SqlSession sqlSession = getSqlSession();
    try {
        CountryMapper countryMapper = sqlSession.getMapper(CountryMapper.class);
        CountryExample example = new CountryExample();
        CountryExample.Criteria criteria = example.createCriteria();
        criteria.andIdGreaterThan(2);  // 更新所有 id > 2 的记录

        Country country = new Country();
        country.setCountryname("China");  // 只更新 countryname 字段

        countryMapper.updateByExampleSelective(country, example);
    } finally {
        sqlSession.close();
    }
}
```

### deleteByExample 和 countByExample 示例

```java
@Test
public void testDeleteByExample() {
    SqlSession sqlSession = getSqlSession();
    try {
        CountryMapper countryMapper = sqlSession.getMapper(CountryMapper.class);
        CountryExample example = new CountryExample();
        CountryExample.Criteria criteria = example.createCriteria();
        criteria.andIdGreaterThan(2);  // 删除所有 id > 2 的记录

        countryMapper.deleteByExample(example);
        // 验证删除结果
        Assert.assertEquals(0, countryMapper.countByExample(example));
    } finally {
        sqlSession.close();
    }
}
```

---

## 本章核心要点总结

| 要点 | 说明 |
| --- | --- |
| **MBG 的作用** | 根据数据库表自动生成实体类、Mapper 接口、Mapper XML 和 Example 对象 |
| **targetRuntime** | `MyBatis3` 生成完整代码含 Example；`MyBatis3Simple` 不含 Example |
| **defaultModelType** | 推荐 `flat`，一个表对应一个实体类 |
| **推荐运行方式** | 使用 Java 编写代码运行，问题最少、配置最容易 |
| **Example 的定位** | 适合单表的复杂查询；条件过多时建议使用 XML 方式更高效 |
| **Selective 方法** | `updateByExampleSelective` 不会更新 null 属性字段，更安全 |