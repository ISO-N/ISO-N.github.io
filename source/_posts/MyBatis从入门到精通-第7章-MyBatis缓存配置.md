---
title: 第7章 MyBatis缓存配置
date: 2026-02-17 15:09:40
categories:
- [编程技术, MyBatis]
tags:
- MyBatis从入门到精通
---
## 7.1 一级缓存

一级缓存也叫**本地缓存**（Local Cache），MyBatis 的一级缓存默认开启，存在于 `SqlSession` 的生命周期中。

### 基本概念

- 一级缓存基于 `PerpetualCache` 的 **HashMap** 实现
- 作用域为 **SqlSession** 级别，同一个 SqlSession 中执行相同的查询会直接从缓存中获取
- 当调用 SqlSession 的 `close()` 方法时，一级缓存会被清空

### 一级缓存失效的情况

1. 不同的 SqlSession 对应不同的一级缓存
2. 同一个 SqlSession，但查询条件不同
3. 同一个 SqlSession 中，两次相同查询之间执行了 **增删改操作**（`insert`、`update`、`delete`）
4. 手动调用 `SqlSession.clearCache()` 清空缓存

### 一级缓存配置

在 `mybatis-config.xml` 中可通过 `localCacheScope` 设置一级缓存作用域：

```xml
<settings>
    <!-- SESSION（默认）：在同一个 SqlSession 中共享 -->
    <!-- STATEMENT：缓存仅对当前执行的语句有效，语句执行完后缓存立即清空 -->
    <setting name="localCacheScope" value="SESSION"/>
</settings>
```

> **注意：** 在分布式环境或多个 SqlSession 并发操作下，一级缓存可能导致脏数据，建议设置为 `STATEMENT`。
> 

---

## 7.2 二级缓存

二级缓存是 **Mapper（namespace）级别** 的缓存，多个 SqlSession 可以共享同一个 Mapper 的二级缓存。

### 二级缓存的特点

- 二级缓存默认**关闭**，需要手动开启
- 作用域为同一个 `namespace` 下的所有 SqlSession
- 查询顺序：**二级缓存 → 一级缓存 → 数据库**
- SqlSession 关闭或提交后，一级缓存中的数据会刷新到二级缓存

### 开启二级缓存的步骤

**第一步：** 在 `mybatis-config.xml` 中开启全局二级缓存开关

```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

**第二步：** 在 Mapper XML 映射文件中添加 `<cache/>` 标签

```xml
<mapper namespace="com.example.mapper.UserMapper">
    <cache/>
    <!-- SQL 映射语句 -->
</mapper>
```

**第三步：** POJO 类实现 `Serializable` 接口

```java
public class SysUser implements Serializable {
    private static final long serialVersionUID = 1L;
    // ...
}
```

### `<cache/>` 标签的属性

```xml
<cache
    eviction="LRU"
    flushInterval="60000"
    size="512"
    readOnly="true"/>
```

| 属性 | 说明 | 默认值 |
| --- | --- | --- |
| `eviction` | 缓存回收策略 | `LRU` |
| `flushInterval` | 刷新间隔（毫秒），不设置则仅在增删改时刷新 | 无 |
| `size` | 缓存引用数量 | `1024` |
| `readOnly` | 是否只读。`true` 返回缓存对象本身（性能好），`false` 返回序列化副本（安全） | `false` |

### 缓存回收策略（eviction）

- **LRU**（Least Recently Used）：移除最近最少使用的对象（默认）
- **FIFO**（First In First Out）：按先进先出顺序移除
- **SOFT**：基于软引用移除对象，当内存不足时回收
- **WEAK**：基于弱引用移除对象，更积极地回收

### 方法级别的缓存控制

可以在单个 `<select>` 语句上通过 `useCache` 控制是否使用二级缓存：

```xml
<select id="selectById" useCache="true" resultType="SysUser">
    SELECT * FROM sys_user WHERE id = #{id}
</select>
```

也可以通过 `flushCache` 控制是否在执行后清空缓存：

```xml
<!-- 增删改默认 flushCache="true"，查询默认 flushCache="false" -->
<select id="selectById" flushCache="false" resultType="SysUser">
    SELECT * FROM sys_user WHERE id = #{id}
</select>
```

---

## 7.3 集成 EhCache 缓存

MyBatis 提供了 `Cache` 接口，方便集成第三方缓存框架。EhCache 是一个广泛使用的 Java 分布式缓存。

### 添加依赖

```xml
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.2.1</version>
</dependency>
```

### 配置 Mapper 使用 EhCache

在 Mapper XML 中将 `<cache>` 的 `type` 指定为 EhCache 实现类：

```xml
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

### 配置 ehcache.xml

在 classpath 下创建 `ehcache.xml` 配置文件：

```xml
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="ehcache.xsd">
    <defaultCache
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        overflowToDisk="false"
        diskPersistent="false"
        memoryStoreEvictionPolicy="LRU"/>
</ehcache>
```

| 属性 | 说明 |
| --- | --- |
| `maxElementsInMemory` | 内存中最大缓存对象数 |
| `eternal` | 是否永不过期 |
| `timeToIdleSeconds` | 对象空闲多久后过期 |
| `timeToLiveSeconds` | 对象存活最长时间 |
| `overflowToDisk` | 超过内存限制时是否写入磁盘 |
| `memoryStoreEvictionPolicy` | 内存回收策略（LRU / FIFO / LFU） |

---

## 7.4 集成 Redis 缓存

在分布式环境下，本地缓存无法跨实例共享，需要使用 Redis 等集中式缓存。

### 添加依赖

```xml
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-redis</artifactId>
    <version>1.0.0-beta2</version>
</dependency>
```

### 配置 Mapper 使用 Redis

```xml
<cache type="org.mybatis.caches.redis.RedisCache"/>
```

### 配置 [redis.properties](http://redis.properties)

在 classpath 下创建 [`redis.properties`](http://redis.properties)：

```
redis.host=localhost
redis.port=6379
redis.connectionTimeout=5000
redis.password=
redis.database=0
```

---

## 7.5 二级缓存适用场景与注意事项

### 适用场景

- 查询频率远高于增删改的表
- 对数据实时性要求不高的场景
- 单表操作的 Mapper，避免多表关联产生脏数据

### 注意事项

- **脏数据问题：** 多表查询时，如果关联表在另一个 namespace 中被修改，当前 namespace 的缓存不会被刷新，可能产生脏数据
- **解决方案：** 使用 `<cache-ref>` 引用同一个缓存命名空间

```xml
<!-- 在 UserMapper.xml 中引用 RoleMapper 的缓存 -->
<cache-ref namespace="com.example.mapper.RoleMapper"/>
```

- **频繁增删改：** 如果增删改操作频繁，二级缓存会不断被清空，形同虚设，反而增加开销
- **分布式环境：** 默认的二级缓存是本地缓存，分布式部署下必须使用 Redis、EhCache 等集中式缓存

### 一级缓存 vs 二级缓存总结

| 对比项 | 一级缓存 | 二级缓存 |
| --- | --- | --- |
| 作用域 | SqlSession 级别 | Mapper（namespace）级别 |
| 默认状态 | 默认开启 | 默认关闭，需手动开启 |
| 生命周期 | 与 SqlSession 一致 | 与应用程序一致 |
| 共享范围 | 同一 SqlSession 内有效 | 多个 SqlSession 共享 |
| 实现方式 | HashMap | PerpetualCache + 可扩展第三方缓存 |
| 序列化要求 | 不需要 | 需实现 Serializable 接口 |