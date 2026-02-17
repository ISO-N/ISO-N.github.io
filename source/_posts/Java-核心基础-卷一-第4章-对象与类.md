---
title: 第4章 对象与类
date: 2026-02-17 03:20:28
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷一
---
## 4.1 面向对象程序设计概述

- **面向对象程序设计（OOP）** 是当前主流的程序设计范式，Java 是完全面向对象的语言
- OOP 的核心概念：
    - **封装（Encapsulation）**：将数据和操作数据的方法绑定在一起，对外隐藏实现细节
    - **继承（Inheritance）**：在已有类的基础上构建新类，实现代码复用
    - **多态（Polymorphism）**：同一操作作用于不同对象，可以有不同的行为

### 类与对象的关系

- **类（Class）** 是构造对象的模板或蓝图，定义了对象的属性和行为
- **对象（Object）** 是类的实例，具有具体的状态和行为
- 类比：类 = 饼干模具，对象 = 用模具做出来的饼干

### 对象的三个特征

- **行为（Behavior）**：对象能做什么 → 由方法定义
- **状态（State）**：对象当前的信息 → 由实例字段存储
- **标识（Identity）**：如何区分相同状态的不同对象 → 每个对象有唯一标识

---

## 4.2 使用预定义类

### Date 与 LocalDate

- `java.time.LocalDate` 用于表示日期（推荐使用）
- `Date` 类表示时间点，已不推荐用于日期操作

```java
LocalDate today = LocalDate.now();
LocalDate birthday = LocalDate.of(1999, 12, 31);

int year = today.getYear();       // 获取年
int month = today.getMonthValue(); // 获取月
int day = today.getDayOfMonth();   // 获取日
```

### 访问器方法与更改器方法

- **访问器方法（Accessor）**：只读取对象状态，不修改 → 如 `getYear()`
- **更改器方法（Mutator）**：会修改对象状态 → 如 `GregorianCalendar.add()`
- `LocalDate` 的方法如 `plusDays()` 不修改原对象，而是返回新对象（不可变类）

---

## 4.3 用户自定义类

### 类的基本结构

```java
public class Employee {
    // 实例字段
    private String name;
    private double salary;
    private LocalDate hireDay;

    // 构造器
    public Employee(String n, double s, int year, int month, int day) {
        name = n;
        salary = s;
        hireDay = LocalDate.of(year, month, day);
    }

    // 方法
    public String getName() {
        return name;
    }

    public double getSalary() {
        return salary;
    }

    public void raiseSalary(double byPercent) {
        double raise = salary * byPercent / 100;
        salary += raise;
    }
}
```

### 关键要点

- 一个源文件中只能有**一个** `public` 类，文件名必须与该类同名
- 实例字段建议声明为 `private`，通过公有方法访问
- 构造器特点：
    - 与类同名
    - 没有返回值（连 `void` 都没有）
    - 使用 `new` 操作符调用
    - 可以有多个构造器（重载）

---

## 4.4 静态字段与静态方法

### 静态字段（static field）

- 属于类而非对象，所有实例共享同一个值
- 也叫**类变量**

```java
class Employee {
    private static int nextId = 1;
    private int id;
}
```

### 静态方法（static method）

- 不能访问实例字段，只能访问静态字段
- 通过 `类名.方法名()` 调用

```java
public static int getNextId() {
    return nextId;
}
// 调用：Employee.getNextId()
```

### 适合使用静态方法的场景

- 方法不需要访问对象状态（所有参数通过参数列表提供）
- 方法只需要访问类的静态字段
- 工厂方法（如 `LocalDate.of()`、`NumberFormat.getCurrencyInstance()`）

### `main` 方法

- `main` 方法是静态方法，程序启动时还没有任何对象
- 每个类都可以有自己的 `main` 方法，用于单元测试

---

## 4.5 方法参数

<aside>
⚠️

Java 对方法参数采用 **值传递（call by value）**，不是引用传递。

</aside>

### 两种参数类型的表现

- **基本类型参数**：方法接收的是值的副本，无法修改原始变量

```java
public static void tripleValue(double x) {
    x = 3 * x; // 只修改了副本，原始值不变
}
```

- **对象引用参数**：方法接收的是引用的副本，可以通过引用修改对象状态，但不能让引用指向新对象

