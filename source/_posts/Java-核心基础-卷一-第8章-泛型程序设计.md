---
title: 第8章 泛型程序设计
date: 2026-02-17 03:20:30
categories:
- [编程技术, Java]
---
## 8.1 为什么要使用泛型程序设计

- **泛型程序设计（Generic Programming）** 意味着编写的代码可以对多种不同类型的对象重用
- 在泛型出现之前，通用程序设计使用 `Object` 类型实现，存在两个问题：
    - 获取值时必须进行**强制类型转换**
    - 没有类型检查，运行时可能抛出 `ClassCastException`
- 泛型提供了**类型参数（type parameter）**，让编译器在编译期进行类型检查，提升代码安全性和可读性

```java
// 没有泛型
ArrayList list = new ArrayList();
list.add("hello");
String s = (String) list.get(0); // 需要强制转换

// 使用泛型
ArrayList<String> list = new ArrayList<>();
list.add("hello");
String s = list.get(0); // 无需强制转换，编译器自动检查类型
```

---

## 8.2 定义简单泛型类

- 泛型类就是有一个或多个**类型变量**的类
- 类型变量用尖括号 `<>` 括起来，放在类名后面
- 常见类型变量名：`E`（元素）、`K`（键）、`V`（值）、`T`（任意类型）、`U`/`S`（第二、三个类型）

```java
public class Pair<T> {
    private T first;
    private T second;

    public Pair() { first = null; second = null; }
    public Pair(T first, T second) {
        this.first = first;
        this.second = second;
    }

    public T getFirst() { return first; }
    public T getSecond() { return second; }
    public void setFirst(T newValue) { first = newValue; }
    public void setSecond(T newValue) { second = newValue; }
}
```

- 使用时用具体类型替换类型变量：`Pair<String>`

---

## 8.3 泛型方法

- 泛型方法可以定义在普通类中，也可以定义在泛型类中
- 类型变量放在修饰符之后、返回类型之前

```java
class ArrayAlg {
    public static <T> T getMiddle(T... a) {
        return a[a.length / 2];
    }
}

// 调用方式
String mid = ArrayAlg.<String>getMiddle("John", "Q", "Public");
// 大多数情况编译器可以推断类型，可省略 <String>
String mid = ArrayAlg.getMiddle("John", "Q", "Public");
```

---

## 8.4 类型变量的限定

- 使用 `extends` 关键字对类型变量进行限定
- 格式：`<T extends BoundingType>`
- T 必须是限定类型的子类型（subtype），这里 `extends` 同时表示类的继承和接口的实现
- 可以有多个限定，用 `&` 分隔：`<T extends Comparable & Serializable>`

```java
// 限定 T 必须实现 Comparable 接口
public static <T extends Comparable> T min(T[] a) {
    if (a == null || a.length == 0) return null;
    T smallest = a[0];
    for (int i = 1; i < a.length; i++) {
        if (smallest.compareTo(a[i]) > 0)
            smallest = a[i];
    }
    return smallest;
}
```

<aside>
💡

限定中可以有多个接口，但**至多一个类**，且类必须放在限定列表的**第一个**。

</aside>

---

## 8.5 泛型代码和虚拟机

### 类型擦除

- 虚拟机没有泛型类型对象，所有对象都属于普通类
- 编译器会对泛型类型进行**类型擦除（type erasure）**：
    - 无限定 → 替换为 `Object`
    - 有限定 → 替换为**第一个限定类型**
- 例如 `Pair<T>` 擦除后变成原始类型 `Pair`，其中 `T` 变成 `Object`

### 转换泛型表达式

- 编译器在擦除返回类型后自动插入**强制类型转换**
- 例：`Pair<Employee>` 的 `getFirst()` 擦除后返回 `Object`，编译器插入转换为 `Employee` 的代码

### 桥方法

- 类型擦除可能与多态冲突，编译器会生成**桥方法（bridge method）** 来保持多态
- 例如子类重写泛型方法时，编译器在擦除后的类中生成桥方法以正确覆盖父类方法

<aside>
📌

**关于 Java 泛型，需要记住以下事实：**

- 虚拟机中没有泛型，只有普通的类和方法
- 所有的类型参数都会被替换为它们的限定类型
- 会合成桥方法来保持多态
- 为保持类型安全性，必要时会插入强制类型转换
</aside>

---

## 8.6 限制与局限性

