---
title: 第9章 Java平台模块系统
date: 2026-02-17 15:12:06
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 9.1 模块的概念

- Java 9 引入了 **Java 平台模块系统（JPMS）**，也称为 **Project Jigsaw**
- 模块是一组**包（package）**的集合，附带一个 **模块描述符（[module-info.java](http://module-info.java)）**，明确声明：
    - 该模块**导出（exports）**哪些包
    - 该模块**需要（requires）**哪些其他模块
- 模块化的核心目标：
    - **强封装**：隐藏内部实现细节，只暴露公开 API
    - **可靠的依赖管理**：在编译和启动时就能发现缺失的依赖
    - **精简运行时**：通过 `jlink` 创建仅包含所需模块的自定义 JRE

---

## 9.2 对模块命名

- 模块名推荐使用**反向域名**风格，如 `com.mycompany.myapp`
- 模块名与包名类似，但它们是**独立的命名空间**
- 一个模块可以包含多个包，但**一个包只能属于一个模块**

---

## 9.3 模块化的 "Hello World"

一个最简单的模块结构：

```
mymodule/
├── module-info.java
└── com/
    └── example/
        └── Main.java
```

[`module-info.java`](http://module-info.java) 示例：

```java
module com.example.mymodule {
    // 暂无依赖和导出
}
```

编译与运行：

```bash
# 编译
javac -d out --module-source-path src -m com.example.mymodule

# 运行
java --module-path out -m com.example.mymodule/com.example.Main
```

---

## 9.4 对模块的需求（requires）

- 使用 `requires` 声明当前模块依赖的其他模块

```java
module com.example.app {
    requires java.sql;        // 需要 java.sql 模块
    requires java.logging;    // 需要 java.logging 模块
}
```

- `requires transitive`：**传递依赖**，任何依赖本模块的模块也会自动获得该依赖

```java
module com.example.app {
    requires transitive java.sql;
}
```

- `requires static`：**编译时需要，运行时可选**，用于可选依赖

```java
module com.example.app {
    requires static java.compiler;
}
```

---

## 9.5 导出包（exports）

- 使用 `exports` 使包中的 `public` 类型对其他模块可见
- **未导出的包**中的 public 类对外部模块不可访问（强封装）

```java
module com.example.lib {
    exports com.example.lib.api;           // 导出给所有模块
    exports com.example.lib.spi to         // 限定导出，只对指定模块可见
        com.example.plugin;
}
```

---

## 9.6 模块化的 JAR

- 模块化 JAR 就是在根目录下包含 `module-info.class` 的普通 JAR
- 放在**模块路径（module path）**上时作为**命名模块**
- 放在**类路径（classpath）**上时 `module-info.class` 会被忽略，行为与普通 JAR 相同

---

## 9.7 模块与反射访问

- 模块系统默认**阻止**对未导出包的反射访问
- 使用 `opens` 关键字允许运行时反射访问（如框架需要）：

```java
module com.example.app {
    opens com.example.app.model;             // 对所有模块开放反射
    opens com.example.app.internal to        // 仅对指定模块开放反射
        com.google.gson;
}
```

- 也可以声明整个模块为 **开放模块（open module）**：

```java
open module com.example.app {
    // 所有包都允许反射访问
}
```

---

## 9.8 自动模块

- 将**非模块化的 JAR** 放到模块路径上时，它会变成**自动模块**
- 自动模块的特点：
    - 模块名从 JAR 文件名推导（去掉版本号，`-` 替换为 `.`），或由 `Automatic-Module-Name` manifest 属性指定
    - **导出所有包**
    - **可以读取所有其他模块**（包括未命名模块）

---

## 9.9 未命名模块

- 类路径上的所有类属于**未命名模块（unnamed module）**
- 未命名模块的特点：
    - 可以读取**所有其他模块**
    - **导出所有包**（但命名模块不能 `requires` 未命名模块）
- 这是 Java 9+ 的**向后兼容机制**，保证旧代码依然能运行

---

## 9.10 用于迁移的命令行标志

当迁移到模块系统遇到问题时，可使用以下 JVM 参数：

| **参数** | **作用** |
| --- | --- |
| `--add-exports module/package=target` | 将 module 的 package 导出给 target 模块 |
| `--add-opens module/package=target` | 对 target 开放 package 的反射访问 |
| `--add-reads module=target` | 让 module 可以读取 target 模块 |
| `--add-modules modules` | 将额外模块加入模块图 |
| `--patch-module module=path` | 将路径中的类注入到指定模块中 |

---

## 9.11 服务加载（ServiceLoader）

- 模块系统与 `ServiceLoader` 深度集成
- **服务提供者**在 [`module-info.java`](http://module-info.java) 中声明：

```java
module com.example.provider {
    provides com.example.api.MyService
        with com.example.provider.MyServiceImpl;
}
```

- **服务消费者**声明使用该服务：

```java
module com.example.consumer {
    uses com.example.api.MyService;
}
```

- 加载服务：

```java
ServiceLoader<MyService> loader = ServiceLoader.load(MyService.class);
for (MyService service : loader) {
    service.doSomething();
}
```

---

## 9.12 操作模块的工具

| **工具** | **说明** |
| --- | --- |
| `jdeps` | 分析类/JAR 的依赖关系，帮助生成 module-info |
| `jlink` | 创建自定义精简 JRE（仅含所需模块） |
| `jmod` | 创建和管理 JMOD 文件（类似 JAR 但支持本地代码） |
| `jar --describe-module` | 查看 JAR 的模块描述信息 |

`jdeps` 示例：

```bash
# 分析 JAR 依赖
jdeps --module-path libs -s myapp.jar

# 自动生成 module-info.java
jdeps --generate-module-info out myapp.jar
```

`jlink` 示例：

```bash
# 创建自定义运行时镜像
jlink --module-path $JAVA_HOME/jmods:out \
      --add-modules com.example.app \
      --output myruntime
```

---

## 小结

<aside>
📌

- 模块系统通过 [`module-info.java`](http://module-info.java) 实现**强封装**和**显式依赖**
- 核心关键字：`module`、`requires`、`exports`、`opens`、`uses`、`provides...with`
- **自动模块**和**未命名模块**提供了向后兼容的迁移路径
- `jdeps` 帮助分析依赖，`jlink` 创建精简运行时
- 实际开发中，大多数应用仍运行在类路径上，模块系统主要用于**库开发**和 **JDK 自身的模块化**
</aside>