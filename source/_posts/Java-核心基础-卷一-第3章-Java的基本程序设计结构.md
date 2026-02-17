---
title: 第3章 Java的基本程序设计结构
date: 2026-02-17 03:20:27
categories:
- [编程技术, Java]
---
## 3.1 一个简单的 Java 程序

```java
public class FirstSample {
    public static void main(String[] args) {
        System.out.println("We will not use 'Hello, World!'");
    }
}
```

- Java 区分**大小写**
- 关键字 `public` 称为**访问修饰符**，控制其他代码对这段代码的访问级别
- `class` 关键字表明 Java 程序中的全部内容都包含在**类**中，类名必须与文件名一致
- `main` 方法必须声明为 `public static void`，是程序的入口点
- Java 中每条语句必须以**分号 `;`** 结束
- 使用**花括号 `{}`** 划分程序的各个部分（块）

## 3.2 注释

Java 支持三种注释方式：

```java
// 单行注释：从 // 到行尾

/*
 * 多行注释：
 * 可跨越多行
 */

/**
 * 文档注释：
 * 用于自动生成 API 文档（javadoc）
 * @param args 命令行参数
 */
```

<aside>
⚠️

注释不能嵌套，`/* */` 注释内部不能再包含 `/* */`。

</aside>

## 3.3 数据类型

Java 是一种**强类型**语言，每个变量都必须声明类型。共有 **8 种基本数据类型**：

### 整型

| **类型** | **存储空间** | **取值范围** |
| --- | --- | --- |
| `byte` | 1 字节 | -128 ~ 127 |
| `short` | 2 字节 | -32 768 ~ 32 767 |
| `int` | 4 字节 | 约 ±21 亿 |
| `long` | 8 字节 | 约 ±9.2 × 10¹⁸ |
- `long` 类型字面量需加后缀 `L`，如 `4000000000L`
- 十六进制前缀 `0x`，八进制前缀 `0`，二进制前缀 `0b`
- 可用下划线分隔数字增强可读性：`1_000_000`

### 浮点型

| **类型** | **存储空间** | **精度** |
| --- | --- | --- |
| `float` | 4 字节 | 约 6~7 位有效数字，需加后缀 `F` |
| `double` | 8 字节 | 约 15 位有效数字（默认类型） |
- 浮点数存在**舍入误差**，不能直接用 `==` 比较
- 特殊值：`Double.POSITIVE_INFINITY`、`Double.NEGATIVE_INFINITY`、`Double.NaN`

### char 类型

- 占 **2 字节**，采用 **UTF-16** 编码
- 字面量用**单引号**括起，如 `'A'`、`'\n'`
- 转义序列：`\n` 换行、`\t` 制表符、`\\` 反斜杠、`\'` 单引号、`\"` 双引号

### boolean 类型

- 只有 `true` 和 `false` 两个值
- **不能**与整数相互转换（与 C/C++ 不同）

## 3.4 变量与常量

### 变量声明与初始化

```java
int vacationDays;           // 声明
vacationDays = 12;          // 赋值
double salary = 65000.0;    // 声明并初始化
var greeting = "Hello";     // Java 10+：局部变量类型推断
```

- 变量名以**字母、下划线 `_` 或 `$`** 开头，区分大小写
- 从 Java 10 起，可以用 `var` 关键字让编译器自动推断局部变量类型

### 常量

```java
final double CM_PER_INCH = 2.54;  // 用 final 声明常量，值不可修改
```

- 常量名习惯使用**全大写**，单词间以下划线分隔
- **类常量**使用 `static final` 声明，可在同一个类的多个方法中使用

### 枚举类型

```java
enum Size { SMALL, MEDIUM, LARGE, EXTRA_LARGE }
Size s = Size.MEDIUM;
```

## 3.5 运算符

### 算术运算符

- `+` `-` `*` `/` `%`（取余）
- 整数除法 `/` 会**截断小数部分**
- 整数被 `0` 除会产生异常，浮点数被 `0` 除得到无穷大或 NaN

### 数学函数与常量

```java
Math.sqrt(x);      // 平方根
Math.pow(x, a);    // x 的 a 次幂
Math.PI;            // π
Math.E;             // e
Math.random();      // [0, 1) 随机数
```

### 类型转换

```
byte → short → int → long → float → double
              char ↗
```

- **自动转换**（隐式）：小范围 → 大范围
- **强制转换**（显式）：可能丢失精度

```java
double x = 9.997;
int nx = (int) x;          // nx = 9，截断小数
int nx2 = (int) Math.round(x);  // nx2 = 10，四舍五入
```

### 赋值与自增/自减

```java
x += 4;    // 等同于 x = x + 4
n++;       // 后缀自增：先使用后加 1
++n;       // 前缀自增：先加 1 后使用
```

### 关系与逻辑运算符

- 关系：`==` `!=` `<` `>` `<=` `>=`
- 逻辑：`&&`（短路与）、`||`（短路或）、`!`（非）
- 三元运算符：`condition ? expr1 : expr2`

### 位运算符

- `&`（与）、`|`（或）、`^`（异或）、`~`（取反）
- `<<`（左移）、`>>`（算术右移）、`>>>`（逻辑右移，高位补 0）

## 3.6 字符串

### String 基础

- Java 字符串是 **Unicode 字符序列**，是**不可变**的（immutable）
- 用双引号括起：`"Hello"`

```java
String greeting = "Hello";
String s = greeting.substring(0, 3);  // "Hel"
```

### 常用操作