| **限制** | **说明** |
| --- | --- |
| 不能用基本类型实例化类型参数 | 没有 `Pair<int>`，只有 `Pair<Integer>` |
| 运行时类型查询只适用于原始类型 | `instanceof`、`getClass()` 返回的都是原始类型 |
| 不能创建参数化类型的数组 | `new Pair<String>[10]` 不合法 |
| Varargs 警告 | 向参数可变的方法传递泛型类型实例会收到警告，可用 `@SafeVarargs` 标注 |
| 不能实例化类型变量 | `new T()` 不合法 |
| 不能构造泛型数组 | `new T[10]` 不合法 |
| 泛型类的静态上下文中类型变量无效 | 静态字段或方法中不能引用类型变量 |
| 不能抛出或捕获泛型类的实例 | 泛型类不能扩展 `Throwable` |
| 可以取消对检查型异常的检查 | 利用泛型可绕过检查型异常机制（不推荐） |
| 注意擦除后的冲突 | 擦除可能导致方法签名冲突 |

---

## 8.7 泛型类型的继承规则

- `Pair<Manager>` **不是** `Pair<Employee>` 的子类，即使 `Manager` 是 `Employee` 的子类
- 泛型类型之间没有继承关系（**不可协变**）
- 这是为了保证类型安全

```java
// 这是不合法的：
Pair<Manager> managerPair = new Pair<>(ceo, cfo);
Pair<Employee> employeePair = managerPair; // ❌ 编译错误
```

<aside>
⚠️

数组有协变性（`Manager[]` 是 `Employee[]` 的子类型），但泛型没有。这是 Java 泛型更安全的设计。

</aside>

---

## 8.8 通配符类型

### 上界通配符 `? extends`

- `Pair<? extends Employee>` 表示任何泛型 `Pair` 类型，其类型参数是 `Employee` 的子类
- **可以读取**（返回 `Employee`），**不能写入**（编译器无法确定具体类型）

```java
public static void printBuddies(Pair<? extends Employee> p) {
    Employee first = p.getFirst();   // ✅ 可以读取
    // p.setFirst(new Employee()); // ❌ 不能写入
}
```

### 下界通配符 `? super`

- `Pair<? super Manager>` 表示类型参数是 `Manager` 的超类
- **可以写入**（传入 `Manager` 或其子类），**不能安全读取**（只能读为 `Object`）

```java
public static void minmaxBonus(Manager[] a, Pair<? super Manager> result) {
    result.setFirst(a[0]);  // ✅ 可以写入 Manager
    // Manager m = result.getFirst(); // ❌ 只能读为 Object
}
```

### PECS 原则

<aside>
🌟

**Producer Extends, Consumer Super（PECS）：**

- 如果需要**读取**数据（生产者），使用 `? extends`
- 如果需要**写入**数据（消费者），使用 `? super`
</aside>

### 无限定通配符 `?`

- `Pair<?>` 表示任意类型的 `Pair`
- 不能调用 `setFirst` 方法（`Object` 也不行），但可以调用 `getFirst`（返回 `Object`）
- 适用于只关心结构、不关心具体类型的简单操作

```java
public static boolean hasNulls(Pair<?> p) {
    return p.getFirst() == null || p.getSecond() == null;
}
```

### 通配符捕获

- 编译器可以通过辅助方法"捕获"通配符类型，将 `?` 绑定到一个具体的类型变量
- 典型用法：定义一个私有的泛型辅助方法，让编译器推断通配符的实际类型

```java
public static void swap(Pair<?> p) {
    swapHelper(p); // 编译器捕获 ? 并绑定到 T
}

private static <T> void swapHelper(Pair<T> p) {
    T t = p.getFirst();
    p.setFirst(p.getSecond());
    p.setSecond(t);
}
```

---

## 8.9 反射和泛型

- Java 泛型在运行时会被擦除，但 `Class` 类本身是泛型的：`Class<T>`
- 可利用反射获取泛型相关信息：
    - `Type` 接口及其子接口：`Class`、`TypeVariable`、`WildcardType`、`ParameterizedType`、`GenericArrayType`
- 实用场景：
    - `Class<T>` 的类型参数让某些方法（如 `cast()`、`newInstance()`）更加类型安全
    - 可用 `TypeLiteral` 模式在运行时获取泛型类型信息

```java
// Class<T> 的实际应用
Employee e = Employee.class.cast(obj); // 类型安全的转换
// 等价于 (Employee) obj，但在泛型上下文中更有用
```

---

## 本章小结

<aside>
✅

**核心要点回顾：**

- 泛型用类型参数实现代码复用，提升**类型安全**
- 类型擦除使泛型在 JVM 层面兼容旧代码
- 通配符（`extends` / `super` / `?`）提供灵活的类型限制
- 记住 **PECS 原则**：读用 extends，写用 super
- 泛型有诸多限制（不能用基本类型、不能创建泛型数组等）
- 泛型类型之间**没有继承关系**
</aside>