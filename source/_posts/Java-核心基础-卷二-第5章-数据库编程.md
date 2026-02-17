---
title: 第5章 数据库编程
date: 2026-02-17 15:12:03
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 5.1 JDBC 的设计

- JDBC（Java Database Connectivity）是 Java 访问关系数据库的标准 API
- JDBC 采用**驱动程序管理器**来管理不同数据库的驱动，开发者只需面向 JDBC API 编程，无需关心底层数据库差异
- 常见 JDBC 驱动类型：
    - **Type 1**：JDBC-ODBC 桥（已在 Java 8 中移除）
    - **Type 2**：本地 API 驱动（部分 Java，部分本地代码）
    - **Type 3**：网络协议驱动（纯 Java，通过中间件访问数据库）
    - **Type 4**：本地协议驱动（纯 Java，直连数据库，**最常用**）

---

## 5.2 JDBC URL 与数据库连接

- JDBC URL 格式：`jdbc:子协议:数据源标识`
- 常见示例：

```
jdbc:mysql://localhost:3306/mydb
jdbc:postgresql://host/dbname
jdbc:sqlite:sample.db
jdbc:oracle:thin:@host:1521:sid
```

- 获取连接的基本方式：

```java
// 1. 注册驱动（JDBC 4.0+ 可自动发现，通常无需手动注册）
// Class.forName("com.mysql.cj.jdbc.Driver");

// 2. 获取连接
String url = "jdbc:mysql://localhost:3306/mydb";
Connection conn = DriverManager.getConnection(url, "user", "password");
```

- 推荐使用 **try-with-resources** 自动关闭连接

---

## 5.3 执行 SQL 语句

### Statement

```java
try (Statement stat = conn.createStatement()) {
    // 执行查询
    ResultSet rs = stat.executeQuery("SELECT * FROM books");
    // 执行更新（INSERT / UPDATE / DELETE）
    int count = stat.executeUpdate("DELETE FROM books WHERE id = 1");
    // execute() 可执行任意 SQL，返回 boolean 表示是否有 ResultSet
}
```

### 处理 ResultSet

```java
while (rs.next()) {
    String title = rs.getString("title");
    double price = rs.getDouble("price");
    int id = rs.getInt(1); // 也可以用列索引（从1开始）
}
```

- `ResultSet` 的 `getXxx()` 返回值为 `null` 时，数值类型返回 0，需用 `rs.wasNull()` 判断

---

## 5.4 PreparedStatement（预编译语句）

- 使用 `?` 作为占位符，**防止 SQL 注入**，且性能更优

```java
String sql = "SELECT * FROM books WHERE author = ? AND price < ?";
try (PreparedStatement pstat = conn.prepareStatement(sql)) {
    pstat.setString(1, "Cay Horstmann");
    pstat.setDouble(2, 50.0);
    ResultSet rs = pstat.executeQuery();
}
```

<aside>
⚠️

**永远不要用字符串拼接构建 SQL**，应始终使用 `PreparedStatement` 来避免 SQL 注入攻击。

</aside>

---

## 5.5 读写 LOB（大对象）

- **BLOB**（Binary Large Object）：存储二进制数据（如图片）
- **CLOB**（Character Large Object）：存储大文本数据

```java
// 读取 BLOB
Blob blob = rs.getBlob("cover");
InputStream in = blob.getBinaryStream();

// 写入 BLOB
PreparedStatement pstat = conn.prepareStatement(
    "INSERT INTO covers VALUES (?, ?)");
pstat.set(1, isbn);
pstat.setBlob(2, inputStream, length);
```

---

## 5.6 SQL 转义语法

- JDBC 提供转义语法来处理不同数据库的差异：
    - 日期字面量：`{d '2026-02-13'}`
    - 时间字面量：`{t '12:30:00'}`
    - 时间戳：`{ts '2026-02-13 12:30:00'}`
    - 调用标量函数：`{fn LEFT(title, 3)}`
    - 调用存储过程：`{call PROC_NAME(?, ?)}`
    - 外连接：`{oj ...}`

---

## 5.7 多结果集与生成键

### 获取自动生成的键

```java
stat.executeUpdate(insertSQL, Statement.RETURN_GENERATED_KEYS);
ResultSet rs = stat.getGeneratedKeys();
if (rs.next()) {
    int newId = rs.getInt(1);
}
```

### 多结果集

```java
boolean hasResult = stat.execute(sql);
while (true) {
    if (hasResult) {
        ResultSet rs = stat.getResultSet();
        // 处理结果集...
    } else {
        int count = stat.getUpdateCount();
        if (count == -1) break; // 没有更多结果
    }
    hasResult = stat.getMoreResults();
}
```

