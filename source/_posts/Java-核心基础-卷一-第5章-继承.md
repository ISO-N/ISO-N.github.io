---
title: 第5章 继承
date: 2026-02-17 03:20:28
categories:
- [编程技术, Java]
---
## 5.1 类、超类和子类

### 定义子类

- 使用 `extends` 关键字表示继承关系
- 子类**自动继承**超类的字段和方法（`private` 成员除外，虽然继承但无法直接访问）

```java
public class Manager extends Employee {
    private double bonus;

    public void setBonus(double bonus) {
        this.bonus = bonus;
    }
}
```

### 覆盖方法（Override）

- 子类可以**覆盖**超类中的方法，提供新的实现
- 使用 `super` 关键字调用超类的方法

```java
@Override
public double getSalary() {
    double baseSalary = super.getSalary();  // 调用超类方法
    return baseSalary + bonus;
}
```

<aside>
⚠️

`super` 不是对象引用，不能赋值给变量，它只是一个指示编译器调用超类方法的特殊关键字。

</aside>

### 子类构造器

- 子类构造器中使用 `super(...)` 调用超类构造器，且**必须是第一条语句**
- 如果没有显式调用，编译器会自动调用超类的**无参构造器**

```java
public Manager(String name, double salary, int year, int month, int day) {
    super(name, salary, year, month, day);  // 调用超类构造器
    bonus = 0;
}
```

### 继承层次

- 由一个公共超类派生出的所有类的集合称为**继承层次**（inheritance hierarchy）
- 从某个类到其祖先的路径称为**继承链**（inheritance chain）
- Java **不支持多继承**，每个类只能有一个直接超类

### 多态与动态绑定

- **多态**（polymorphism）：一个对象变量可以引用多种实际类型
- **动态绑定**（dynamic binding）：运行时自动选择适当的方法

```java
Employee e;
e = new Employee("Carl", 75000, 1987, 12, 15);   // ✅
e = new Manager("Boss", 80000, 1987, 12, 15);     // ✅ 超类变量引用子类对象
```

<aside>
💡

**方法调用的解析过程：**

1. 编译器查看对象的**声明类型**和方法名，列出所有候选方法
2. 编译器根据参数类型进行**重载解析**，选出匹配的方法签名
3. 如果方法是 `private`、`static`、`final` 或构造器，执行**静态绑定**
4. 否则采用**动态绑定**，在运行时根据对象的实际类型查找方法
</aside>

### 阻止继承：final 类和方法

```java
// final 类：不能被继承
public final class String { ... }

// final 方法：不能被子类覆盖
public final String getName() {
    return name;
}
```

- 将类或方法声明为 `final` 的主要原因是确保语义不会在子类中被改变
- `final` 方法可以让编译器进行**内联优化**

### 强制类型转换

```java
Manager boss = (Manager) staff[0];  // 将 Employee 转为 Manager
```

- 只能在继承层次内进行转换
- 转换前应使用 `instanceof` 检查

```java
if (staff[1] instanceof Manager m) {
    // Java 16+：模式匹配，转换成功后直接绑定到变量 m
    m.setBonus(5000);
}
```

<aside>
🚨

如果转换失败（对象实际上不是目标类型），会抛出 `ClassCastException`。**始终使用 `instanceof` 在转换前进行检查。**

</aside>

### 受保护访问（protected）

| **修饰符** | **本类** | **同包** | **子类** | **其他** |
| --- | --- | --- | --- | --- |
| `private` | ✅ | ❌ | ❌ | ❌ |
| 默认（包访问） | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |
- `protected` 成员对**子类**和**同一个包**中的类可见
- 实际开发中应谨慎使用 `protected` 字段，优先使用 `protected` 方法

## 5.2 Object：所有类的超类

- Java 中每个类都直接或间接地继承自 `Object` 类
- 如果没有明确 `extends`，则默认继承 `Object`

### equals 方法

- `Object.equals()` 默认比较**引用**（是否为同一对象）
- 通常需要覆盖以比较对象的**内容**

