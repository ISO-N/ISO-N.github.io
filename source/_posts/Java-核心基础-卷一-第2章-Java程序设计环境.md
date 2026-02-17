---
title: 第2章 Java程序设计环境
date: 2026-02-17 03:20:27
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷一
---
## 2.1 安装 Java 开发工具包（JDK）

### JDK 与 JRE 的区别

| **名称** | **全称** | **说明** |
| --- | --- | --- |
| **JDK** | Java Development Kit | 编写 Java 程序所需的完整开发工具包，包含编译器、工具和 JRE |
| **JRE** | Java Runtime Environment | 运行 Java 程序所需的最小环境，包含 JVM 和核心类库 |
| **JVM** | Java Virtual Machine | 执行字节码的虚拟机，是 Java 跨平台的核心 |

<aside>
💡

**开发 Java 程序必须安装 JDK，仅运行 Java 程序只需 JRE。**

</aside>

### JDK 版本术语

- **Java SE**（Standard Edition）：标准版，用于桌面和服务器开发
- **Java EE**（Enterprise Edition）：企业版，用于大型分布式企业应用（现已更名为 Jakarta EE）
- **Java ME**（Micro Edition）：微型版，用于嵌入式和移动设备

### 安装步骤

1. 从 Oracle 官网或 OpenJDK 下载对应操作系统的 JDK 安装包
2. 运行安装程序，记住安装路径（如 Windows 下 `C:\Program Files\Java\jdk-xx`）
3. 安装完成后验证：终端输入 `java -version` 和 `javac -version`

## 2.2 设置 JDK

### 配置环境变量

在安装 JDK 后，需要正确设置环境变量以便在终端中使用 Java 工具：

- **JAVA_HOME**：指向 JDK 安装目录
- **PATH**：添加 `$JAVA_HOME/bin`（Linux/macOS）或 `%JAVA_HOME%\bin`（Windows）

### Windows 配置示例

```bash
# 设置 JAVA_HOME
set JAVA_HOME=C:\Program Files\Java\jdk-17

# 添加到 PATH
set PATH=%JAVA_HOME%\bin;%PATH%
```

永久设置：右键"此电脑" → 属性 → 高级系统设置 → 环境变量

### Linux / macOS 配置示例

```bash
# 在 ~/.bashrc 或 ~/.zshrc 中添加
export JAVA_HOME=/usr/lib/jvm/jdk-17
export PATH=$JAVA_HOME/bin:$PATH
```

执行 `source ~/.bashrc` 或 `source ~/.zshrc` 使配置生效

### 安装源文件和文档

- **源文件**：JDK 安装目录中的 `lib/[src.zip](http://src.zip)` 包含所有公共类库的源代码，建议解压以便查阅
- **API 文档**：可从 Oracle 官网下载或在线查阅，是开发时的重要参考

## 2.3 使用命令行工具

### 编写第一个 Java 程序

```java
public class Welcome {
    public static void main(String[] args) {
        String greeting = "Welcome to Core Java!";
        System.out.println(greeting);
    }
}
```

### 编译与运行流程

```
源代码（.java）──▶ 编译器（javac）──▶ 字节码（.class）──▶ JVM（java）──▶ 运行结果
```

具体命令：

```bash
# 1. 编译：将 .java 文件编译为 .class 字节码文件
javac Welcome.java

# 2. 运行：JVM 加载并执行字节码（注意不要加 .class 后缀）
java Welcome
```

<aside>
⚠️

**常见注意事项：**

- 文件名必须与 `public class` 名称**完全一致**（包括大小写），如 [`Welcome.java`](http://Welcome.java)
- `javac` 命令后跟**文件名**（带 `.java` 后缀）
- `java` 命令后跟**类名**（不带 `.class` 后缀）
- 如果出现 "找不到或无法加载主类" 错误，检查类路径（classpath）设置
</aside>

## 2.4 使用集成开发环境（IDE）

### 主流 Java IDE

| **IDE** | **特点** |
| --- | --- |
| **IntelliJ IDEA** | 功能强大，智能补全出色，社区版免费，业界最流行 |
| **Eclipse** | 开源免费，插件生态丰富，老牌 Java IDE |
| **VS Code** | 轻量级编辑器，通过 Java 扩展包支持 Java 开发 |

### IDE 的核心优势

- **自动编译**：保存即编译，无需手动执行 `javac`
- **智能提示**：代码补全、错误检测、快速修复
- **调试功能**：断点调试、变量监视、表达式求值
- **项目管理**：自动管理类路径、依赖和构建配置
- **重构工具**：重命名、提取方法、移动类等

### 在 IDE 中创建项目的基本步骤

1. 创建新项目，选择 JDK 版本
2. 创建包（package）和类（class）
3. 编写代码
4. 点击运行按钮或使用快捷键运行

## 2.5 JShell（交互式编程工具）

- **JShell** 是 Java 9 引入的交互式 REPL（Read-Eval-Print Loop）工具
- 可以直接输入 Java 表达式和语句并立即看到结果，无需编写完整的类和 `main` 方法
- 非常适合**快速测试代码片段**和**学习 Java 语法**

```bash
# 启动 JShell
jshell

# 在 JShell 中直接执行
jshell> "Hello".length()
$1 ==> 5

jshell> int x = 10;
x ==> 10

jshell> Math.sqrt(2)
$3 ==> 1.4142135623730951

# 退出
jshell> /exit
```

<aside>
✅

**本章小结：**

- 开发 Java 程序需要安装 **JDK** 并正确配置 **JAVA_HOME** 和 **PATH** 环境变量
- Java 程序的基本流程：**编写 → javac 编译 → java 运行**
- 推荐使用 **IDE**（如 IntelliJ IDEA）提升开发效率
- 使用 **JShell** 快速试验代码片段，加速学习
</aside>