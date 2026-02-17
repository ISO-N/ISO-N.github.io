---
title: 第1章 MyBatis入门
date: 2026-02-17 15:09:36
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
## 1.1 MyBatis 简介

- MyBatis 是一款支持**自定义 SQL 查询、存储过程和高级映射**的持久层框架
- 前身是 **iBATIS**，原属 Apache，后迁移到 Google Code，2013 年迁移到 **GitHub**
- 消除了几乎所有 JDBC 代码和参数的手动设置以及结果集的检索
- 可以使用 **XML** 或**注解**进行配置和映射
- 工作流程：将参数映射到配置的 SQL → 形成最终执行的 SQL 语句 → 将结果映射成 Java 对象返回

### 与其他 ORM 框架的区别

- 其他 ORM 框架（如 Hibernate）将 **Java 对象与数据库表**关联
- MyBatis 将 **Java 方法与 SQL 语句**关联，属于**半自动化 ORM**框架
- 开发者可以直接编写 SQL，灵活性更高

### 声明式数据缓存

- 当一条 SQL 被标记为"可缓存"后，首次执行从数据库获取数据并存入**高速缓存**
- 后续执行相同语句时直接从缓存读取，不再命中数据库
- MyBatis 提供基于 **HashMap** 的默认缓存，也可集成第三方缓存（如 Ehcache、Redis）

---

## 1.2 创建 Maven 项目

### 项目结构

```
simple
├── src
│   ├── main
│   │   ├── java           ← Java 源代码
│   │   └── resources      ← 配置文件（mybatis-config.xml 等）
│   └── test
│       └── java           ← 测试代码
└── pom.xml
```

### 关键依赖（pom.xml）

```xml
<!-- MyBatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.x</version>
</dependency>

<!-- MySQL 驱动 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.x</version>
</dependency>

<!-- JUnit 测试 -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>

<!-- Log4j 日志 -->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
</dependency>
```

---

## 1.3 简单配置让 MyBatis 跑起来

### 1.3.1 准备数据库

创建一个简单的国家表 `country` 用于入门示例：

```sql
CREATE TABLE country (
    id          INT          NOT NULL AUTO_INCREMENT,
    countryname VARCHAR(255) NULL,
    countrycode VARCHAR(255) NULL,
    PRIMARY KEY (id)
);

INSERT INTO country (countryname, countrycode)
VALUES ('中国', 'CN'),
       ('美国', 'US'),
       ('俄罗斯', 'RU'),
       ('英国', 'GB'),
       ('法国', 'FR');
```

### 1.3.2 配置 MyBatis

在 `src/main/resources` 下创建 **mybatis-config.xml**（文件名无强制要求，整合 Spring 后可省略）：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 配置日志实现 -->
        <setting name="logImpl" value="LOG4J"/>
    </settings>

    <typeAliases>
        <!-- 配置包别名 -->
        <package name="cn.bjut.simple.model"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="UNPOOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value=""/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!-- 注册 Mapper XML -->
        <mapper resource="cn/bjut/simple/mapper/CountryMapper.xml"/>
    </mappers>
</configuration>
```

<aside>
💡

**核心配置说明**

- `settings`：全局设置，如日志实现、缓存、延迟加载等
- `typeAliases`：为 Java 类型设置别名，减少 XML 中的全限定类名书写
- `environments`：数据库环境配置，支持多环境（开发、测试、生产）
- `mappers`：注册 SQL 映射文件的位置
</aside>

### 1.3.3 创建实体类和 Mapper.xml

**实体类** [`Country.java`](http://Country.java)：

```java
public class Country {
    private Long id;
    private String countryname;
    private String countrycode;
    // getter 和 setter 方法省略
}
```

**Mapper XML** `CountryMapper.xml`：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cn.bjut.simple.mapper.CountryMapper">
    <select id="selectAll" resultType="Country">
        SELECT id, countryname, countrycode FROM country
    </select>
</mapper>
```

<aside>
📌

**要点**

- `namespace`：命名空间，推荐与对应的 Mapper 接口全限定名一致
- `resultType`：返回值类型，这里用了别名 `Country`（来自 typeAliases 配置）
- SQL 语句中列名需要与实体类属性名一致（或通过 resultMap 映射）
</aside>

### 1.3.4 编写测试代码

```java
public class CountryMapperTest {
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

    @Test
    public void testSelectAll() {
        // 获取 SqlSession
        SqlSession sqlSession = sqlSessionFactory.openSession();
        try {
            // 通过命名空间 + 方法名调用
            List<Country> countryList = sqlSession.selectList(
                "cn.bjut.simple.mapper.CountryMapper.selectAll");
            printCountryList(countryList);
        } finally {
            // 必须关闭 SqlSession
            sqlSession.close();
        }
    }

    private void printCountryList(List<Country> countryList) {
        for (Country country : countryList) {
            System.out.printf("%-4d%4s%4s\n",
                country.getId(),
                country.getCountryname(),
                country.getCountrycode());
        }
    }
}
```

---

## 本章核心概念总结

| 概念 | 说明 |
| --- | --- |
| **SqlSessionFactory** | 由 `SqlSessionFactoryBuilder` 根据配置文件构建，是创建 SqlSession 的工厂，全局唯一 |
| **SqlSession** | 与数据库交互的会话对象，包含执行 SQL 的所有方法，**非线程安全**，用完必须关闭 |
| **Mapper XML** | 定义 SQL 语句和结果映射的 XML 文件 |
| **namespace** | Mapper XML 的命名空间，用于区分不同 Mapper，推荐与接口全限定名一致 |
| **mybatis-config.xml** | MyBatis 全局配置文件，配置数据源、日志、别名、映射文件等 |