```java
@Override
public boolean equals(Object otherObject) {
    if (this == otherObject) return true;          // 引用相同
    if (otherObject == null) return false;         // null 检查
    if (getClass() != otherObject.getClass()) return false;  // 类型检查
    Employee other = (Employee) otherObject;
    return name.equals(other.name)
        && salary == other.salary
        && hireDay.equals(other.hireDay);
}
```

<aside>
💡

**`equals` 方法的约定（等价关系）：**

- **自反性**：`x.equals(x)` 返回 `true`
- **对称性**：`x.equals(y)` 和 `y.equals(x)` 结果相同
- **传递性**：若 `x.equals(y)` 且 `y.equals(z)`，则 `x.equals(z)`
- **一致性**：多次调用结果不变
- 对任意非 null 的 `x`，`x.equals(null)` 返回 `false`
</aside>

### hashCode 方法

- 每个对象都有一个散列码（hash code）
- **如果覆盖了 `equals`，就必须覆盖 `hashCode`**
- 相等的对象必须有相同的散列码

```java
@Override
public int hashCode() {
    return Objects.hash(name, salary, hireDay);
}
```

### toString 方法

- 返回对象的字符串表示，建议每个类都覆盖
- 当对象与字符串通过 `+` 拼接时，会自动调用 `toString()`

```java
@Override
public String toString() {
    return getClass().getName()
        + "[name=" + name
        + ",salary=" + salary
        + ",hireDay=" + hireDay + "]";
}
```

## 5.3 泛型数组列表（ArrayList）

- 数组大小固定，`ArrayList` 可以**动态调整容量**

```java
ArrayList<Employee> staff = new ArrayList<>();
staff.add(new Employee("Harry", ...));  // 添加元素
staff.add(new Employee("Tony", ...));

Employee e = staff.get(0);     // 获取元素
staff.set(0, harry);           // 替换元素
staff.remove(0);               // 删除元素
int n = staff.size();          // 元素数量
```

| **方法** | **说明** |
| --- | --- |
| `add(E e)` | 在末尾添加元素 |
| `add(int index, E e)` | 在指定位置插入元素 |
| `get(int index)` | 获取指定位置的元素 |
| `set(int index, E e)` | 替换指定位置的元素 |
| `remove(int index)` | 删除指定位置的元素 |
| `size()` | 返回元素数量 |
| `ensureCapacity(int n)` | 预分配容量，减少扩容次数 |
| `trimToSize()` | 将容量缩减到当前大小 |

<aside>
⚠️

`ArrayList` 的类型参数不能是基本类型，必须使用**包装类**：`ArrayList<Integer>` 而非 `ArrayList<int>`。

</aside>

## 5.4 对象包装器与自动装箱

- 每种基本类型都有对应的**包装类**（wrapper class）

| **基本类型** | **包装类** |
| --- | --- |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `short` | `Short` |
| `byte` | `Byte` |
| `char` | `Character` |
| `boolean` | `Boolean` |

### 自动装箱与拆箱

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(3);           // 自动装箱：int → Integer
int n = list.get(0);   // 自动拆箱：Integer → int
```

<aside>
🚨

包装类对象用 `==` 比较可能不可靠！应使用 `equals()` 方法。自动装箱规范要求 `boolean`、`byte`、小于 128 的 `short` 和 `int` 会被缓存，但超出范围的值不保证。

</aside>

## 5.5 参数数量可变的方法

```java
public static double max(double... values) {
    double largest = Double.NEGATIVE_INFINITY;
    for (double v : values)
        if (v > largest) largest = v;
    return largest;
}

// 调用
double m = max(3.1, 40.4, -5);
```

- `double... values` 等同于 `double[]` 参数
- `printf` 就是一个典型的可变参数方法：`public PrintStream printf(String fmt, Object... args)`

## 5.6 抽象类

- 使用 `abstract` 关键字定义抽象类和抽象方法
- 抽象类**不能实例化**，但可以声明抽象类的变量来引用子类对象
- 抽象方法**没有方法体**，子类必须实现所有抽象方法（除非子类也是抽象类）

```java
public abstract class Person {
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    // 抽象方法：没有实现，子类必须覆盖
    public abstract String getDescription();
}

