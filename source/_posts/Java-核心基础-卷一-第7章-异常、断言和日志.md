---
title: 第7章 异常、断言和日志
date: 2026-02-17 03:20:30
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷一
---
## 7.1 处理错误

- Java 中的异常处理机制用于在程序运行期间捕获和处理错误
- 如果某个操作没有正常完成，程序可以转到**错误处理器**，而不是直接终止
- 常见的错误类型：
    - 用户输入错误（格式错误、非法值等）
    - 设备错误（硬件故障、网络中断等）
    - 物理限制（磁盘满、内存不足等）
    - 代码错误（数组越界、空指针等）

### 异常分类

- Java 中所有异常都派生自 `Throwable` 类
- 两大分支：

<aside>
📌

**Error** — 系统内部错误和资源耗尽（程序无法处理）

**Exception** — 程序本身可以捕获和处理的异常

</aside>

- `Exception` 又分为：
    - **RuntimeException**（运行时异常 / 非检查型异常）
        - `NullPointerException`
        - `ArrayIndexOutOfBoundsException`
        - `ClassCastException`
        - `ArithmeticException`
    - **其他异常**（检查型异常 / checked exception）
        - `IOException`
        - `SQLException`
        - `FileNotFoundException`

<aside>
⚡

**规则：** 编译器要求必须为所有 *检查型异常* 提供异常处理器，而 *非检查型异常*（RuntimeException 及其子类）和 Error 不强制要求。

</aside>

---

## 7.2 声明检查型异常

- 方法可以用 `throws` 关键字声明可能抛出的检查型异常

```java
public void readFile(String name) throws FileNotFoundException, IOException {
    // ...
}
```

- **何时需要声明？**
    1. 调用了一个声明了检查型异常的方法
    2. 用 `throw` 语句主动抛出检查型异常
    3. 程序出现错误（如数组越界 → 不需要声明，属于非检查型）
    4. Java 虚拟机或运行时库的内部错误
- 子类重写方法时，声明的检查型异常**不能比**父类方法声明的更宽泛（可以更窄或不抛出）

---

## 7.3 如何抛出异常

```java
// 抛出已有异常类
throw new FileNotFoundException("找不到文件: " + fileName);
```

```java
// 自定义异常类
public class MyException extends Exception {
    public MyException() {}
    public MyException(String message) {
        super(message);
    }
    public MyException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

---

## 7.4 捕获异常

### try-catch 基本语法

```java
try {
    // 可能抛出异常的代码
} catch (ExceptionType1 e) {
    // 处理异常1
} catch (ExceptionType2 e) {
    // 处理异常2
}
```

### 多异常合并捕获（Java 7+）

```java
catch (FileNotFoundException | UnknownHostException e) {
    // 统一处理
}
```

### 再次抛出与异常链

```java
catch (SQLException e) {
    throw new MyException("数据库错误", e); // 包装原始异常
}
```

- 使用 `getCause()` 获取原始异常

### finally 子句

```java
try {
    // 获取资源
} catch (Exception e) {
    // 处理异常
} finally {
    // 无论是否异常，都会执行（通常用于释放资源）
}
```

<aside>
⚠️

**注意：** `finally` 中的 `return` 会覆盖 `try` 中的 `return`，应避免在 `finally` 中使用 `return`。

</aside>

### try-with-resources（Java 7+）

```java
try (var in = new FileInputStream("data.txt");
     var out = new FileOutputStream("out.txt")) {
    // 读写操作
}
// 自动调用 close()，无需手动关闭
```

- 资源类必须实现 `AutoCloseable` 接口
- 如果 `try` 块和 `close()` 都抛异常，`close()` 的异常会被**抑制**（suppressed），可通过 `getSuppressed()` 获取

---

## 7.5 异常使用技巧

1. **异常不能代替简单的测试** — 捕获异常的时间远超简单的判断检查
2. **不要过分细化异常** — 不必把每条语句都放进单独的 try 块
3. **合理利用异常层次** — 选择最具体的异常类型
4. **不要压制异常** — 空的 catch 块是不良实践
5. **检测错误时，苛刻比放任好** — 早抛出，早发现
6. **不要羞于传递异常** — 让更高层来处理有时更好

---

## 7.6 断言

- 断言是一种在开发和测试阶段进行条件检查的机制

```java
assert 条件;
assert 条件 : 表达式; // 失败时将表达式的值传给 AssertionError
```

- 示例：

```java
assert x >= 0 : "x 不能为负数: " + x;
```

### 启用与禁用

- 默认情况下断言是**禁用**的
- 启用：`java -ea MyApp` 或 `java -enableassertions MyApp`
- 禁用特定类或包：`java -ea -da:MyClass MyApp`

<aside>
💡

**断言 vs 异常：**

- 断言用于**开发阶段**捕获程序逻辑错误（assertion failure = bug）
- 异常用于**运行时**处理可预见的错误情况
- 断言失败抛出 `AssertionError`，不应该被捕获
</aside>

---

## 7.7 日志

- Java 标准库自带日志系统 `java.util.logging`

### 基本日志

```java
Logger.getGlobal().info("打印一条信息日志");
Logger.getGlobal().setLevel(Level.OFF); // 关闭所有日志
```

### 自定义 Logger

```java
private static final Logger logger = Logger.getLogger("com.myapp.package");
```

- Logger 名称具有层次结构，子 Logger 继承父 Logger 的设置

### 7 个日志级别（从高到低）

| 级别 | 用途 |
| --- | --- |
| `SEVERE` | 严重错误 |
| `WARNING` | 潜在问题 |
| `INFO` | 一般信息 |
| `CONFIG` | 配置信息 |
| `FINE` | 调试信息 |
| `FINER` | 更详细的调试 |
| `FINEST` | 最详细的调试 |
- 默认只记录 `INFO` 及以上级别

### 常用日志方法

```java
logger.warning("这是一条警告");
logger.fine("调试信息");
logger.log(Level.FINE, "变量值: {0}", value);

// 记录异常
logger.log(Level.SEVERE, "出错了", exception);

// 进入/退出方法（用于追踪执行流）
logger.entering("MyClass", "myMethod", params);
logger.exiting("MyClass", "myMethod", result);
```

### 处理器（Handler）与格式化器（Formatter）

- **Handler** 决定日志输出到哪里
    - `ConsoleHandler` — 输出到控制台（默认）
    - `FileHandler` — 输出到文件
    - `SocketHandler` — 输出到网络
- **Formatter** 决定日志的输出格式
    - `SimpleFormatter` — 文本格式
    - `XMLFormatter` — XML 格式

```java
var handler = new FileHandler("app.log");
handler.setFormatter(new SimpleFormatter());
logger.addHandler(handler);
```

### 日志配置文件

- 默认配置：`jre/lib/[logging.properties](http://logging.properties)`
- 自定义配置：`java -Djava.util.logging.config.file=[mylog.properties](http://mylog.properties) MyApp`

---

## 7.8 调试技巧

1. 使用 `Logger.getGlobal().info()` 打印变量值
2. 每个类中放一个 `main` 方法用于单元测试
3. 使用 JUnit 进行系统化的单元测试
4. 利用日志代理（logging proxy）拦截方法调用
5. 利用 `Throwable().printStackTrace()` 获取调用栈
6. 使用 `Thread.setDefaultUncaughtExceptionHandler()` 捕获未处理的异常
7. 使用 `-verbose` 标志观察类加载过程
8. 使用 `jconsole` 监控 Java 程序运行状态
9. 使用 `jmap` 查看堆内存信息