```java
public static void raiseSalary(Employee e, double percent) {
    e.raiseSalary(percent); // ✅ 可以修改对象状态
}

public static void swap(Employee a, Employee b) {
    Employee temp = a;
    a = b;
    b = temp; // ❌ 只交换了副本，原始引用不变
}
```

### 总结

- 方法**不能**修改基本类型的参数
- 方法**可以**改变对象参数的状态
- 方法**不能**让对象参数引用一个新对象

---

## 4.6 对象构造

### 重载（Overloading）

- 多个方法有相同名字但参数列表不同
- 编译器通过参数类型匹配来选择具体调用哪个方法

### 默认字段初始化

- 数值类型 → `0`
- 布尔类型 → `false`
- 对象引用 → `null`

### 无参构造器

- 如果类没有编写任何构造器，系统会提供一个默认无参构造器
- 如果类已有构造器，则不会自动提供无参构造器

### 显式字段初始化

```java
class Employee {
    private String name = "";  // 直接初始化
    private static int nextId;

    // 初始化块
    {
        id = nextId;
        nextId++;
    }

    // 静态初始化块
    static {
        nextId = new Random().nextInt(10000);
    }
}
```

### `this` 关键字

- `this` 引用当前对象
- `this(...)` 在一个构造器中调用另一个构造器

```java
public Employee(double s) {
    this("Employee #" + nextId, s); // 调用另一个构造器
    nextId++;
}
```

### 构造对象的完整流程

1. 如果构造器第一行调用了 `this(...)`，先执行对应构造器
2. 所有数据字段初始化为默认值（0 / false / null）
3. 按照声明顺序执行字段初始化语句和初始化块
4. 执行构造器体中的语句

---

## 4.7 包（Package）

### 包的作用

- 组织类，避免命名冲突
- 包名通常使用因特网域名的逆序：`com.horstmann.corejava`

### import 语句

```java
import java.time.LocalDate;      // 导入单个类
import java.time.*;               // 导入整个包
import static java.lang.Math.*;   // 静态导入
```

- `java.lang` 包自动导入，无需手动 import
- 如果两个包中有同名类（如 [`java.util.Date`](http://java.util.Date) 和 [`java.sql.Date`](http://java.sql.Date)），需要使用完全限定名

### 将类放入包中

```java
package com.horstmann.corejava;

public class Employee {
    // ...
}
```

- 源文件必须放在与包名匹配的目录中：`com/horstmann/corejava/[Employee.java](http://Employee.java)`

---

## 4.8 JAR 文件

- **JAR（Java Archive）** 文件是将多个类文件和资源打包的压缩文件（ZIP 格式）
- 使用 `jar` 工具创建：

```bash
jar cvf MyArchive.jar com/horstmann/corejava/*.class
```

- 可通过**清单文件（MANIFEST.MF）** 指定主类：

```
Main-Class: com.horstmann.corejava.MyApp
```

---

## 4.9 文档注释（Javadoc）

### 常用标记

| 标记 | 用途 |
| --- | --- |
| `@param` | 方法参数描述 |
| `@return` | 返回值描述 |
| `@throws` | 异常描述 |
| `@author` | 作者 |
| `@version` | 版本 |
| `@see` | 交叉引用 |
| `@since` | 引入版本 |
| `@deprecated` | 已弃用说明 |

### 示例

```java
/**
 * 提高员工工资
 * @param byPercent 加薪百分比（如 10 表示 10%）
 * @return 加薪后的薪资
 */
public double raiseSalary(double byPercent) {
    // ...
}
```

- 使用 `javadoc -d docDir package` 生成文档

---

## 4.10 类设计技巧

<aside>
💡

**设计良好的类应遵循以下原则：**

1. **保证数据私有** — 绝对不要破坏封装性
2. **一定要对数据进行初始化** — 不要依赖默认值
3. **不要在类中使用过多的基本类型** — 用其他类替换多个相关的基本类型
4. **不是所有字段都需要 getter 和 setter** — 按需提供
5. **将职责过多的类进行分解** — 一个类应专注于一件事
6. **类名和方法名要能体现其职责** — 命名要有意义
7. **优先使用不可变的类** — 线程安全，易于推理
</aside>