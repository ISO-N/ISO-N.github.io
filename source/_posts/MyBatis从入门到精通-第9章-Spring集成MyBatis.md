---
title: 第9章 Spring集成MyBatis
date: 2026-02-17 15:09:42
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
## 9.1 创建基本的 Maven Web 项目

本章的示例项目是一个基于 Maven 的标准 Web 项目，用于演示 Spring 与 MyBatis 的集成。

### 项目结构

```
mybatis-spring
├── src
│   ├── main
│   │   ├── java              ← Java 源代码
│   │   ├── resources          ← 配置文件
│   │   └── webapp
│   │       └── WEB-INF
│   │           ├── jsp        ← JSP 视图文件
│   │           └── web.xml    ← Web 应用部署描述符
│   └── test
│       └── java               ← 测试代码
└── pom.xml
```

### 关键依赖（pom.xml）

```xml
<!-- Spring 核心 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
</dependency>

<!-- MyBatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
</dependency>

<!-- MyBatis-Spring 集成包 -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
</dependency>

<!-- 数据库驱动 & 数据源 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
</dependency>

<!-- Servlet & JSP -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet.jsp</groupId>
    <artifactId>jsp-api</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>
```

<aside>
💡

**关键依赖说明**

- `mybatis-spring`：MyBatis 官方提供的 Spring 集成包，是本章的核心依赖
- `spring-jdbc`：提供数据源管理和事务支持
- `druid`：阿里巴巴开源的数据库连接池，也可替换为 DBCP、C3P0 等
</aside>

---

## 9.2 集成 Spring 和 Spring MVC

### 配置 web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         version="3.1">

    <!-- Spring 根上下文配置 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

    <!-- Spring MVC 前端控制器 -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>
            org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- 字符编码过滤器 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>
            org.springframework.web.filter.CharacterEncodingFilter
        </filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

### Spring MVC 配置（spring-mvc.xml）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="...">

    <!-- 扫描 Controller -->
    <context:component-scan base-package="cn.bjut.web.controller"/>

    <!-- 启用注解驱动 -->
    <mvc:annotation-driven/>

    <!-- 视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!-- 静态资源处理 -->
    <mvc:default-servlet-handler/>
</beans>
```

### Spring 根上下文配置（applicationContext.xml）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="...">

    <!-- 扫描 Service 层 -->
    <context:component-scan base-package="cn.bjut.web.service"/>

    <!-- 导入 MyBatis 配置 -->
    <import resource="classpath:mybatis-config-spring.xml"/>
</beans>
```

<aside>
📌

**两个上下文的分工**

- **Root WebApplicationContext**（`applicationContext.xml`）：管理 Service、Dao、数据源等业务 Bean
- **Servlet WebApplicationContext**（`spring-mvc.xml`）：管理 Controller、视图解析器等 Web 层 Bean
- 子上下文（MVC）可以访问父上下文（Root）中的 Bean，反之不行
</aside>

---

## 9.3 集成 MyBatis

这是本章的**核心内容**。通过 `mybatis-spring` 集成包，将 MyBatis 纳入 Spring 容器管理。

### MyBatis-Spring 配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="...">

    <!-- 1. 配置数据源 -->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          init-method="init" destroy-method="close">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
        <property name="username" value="root"/>
        <property name="password" value=""/>
    </bean>

    <!-- 2. 配置 SqlSessionFactoryBean -->
    <bean id="sqlSessionFactory"
          class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 指定 MyBatis 全局配置文件（可选） -->
        <property name="configLocation"
                  value="classpath:mybatis-config.xml"/>
        <!-- 指定 Mapper XML 文件位置 -->
        <property name="mapperLocations"
                  value="classpath:cn/bjut/web/mapper/*.xml"/>
        <!-- 类型别名包 -->
        <property name="typeAliasesPackage"
                  value="cn.bjut.web.model"/>
    </bean>

    <!-- 3. 配置 MapperScannerConfigurer -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cn.bjut.web.mapper"/>
        <property name="sqlSessionFactoryBeanName"
                  value="sqlSessionFactory"/>
    </bean>

    <!-- 4. 配置事务管理器 -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