| **方法** | **说明** |
| --- | --- |
| `length()` | 返回字符串长度 |
| `charAt(i)` | 返回位置 i 的字符 |
| `substring(a, b)` | 提取子串 [a, b) |
| `equals(str)` | 比较内容是否相等 |
| `equalsIgnoreCase(str)` | 忽略大小写比较 |
| `startsWith()` / `endsWith()` | 前缀 / 后缀判断 |
| `indexOf(str)` | 查找子串位置 |
| `trim()` / `strip()` | 去除首尾空白 |
| `toUpperCase()` / `toLowerCase()` | 大小写转换 |
| `replace(old, new)` | 替换字符或子串 |

### 拼接

```java
String a = "Hello";
String b = "World";
String c = a + " " + b;              // "Hello World"
String d = String.join(" / ", "S", "M", "L");  // "S / M / L"
```

### 字符串比较

<aside>
🚨

**千万不要用 `==` 比较字符串内容！** `==` 比较的是引用（内存地址），必须使用 `equals()` 方法。

```java
"Hello".equals(greeting)  // ✅ 正确
greeting == "Hello"       // ❌ 不可靠
```

</aside>

### StringBuilder

- 当需要**频繁拼接**字符串时，使用 `StringBuilder` 避免产生大量中间对象

```java
StringBuilder builder = new StringBuilder();
builder.append("Hello");
builder.append(" World");
String result = builder.toString();  // "Hello World"
```

### 文本块（Java 13+）

```java
String html = """
        <html>
            <body>
                <p>Hello</p>
            </body>
        </html>
        """;
```

## 3.7 输入与输出

### 读取输入（Scanner）

```java
import java.util.Scanner;

Scanner in = new Scanner(System.in);
String name = in.nextLine();    // 读取一行
int age = in.nextInt();         // 读取整数
double salary = in.nextDouble(); // 读取浮点数
```

### 格式化输出

```java
System.out.printf("Hello, %s. Next year, you'll be %d.\n", name, age + 1);
```

常用格式转换符：

- `%d` 整数、`%f` 浮点数、`%s` 字符串、`%n` 换行
- `%8.2f` 宽度 8、小数 2 位

### 文件输入与输出

```java
// 读文件
Scanner in = new Scanner(Path.of("data.txt"), StandardCharsets.UTF_8);

// 写文件
PrintWriter out = new PrintWriter("output.txt", StandardCharsets.UTF_8);
out.println("Hello, File!");
out.close();
```

## 3.8 控制流程

### 条件语句

```java
if (condition) {
    // ...
} else if (condition2) {
    // ...
} else {
    // ...
}
```

### switch 语句

```java
// 传统写法
switch (choice) {
    case 1:
        // ...
        break;
    case 2:
        // ...
        break;
    default:
        // ...
}

// Java 14+ 新写法（switch 表达式）
String label = switch (choice) {
    case 1 -> "One";
    case 2 -> "Two";
    default -> "Other";
};
```

### 循环

```java
// while 循环
while (condition) {
    // ...
}

// do-while 循环（至少执行一次）
do {
    // ...
} while (condition);

// for 循环
for (int i = 0; i < 10; i++) {
    // ...
}

// for-each 循环（遍历数组或集合）
for (int element : array) {
    System.out.println(element);
}
```

### 中断控制流程

- `break`：跳出当前循环
- `continue`：跳过本次迭代，进入下一次
- **带标签的 break**：可跳出多层嵌套循环

```java
outer:
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if (condition) break outer;  // 跳出外层循环
    }
}
```

## 3.9 大数

- 当基本整数和浮点数**精度不够**时，使用 `java.math` 包中的：
    - `BigInteger`：任意精度的**整数**运算
    - `BigDecimal`：任意精度的**浮点数**运算

```java
BigInteger a = BigInteger.valueOf(100);
BigInteger b = a.multiply(BigInteger.valueOf(200));  // 不能用 * 运算符
BigDecimal c = new BigDecimal("0.1");                // 用字符串构造避免精度问题
```

## 3.10 数组

### 声明与初始化

```java
int[] a = new int[100];                 // 声明长度为 100 的数组，默认值 0
int[] primes = { 2, 3, 5, 7, 11 };     // 直接初始化
String[] names = new String[10];        // 默认值 null
```

- 数组长度通过 `a.length` 获取
- **数组一旦创建，大小不可改变**

### 数组拷贝

```java
int[] copied = Arrays.copyOf(original, original.length);
// 第二个参数可以大于原长度（多余位置补默认值），常用于扩展数组
```

### 数组排序

```java
Arrays.sort(a);  // 使用优化的快速排序算法
```

### 多维数组

```java
double[][] balances = new double[10][6];  // 10 行 6 列

// 初始化
int[][] magicSquare = {
    { 16, 3, 2, 13 },
    {  5, 10, 11, 8 },
    {  9,  6, 7, 12 },
    {  4, 15, 14, 1 }
};
```

### 不规则数组

- Java 的多维数组实际上是**数组的数组**，每一行可以有不同的长度

```java
int[][] odds = new int[10][];
for (int i = 0; i < 10; i++) {
    odds[i] = new int[i + 1];  // 每行长度不同
}
```

<aside>
✅

**本章小结：**

- Java 的 **8 种基本数据类型**：`byte`、`short`、`int`、`long`、`float`、`double`、`char`、`boolean`
- 字符串是**不可变对象**，用 `equals()` 比较内容，用 `StringBuilder` 高效拼接
- 控制流程包括 `if`、`switch`、`while`、`do-while`、`for`、`for-each`
- `BigInteger` 和 `BigDecimal` 用于高精度计算
- 数组大小固定，多维数组本质是数组的数组，支持不规则结构
</aside>