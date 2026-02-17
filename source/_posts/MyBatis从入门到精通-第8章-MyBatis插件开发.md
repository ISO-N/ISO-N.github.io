---
title: 第8章 MyBatis插件开发
date: 2026-02-17 15:09:41
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
## 8.1 拦截器接口介绍

MyBatis 允许在已映射语句执行过程中的某一点进行拦截调用，通过插件机制来实现。插件的核心是 `Interceptor` 接口，该接口定义了以下三个方法：

```java
public interface Interceptor {
    // 执行拦截逻辑的方法
    Object intercept(Invocation invocation) throws Throwable;

    // 用于创建目标对象的代理，可使用 MyBatis 提供的 Plugin.wrap 方法
    Object plugin(Object target);

    // 用于设置在 XML 配置中传递的属性
    void setProperties(Properties properties);
}
```

- **intercept**：插件的核心方法，拦截逻辑写在此方法中。通过 `invocation.proceed()` 调用被拦截的原始方法
- **plugin**：用于生成代理对象。如果被拦截的对象匹配拦截签名，就创建动态代理；否则直接返回目标对象
- **setProperties**：用于在 `mybatis-config.xml` 配置插件时传入自定义属性

## 8.2 可拦截的四大对象

MyBatis 允许拦截以下四个接口中的方法：

| **接口** | **作用** | **可拦截的方法** |
| --- | --- | --- |
| `Executor` | 执行器，负责 SQL 的执行和事务管理 | `update`、`query`、`flushStatements`、`commit`、`rollback`、`getTransaction`、`close`、`isClosed` |
| `ParameterHandler` | 参数处理器，负责设置 SQL 参数 | `getParameterObject`、`setParameters` |
| `ResultSetHandler` | 结果集处理器，负责处理查询结果 | `handleResultSets`、`handleOutputParameters` |
| `StatementHandler` | 语句处理器，负责 JDBC Statement 操作 | `prepare`、`parameterize`、`batch`、`update`、`query` |

## 8.3 插件签名

使用 `@Intercepts` 和 `@Signature` 注解来标识一个拦截器类及其拦截的方法：

```java
@Intercepts({
    @Signature(
        type = ResultSetHandler.class,   // 拦截的接口类型
        method = "handleResultSets",      // 拦截的方法名
        args = {Statement.class}          // 方法参数类型，用于精确匹配重载方法
    )
})
public class MyPlugin implements Interceptor {
    // ...
}
```

- **type**：被拦截的四大接口之一
- **method**：接口中要拦截的方法名
- **args**：被拦截方法的参数类型数组，用于区分重载方法

## 8.4 插件配置

在 `mybatis-config.xml` 中配置插件：

```xml
<plugins>
    <plugin interceptor="com.example.MyPlugin">
        <property name="prop1" value="value1"/>
        <property name="prop2" value="value2"/>
    </plugin>
</plugins>
```

> ⚠️ 多个插件按配置顺序形成拦截链，执行时**后配置的先执行**（类似栈结构）。关闭时**先配置的先执行**。
> 

## 8.5 插件示例一：下画线键值转小驼峰

该插件用于将数据库中下画线风格的列名自动转换为 Java 的小驼峰命名风格，拦截 `ResultSetHandler` 的 `handleResultSets` 方法：

```java
@Intercepts({
    @Signature(
        type = ResultSetHandler.class,
        method = "handleResultSets",
        args = {Statement.class}
    )
})
public class CamelHumpInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 先执行原方法，获取结果
        List<Object> list = (List<Object>) invocation.proceed();
        for (Object object : list) {
            if (object instanceof Map) {
                processMap((Map) object);
            }
        }
        return list;
    }

    // 将 Map 中的下画线 key 转换为驼峰 key
    private void processMap(Map<String, Object> map) {
        Set<String> keySet = new HashSet<>(map.keySet());
        for (String key : keySet) {
            if (key.contains("_")) {
                Object value = map.get(key);
                map.remove(key);
                map.put(underlineToCamelHump(key), value);
            }
        }
    }

    // 下画线转小驼峰
    private String underlineToCamelHump(String input) {
        StringBuilder result = new StringBuilder();
        boolean nextUpperCase = false;
        for (char c : input.toCharArray()) {
            if (c == '_') {
                nextUpperCase = true;
            } else {
                result.append(nextUpperCase ? Character.toUpperCase(c) : c);
                nextUpperCase = false;
            }
        }
        return result.toString();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
    }
}
```

- 适用于 `resultType` 为 `Map` 的查询
- 当 `resultType` 为实体类时，MyBatis 本身的 `mapUnderscoreToCamelCase` 配置更合适

## 8.6 插件示例二：分页插件

分页插件拦截 `Executor` 的 `query` 方法，在 SQL 执行前修改 SQL 语句以实现物理分页：

```java
@Intercepts({
    @Signature(
        type = Executor.class,
        method = "query",
        args = {MappedStatement.class, Object.class,
                RowBounds.class, ResultHandler.class}
    )
})
public class PageInterceptor implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        Object[] args = invocation.getArgs();
        MappedStatement ms = (MappedStatement) args[0];
        Object parameterObject = args[1];
        RowBounds rowBounds = (RowBounds) args[2];

        // 判断是否需要分页（RowBounds 不为默认值时）
        if (rowBounds != RowBounds.DEFAULT) {
            BoundSql boundSql = ms.getBoundSql(parameterObject);
            String sql = boundSql.getSql();
            // 拼接分页 SQL
            String pageSql = sql + " LIMIT " + rowBounds.getLimit()
                             + " OFFSET " + rowBounds.getOffset();
            // 通过反射修改 BoundSql 中的 sql
            Field sqlField = BoundSql.class.getDeclaredField("sql");
            sqlField.setAccessible(true);
            sqlField.set(boundSql, pageSql);
            // 重置 RowBounds 避免内存分页
            args[2] = RowBounds.DEFAULT;
        }
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
    }
}
```

<aside>
💡

**物理分页 vs 逻辑分页**

- **逻辑分页**：MyBatis 默认的 `RowBounds` 方式，先查出所有数据再在内存中截取，数据量大时性能差
- **物理分页**：通过插件改写 SQL（如加 `LIMIT`），在数据库层面分页，推荐用于生产环境
</aside>

## 8.7 插件开发的注意事项

- **不要轻易修改核心行为**：插件可以改变 MyBatis 底层的运行机制，使用不当可能导致不可预期的问题
- **理解被拦截方法的参数和含义**：需要深入了解被拦截对象的源码
- **尽量使用 `Plugin.wrap` 方法**：它会自动判断是否需要创建代理，避免不必要的代理嵌套
- **多插件的执行顺序**：多个插件按配置的相反顺序执行拦截逻辑，需注意插件之间的依赖关系
- **实际开发中推荐使用成熟的开源插件**，如 PageHelper（分页插件）、MyBatis-Plus（增强工具包）等，避免重复造轮子