</beans>
```

### 三大核心组件详解

| **组件** | **作用** | **说明** |
| --- | --- | --- |
| `SqlSessionFactoryBean` | 创建 `SqlSessionFactory` | 替代手动通过 `SqlSessionFactoryBuilder` 构建；可配置数据源、Mapper 位置、别名包、插件等 |
| `MapperScannerConfigurer` | 自动扫描 Mapper 接口 | 扫描指定包下的所有接口，自动注册为 Spring Bean，无需逐个手动配置 |
| `DataSourceTransactionManager` | 管理数据库事务 | 将 MyBatis 的事务交给 Spring 统一管理，支持声明式事务（`@Transactional`） |

<aside>
💡

**SqlSessionFactoryBean 常用属性**

- `dataSource`：数据源（**必需**）
- `configLocation`：MyBatis 全局配置文件路径（可选，部分配置可直接在 Spring 中完成）
- `mapperLocations`：Mapper XML 文件路径，支持 Ant 风格通配符（如 `classpath:mapper/*.xml`）
- `typeAliasesPackage`：类型别名扫描包
- `plugins`：MyBatis 插件数组
</aside>

<aside>
📌

**MapperScannerConfigurer 注意事项**

- 使用 `sqlSessionFactoryBeanName`（Bean **名称**，字符串）而非 `sqlSessionFactory`（Bean **引用**）
- 原因：`MapperScannerConfigurer` 是 `BeanDefinitionRegistryPostProcessor`，在 Spring 生命周期中**早于属性占位符解析**执行。如果使用 Bean 引用，可能导致数据源中的占位符（如 `${jdbc.url}`）无法被正确替换
</aside>

---

## 9.4 几个简单实例

### 9.4.1 基本准备

使用前面章节中的权限管理数据库（包含 `sys_user`、`sys_role`、`sys_privilege` 等表），复用已有的实体类和 Mapper 接口。

### 9.4.2 开发 Mapper 层（Dao 层）

Mapper 接口定义与之前章节一致，但在 Spring 集成后**无需手动获取 SqlSession**，由 Spring 自动注入代理对象：

```java
public interface SysUserMapper {
    // 根据用户 id 查询
    SysUser selectById(Long id);

    // 查询所有用户
    List<SysUser> selectAll();
}
```

对应的 Mapper XML：

```xml
<mapper namespace="cn.bjut.web.mapper.SysUserMapper">
    <select id="selectById" resultType="SysUser">
        SELECT * FROM sys_user WHERE id = #{id}
    </select>

    <select id="selectAll" resultType="SysUser">
        SELECT * FROM sys_user
    </select>
</mapper>
```

<aside>
💡

**与纯 MyBatis 的区别**

- 不再需要手动创建 `SqlSession`、手动提交/回滚事务、手动关闭 `SqlSession`
- `MapperScannerConfigurer` 自动为接口创建代理实现并注册为 Spring Bean
- 直接通过 `@Autowired` 注入 Mapper 即可使用
</aside>

### 9.4.3 开发业务层（Service 层）

```java
@Service
public class SysUserService {

    @Autowired
    private SysUserMapper sysUserMapper;

    public SysUser findById(Long id) {
        return sysUserMapper.selectById(id);
    }

    public List<SysUser> findAll() {
        return sysUserMapper.selectAll();
    }

    @Transactional
    public void saveUser(SysUser user) {
        sysUserMapper.insert(user);
        // 在事务中，若抛出异常会自动回滚
    }
}
```

- 使用 `@Service` 注解将类注册为 Spring Bean
- 使用 `@Autowired` 自动注入 Mapper
- 使用 `@Transactional` 开启声明式事务

### 9.4.4 开发控制层（Controller 层）

```java
@Controller
public class SysUserController {

    @Autowired
    private SysUserService sysUserService;

    @RequestMapping("/users")
    public String userList(Model model) {
        List<SysUser> userList = sysUserService.findAll();
        model.addAttribute("userList", userList);
        return "userList";  // 返回视图名，解析为 /WEB-INF/jsp/userList.jsp
    }

    @RequestMapping("/user/{id}")
    public String userInfo(@PathVariable Long id, Model model) {
        SysUser user = sysUserService.findById(id);
        model.addAttribute("user", user);
        return "userInfo";
    }
}
```

### 9.4.5 开发视图层（View 层）

使用 JSP + JSTL 展示数据，例如用户列表页面 `userList.jsp`：

```html
<%@ page contentType="text/html;charset=UTF-8" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head><title>用户列表</title></head>
<body>
    <h2>用户列表</h2>
    <table border="1">
        <tr>
            <th>ID</th>
            <th>用户名</th>
            <th>邮箱</th>
        </tr>
        <c:forEach items="${userList}" var="user">
            <tr>
                <td>${user.id}</td>
                <td>${user.userName}</td>
                <td>${user.userEmail}</td>
            </tr>
        </c:forEach>
    </table>
</body>
</html>
```

### 9.4.6 部署和运行应用

- 配置 Tomcat 或使用 Maven 插件（如 `tomcat7-maven-plugin`）运行项目
- 访问 [`http://localhost:8080/users`](http://localhost:8080/users) 即可查看用户列表
- 整个请求链路：**浏览器 → DispatcherServlet → Controller → Service → Mapper → 数据库 → 视图渲染**

---

## 本章核心概念总结

| 概念 | 说明 |
| --- | --- |
| **mybatis-spring** | MyBatis 官方提供的 Spring 集成包，核心类有 `SqlSessionFactoryBean`、`MapperScannerConfigurer`、`SqlSessionTemplate` 等 |
| **SqlSessionFactoryBean** | Spring 工厂 Bean，用于创建 `SqlSessionFactory`，替代手动 `SqlSessionFactoryBuilder` |
| **MapperScannerConfigurer** | 自动扫描指定包下的 Mapper 接口并注册为 Spring Bean，避免逐个手动注册 |
| **声明式事务** | 通过 `@Transactional` 注解 + `DataSourceTransactionManager`，将事务管理从代码中解耦 |
| **分层架构** | Mapper（数据访问）→ Service（业务逻辑）→ Controller（请求处理）→ View（视图渲染），各层职责分明 |