public class Student extends Person {
    private String major;

    public Student(String name, String major) {
        super(name);
        this.major = major;
    }

    @Override
    public String getDescription() {
        return "a student majoring in " + major;
    }
}
```

## 5.7 枚举类

```java
public enum Size {
    SMALL("S"), MEDIUM("M"), LARGE("L"), EXTRA_LARGE("XL");

    private String abbreviation;

    // 枚举构造器始终是 private
    Size(String abbreviation) {
        this.abbreviation = abbreviation;
    }

    public String getAbbreviation() {
        return abbreviation;
    }
}
```

- 所有枚举类型都是 `Enum` 类的子类
- 常用方法：

| **方法** | **说明** |
| --- | --- |
| `toString()` | 返回枚举常量名，如 `"SMALL"` |
| `valueOf("SMALL")` | 由字符串返回对应的枚举常量 |
| `values()` | 返回所有枚举常量的数组 |
| `ordinal()` | 返回枚举常量的声明位置（从 0 开始） |

## 5.8 密封类（Java 17+）

- **密封类**（sealed class）限制哪些类可以继承它
- 使用 `sealed` 和 `permits` 关键字

```java
public sealed class Shape
    permits Circle, Rectangle, Triangle {
    // ...
}

public final class Circle extends Shape { ... }        // final：不可再继承
public sealed class Rectangle extends Shape            // sealed：继续限制
    permits FilledRectangle { ... }
public non-sealed class Triangle extends Shape { ... } // non-sealed：开放继承
```

- 子类必须是以下三种之一：
    - `final`：不允许进一步继承
    - `sealed`：继续限制继承
    - `non-sealed`：开放继承

## 5.9 反射

### Class 类

- 每个类都有一个对应的 `Class` 对象，包含类的**元数据**信息

```java
// 获取 Class 对象的三种方式
Class<?> cl1 = e.getClass();                    // 通过对象
Class<?> cl2 = Class.forName("java.util.Random"); // 通过完整类名
Class<?> cl3 = Random.class;                     // 通过 .class 字面量
```

### 利用反射分析类

```java
Class<?> cl = Class.forName("java.lang.Double");

// 获取字段
Field[] fields = cl.getDeclaredFields();

// 获取方法
Method[] methods = cl.getDeclaredMethods();

// 获取构造器
Constructor<?>[] constructors = cl.getDeclaredConstructors();
```

- `getFields()` / `getMethods()`：获取**公有**成员（含继承的）
- `getDeclaredFields()` / `getDeclaredMethods()`：获取**本类声明的所有**成员（含私有）

### 利用反射在运行时访问对象

```java
Employee harry = new Employee("Harry Hacker", 35000, 10, 1, 1989);
Class<?> cl = harry.getClass();
Field f = cl.getDeclaredField("name");
f.setAccessible(true);             // 允许访问私有字段
Object v = f.get(harry);          // 获取值："Harry Hacker"
f.set(harry, "New Name");         // 设置值
```

<aside>
⚠️

`setAccessible(true)` 可以突破访问控制，但应**谨慎使用**。Java 9+ 的模块系统可能会阻止对某些包的反射访问。

</aside>

### 调用任意方法

```java
Method m = Employee.class.getMethod("getSalary");
double salary = (Double) m.invoke(harry);  // 调用 harry.getSalary()
```

## 5.10 继承的设计技巧

<aside>
✅

**本章小结 — 继承设计技巧：**

- 将**公共操作和字段**放在超类中
- 不要使用 `protected` 字段，优先使用 `protected` 方法
- 使用继承实现 **"is-a"** 关系，而非随意扩展
- 除非所有继承的方法都有意义，否则**不要使用继承**
- 覆盖方法时，不要偏离最初的设计意图
- 使用**多态**而非类型信息（避免大量 `instanceof` 判断）
- 合理使用 `abstract`、`final`、`sealed` 控制继承层次
- 覆盖 `equals` 时必须同时覆盖 `hashCode`
- 善用 `ArrayList` 代替固定大小的数组
- 反射功能强大但应仅在必要时使用（如框架开发）
</aside>