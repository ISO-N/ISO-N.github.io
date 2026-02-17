---
title: 第8章 脚本、编译与注解处理
date: 2026-02-17 15:12:05
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 8.1 Java 平台的脚本机制

- Java SE 6 引入了 **脚本 API**（`javax.script`），允许在 Java 程序中调用脚本语言（如 JavaScript、Groovy、Ruby 等）
- 核心接口与类：
    - `ScriptEngineManager`：脚本引擎的管理器，用于发现和创建引擎
    - `ScriptEngine`：脚本引擎接口，用于执行脚本代码
    - `Bindings`：键值对映射，用于在 Java 和脚本之间传递变量

### 获取脚本引擎

```java
ScriptEngineManager manager = new ScriptEngineManager();
ScriptEngine engine = manager.getEngineByName("nashorn"); // Java 8~14 内置
```

- 可通过 `getEngineByName()`、`getEngineByExtension()`、`getEngineByMimeType()` 获取引擎
- `manager.getEngineFactories()` 可列出所有可用的脚本引擎

### 执行脚本与绑定变量

```java
// 执行脚本字符串
engine.eval("var x = 10; x + 20;");

// 绑定变量：Java → 脚本
engine.put("name", "Java");
engine.eval("print('Hello from ' + name)");

// 从脚本中获取变量值
Object result = engine.get("x");
```

### 重定向输入输出

```java
// 将脚本输出重定向到指定 Writer
StringWriter writer = new StringWriter();
engine.getContext().setWriter(writer);
engine.eval("print('redirected')");
```

### 调用脚本函数

```java
engine.eval("function greet(name) { return 'Hello, ' + name; }");
Invocable inv = (Invocable) engine;
Object result = inv.invokeFunction("greet", "World");
```

### 编译脚本（提升性能）

```java
if (engine instanceof Compilable) {
    Compilable comp = (Compilable) engine;
    CompiledScript script = comp.compile("x + y");
    // 可多次执行编译后的脚本
    script.eval();
}
```

---

## 8.2 编译器 API