---

## 5.8 可滚动和可更新的结果集

### 可滚动结果集

```java
Statement stat = conn.createStatement(
    ResultSet.TYPE_SCROLL_INSENSITIVE, // 可滚动
    ResultSet.CONCUR_READ_ONLY);
ResultSet rs = stat.executeQuery(sql);
rs.last();           // 移动到最后一行
rs.absolute(3);      // 移动到第3行
rs.relative(-1);     // 向前移动1行
rs.previous();       // 前一行
```

### 可更新结果集

```java
Statement stat = conn.createStatement(
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_UPDATABLE);      // 可更新
ResultSet rs = stat.executeQuery("SELECT * FROM books");
rs.next();
rs.updateDouble("price", 29.99);
rs.updateRow();   // 提交更改

// 插入新行
rs.moveToInsertRow();
rs.updateString("title", "New Book");
rs.insertRow();
rs.moveToCurrentRow();
```

---

## 5.9 行集（RowSet）

- `RowSet` 是 `ResultSet` 的扩展，支持**离线操作**
- 常用实现：`CachedRowSet`（断开连接后仍可操作数据）

```java
RowSetFactory factory = RowSetProvider.newFactory();
CachedRowSet crs = factory.createCachedRowSet();
crs.setUrl("jdbc:mysql://localhost:3306/mydb");
crs.setUsername("user");
crs.setPassword("password");
crs.setCommand("SELECT * FROM books WHERE price < ?");
crs.setDouble(1, 50.0);
crs.execute(); // 获取数据后自动断开连接
// 修改后同步回数据库
crs.acceptChanges(conn);
```

- 支持**分页**：`crs.setPageSize(20)`

---

## 5.10 元数据（Metadata）

### DatabaseMetaData

```java
DatabaseMetaData meta = conn.getMetaData();
// 获取所有表
ResultSet tables = meta.getTables(null, null, null, 
    new String[]{"TABLE"});
// 获取数据库产品名和版本
String product = meta.getDatabaseProductName();
```

### ResultSetMetaData

```java
ResultSetMetaData rsMeta = rs.getMetaData();
int columnCount = rsMeta.getColumnCount();
for (int i = 1; i <= columnCount; i++) {
    String name = rsMeta.getColumnLabel(i);
    String type = rsMeta.getColumnTypeName(i);
}
```

---

## 5.11 事务（Transaction）

### 基本用法

```java
conn.setAutoCommit(false); // 关闭自动提交
try {
    // 执行多条 SQL ...
    stat.executeUpdate(sql1);
    stat.executeUpdate(sql2);
    conn.commit();         // 全部成功则提交
} catch (SQLException e) {
    conn.rollback();       // 任一失败则回滚
}
```

### 保存点（Savepoint）

```java
Savepoint sp = conn.setSavepoint();
// ... 执行操作 ...
conn.rollback(sp);        // 回滚到保存点
conn.releaseSavepoint(sp);
```

### 批量更新

```java
stat.addBatch(sql1);
stat.addBatch(sql2);
stat.addBatch(sql3);
int[] counts = stat.executeBatch(); // 批量执行
```

---

## 5.12 连接池（Connection Pool）

- 频繁创建和关闭数据库连接开销很大，连接池可以**复用连接**
- 常用连接池实现：**HikariCP**、Druid、C3P0
- 使用 `DataSource` 接口获取连接：

```java
// HikariCP 示例
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
config.setUsername("user");
config.setPassword("password");
config.setMaximumPoolSize(10);

DataSource ds = new HikariDataSource(config);
try (Connection conn = ds.getConnection()) {
    // 使用连接...
} // 连接自动归还到池中，而非真正关闭
```

<aside>
💡

在生产环境中应**始终使用连接池**，而不是直接通过 `DriverManager` 获取连接。

</aside>

---

## 本章速查表

| **操作** | **关键类 / 方法** | **注意事项** |
| --- | --- | --- |
| 获取连接 | `DriverManager.getConnection()` | 生产环境用连接池 |
| 执行查询 | `PreparedStatement`  • `executeQuery()` | 禁止拼接 SQL |
| 执行更新 | `executeUpdate()` / `executeBatch()` | 批量操作用 batch |
| 处理结果 | `ResultSet.getXxx()` | `wasNull()` 判断空值 |
| 事务管理 | `setAutoCommit(false)`  • `commit()` / `rollback()` | 异常时务必回滚 |
| 元数据 | `DatabaseMetaData` / `ResultSetMetaData` | 可用于动态 SQL 工具 |
| 连接池 | `DataSource`（HikariCP 等） | 生产环境必须使用 |