- [`javax.tools`](http://javax.tools) 包提供了 **Java 编译器 API**，可以在程序运行时动态编译 Java 源代码

### 基本用法

```java
JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
int result = compiler.run(null, null, null, "MyClass.java");
// result == 0 表示编译成功
```

### 使用编译任务（CompilationTask）

```java
JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
StandardJavaFileManager fileManager = compiler.getStandardFileManager(null, null, null);

Iterable<? extends JavaFileObject> sources =
    fileManager.getJavaFileObjectsFromFiles(Arrays.asList(new File("MyClass.java")));

JavaCompiler.CompilationTask task = compiler.getTask(
    null,        // Writer（错误输出）
    fileManager, // 文件管理器
    null,        // DiagnosticListener
    null,        // 编译选项
    null,        // 注解处理的类名
    sources      // 编译单元
);

boolean success = task.call();
```

### 从内存编译（无需文件）

- 自定义 `JavaFileObject`，将源码字符串作为输入
- 自定义 `JavaFileManager`，将编译结果存放到内存中的字节数组
- 适用于动态生成代码并立即加载执行的场景

### 诊断信息收集

```java
DiagnosticCollector<JavaFileObject> diagnostics = new DiagnosticCollector<>();
JavaCompiler.CompilationTask task = compiler.getTask(
    null, fileManager, diagnostics, null, null, sources);

for (Diagnostic<? extends JavaFileObject> d : diagnostics.getDiagnostics()) {
    System.out.printf("行 %d: %s%n", d.getLineNumber(), d.getMessage(null));
}
```

---

## 8.3 注解（Annotation）

### 什么是注解

- 注解是一种 **元数据**，以 `@` 开头附加在代码元素上（类、方法、字段、参数等）
- 注解本身不改变程序逻辑，但可以被编译器或工具读取并处理

### 常见标准注解

| 注解 | 作用 |
| --- | --- |
| `@Override` | 标记方法覆盖了父类方法 |
| `@Deprecated` | 标记已过时的元素 |
| `@SuppressWarnings` | 抑制编译器警告 |
| `@FunctionalInterface` | 标记函数式接口 |
| `@SafeVarargs` | 抑制可变参数的堆污染警告 |

### 元注解（用于定义注解的注解）

| 元注解 | 说明 |
| --- | --- |
| `@Target` | 指定注解适用的位置（`TYPE`、`METHOD`、`FIELD` 等） |
| `@Retention` | 指定注解保留策略：`SOURCE`、`CLASS`、`RUNTIME` |
| `@Documented` | 注解会出现在 Javadoc 中 |
| `@Inherited` | 子类自动继承父类的该注解 |
| `@Repeatable` | 允许同一位置多次使用该注解 |

### 自定义注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    int timeout() default 0;   // 带默认值的元素
    String description() default "";
}
```

- 注解元素类型只能是：基本类型、`String`、`Class`、枚举、其他注解，及以上类型的数组
- `default` 可设置默认值；没有默认值的元素在使用时必须提供

---

## 8.4 运行时注解处理（反射）

- `Retention` 为 `RUNTIME` 的注解可以在运行时通过 **反射** 读取

```java
Method m = MyClass.class.getMethod("myMethod");
if (m.isAnnotationPresent(Test.class)) {
    Test t = m.getAnnotation(Test.class);
    System.out.println("timeout = " + t.timeout());
}
```

- `getAnnotations()` — 获取所有注解
- `getAnnotation(Class)` — 获取指定注解
- `isAnnotationPresent(Class)` — 判断是否存在某注解

---

## 8.5 源码级注解处理

### 注解处理器（Annotation Processor）

- 在 **编译期** 对源码中的注解进行扫描和处理
- 继承 `AbstractProcessor`，重写 `process()` 方法

```java
@SupportedAnnotationTypes("com.example.MyAnnotation")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class MyProcessor extends AbstractProcessor {
    @Override
    public boolean process(Set<? extends TypeElement> annotations,
                           RoundEnvironment roundEnv) {
        for (Element e : roundEnv.getElementsAnnotatedWith(MyAnnotation.class)) {
            // 处理逻辑：生成代码、校验规则等
        }
        return true;
    }
}
```

### 注册处理器

- 在 `META-INF/services/javax.annotation.processing.Processor` 文件中注册处理器全限定类名
- 或使用 Google 的 `@AutoService` 自动生成注册文件

### Language Model API

- `javax.lang.model` 包提供了对 Java 语言元素的建模：
    - `Element` — 程序元素（类、方法、字段等）
    - `TypeMirror` — 类型信息
    - `AnnotationMirror` — 注解信息

### 使用 Filer 生成源文件

```java
Filer filer = processingEnv.getFiler();
JavaFileObject file = filer.createSourceFile("com.example.Generated");
try (PrintWriter out = new PrintWriter(file.openWriter())) {
    out.println("package com.example;");
    out.println("public class Generated { }");
}
```

---

## 8.6 字节码工程

- 注解处理也可在 **字节码层面** 进行，例如使用 ASM、Byte Buddy 等库
- 典型应用：
    - **AOP（面向切面编程）**：在方法前后插入逻辑
    - **ORM 框架**：如 JPA 的实体增强
    - **序列化框架**：自动生成序列化/反序列化代码

---

## 本章要点总结

<aside>
📌

- **脚本 API**：通过 `ScriptEngine` 在 Java 中执行脚本语言，支持变量绑定、函数调用和脚本编译
- **编译器 API**：通过 `JavaCompiler` 在运行时编译 Java 源代码，支持内存编译和诊断收集
- **注解**：元数据机制，通过元注解控制作用范围和保留策略
- **运行时处理**：使用反射在 `RUNTIME` 读取注解信息
- **源码级处理**：`AbstractProcessor` 在编译期扫描注解，可自动生成代码
- **字节码工程**：在更底层操作 `.class` 文件，实现 AOP 等高级功能
</aside>