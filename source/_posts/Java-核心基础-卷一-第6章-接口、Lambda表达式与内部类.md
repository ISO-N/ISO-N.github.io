---
title: 第6章 接口、Lambda表达式与内部类
date: 2026-02-17 03:20:29
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷一
---
## 6.1 接口

### 接口的概念

- 接口（interface）用来描述类应该做什么，而不指定具体怎么做
- 一个类可以实现（implement）一个或多个接口
- 接口中的所有方法默认为 `public`，声明时不需要显式写出

```java
public interface Comparable<T> {
    int compareTo(T other);
}
```

### 接口的属性

- 接口**不能被实例化**，但可以声明接口类型的变量，该变量必须引用实现了该接口的类的对象
- 可以使用 `instanceof` 检查对象是否实现了某个接口
- 接口中可以定义**常量**（默认 `public static final`）
- 一个类可以实现多个接口，用逗号分隔：`class A implements B, C`

### 接口与抽象类的区别

- Java 不支持多重继承，但支持实现多个接口
- 抽象类可以有实例字段和构造器，接口不能有实例字段
- Java 8 之后接口可以有**默认方法**和**静态方法**

### 默认方法

- 用 `default` 修饰符标记，提供默认实现
- 主要用于"接口演化"——向已有接口添加新方法而不破坏现有实现

```java
public interface Collection {
    default boolean isEmpty() {
        return size() == 0;
    }
}
```

### 默认方法冲突的解决规则

1. **超类优先**：如果超类提供了具体方法，同名的接口默认方法会被忽略
2. **接口冲突**：如果两个接口都提供了同名默认方法，实现类**必须**手动覆盖解决冲突

```java
class Student implements Person, Named {
    public String getName() {
        return Person.super.getName(); // 选择其中一个
    }
}
```

### 接口中的静态方法与私有方法

- Java 8 允许接口中定义**静态方法**
- Java 9 允许接口中定义**私有方法**（用于默认方法之间共享代码）

---

## 6.2 接口示例

### Comparable 接口

- 对象排序时需实现 `Comparable<T>` 接口的 `compareTo` 方法
- 返回值：负数（小于）、零（等于）、正数（大于）

```java
public class Employee implements Comparable<Employee> {
    public int compareTo(Employee other) {
        return Double.compare(salary, other.salary);
    }
}
```

### Cloneable 接口

- `Object.clone()` 是 `protected` 的浅拷贝方法
- **浅拷贝**：只复制基本类型字段，对象字段仍共享引用
- **深拷贝**：需要手动克隆所有可变子对象
- 实现 `Cloneable` 接口（标记接口）表示允许克隆

```java
public class Employee implements Cloneable {
    public Employee clone() throws CloneNotSupportedException {
        Employee cloned = (Employee) super.clone();
        cloned.hireDay = (Date) hireDay.clone(); // 深拷贝可变字段
        return cloned;
    }
}
```

---

## 6.3 Lambda 表达式

### 基本语法

- Lambda 表达式是一个**可传递的代码块**，可以在以后执行一次或多次
- 语法：`(参数) -> 表达式` 或 `(参数) -> { 语句; }`

```java
// 无参数
() -> System.out.println("Hello")

// 单参数（可省略括号）
event -> System.out.println(event)

// 多参数
(first, second) -> first.length() - second.length()

// 带类型声明
(String first, String second) -> first.length() - second.length()
```

### 函数式接口

- **只有一个抽象方法**的接口称为函数式接口
- Lambda 表达式可以转换为函数式接口
- 常用函数式接口：

| 接口 | 方法 | 用途 |
| --- | --- | --- |
| `Runnable` | `void run()` | 无参无返回值 |
| `Supplier<T>` | `T get()` | 无参有返回值 |
| `Consumer<T>` | `void accept(T)` | 有参无返回值 |
| `Function<T,R>` | `R apply(T)` | 有参有返回值 |
| `Predicate<T>` | `boolean test(T)` | 判断条件 |

### 方法引用

- 方法引用是 Lambda 的简写形式，用 `::` 操作符
- 三种形式：
    - **对象::实例方法** → `System.out::println`
    - **类::静态方法** → `Math::pow`
    - **类::实例方法** → `String::compareToIgnoreCase`（第一个参数作为调用者）

```java
Arrays.sort(strings, String::compareToIgnoreCase);
// 等价于
Arrays.sort(strings, (x, y) -> x.compareToIgnoreCase(y));
```

### 构造器引用

- `类名::new` 可以引用构造器

```java
ArrayList<String> names = ...;
Stream<Person> stream = names.stream().map(Person::new);
```

### 变量作用域

- Lambda 可以捕获外部的**自由变量**，但该变量必须是**事实最终变量**（effectively final）
- Lambda 中不能修改捕获的变量值
- Lambda 中的 `this` 指的是**创建 Lambda 的方法所属的对象**

---

## 6.4 内部类

### 内部类的作用

- 内部类可以访问外部类的所有成员（包括 `private`）
- 内部类对同一包中的其他类隐藏
- 适合用于回调等场景

### 成员内部类

```java
public class TalkingClock {
    private int interval;
    private boolean beep;

    public class TimePrinter implements ActionListener {
        public void actionPerformed(ActionEvent event) {
            System.out.println("At the tone, the time is " + Instant.now());
            if (beep) Toolkit.getDefaultToolkit().beep();
            // beep 引用了外部类的字段
        }
    }
}
```

- 内部类对象持有外部类对象的引用：`OuterClass.this`
- 创建内部类对象：[`outerObject.new](http://outerObject.new) InnerClass()`

### 局部内部类

- 定义在方法内部，作用域仅限于该方法
- 不使用访问修饰符
- 可以访问方法的**事实最终**局部变量

```java
public void start() {
    class TimePrinter implements ActionListener {
        public void actionPerformed(ActionEvent event) {
            System.out.println("The time is " + Instant.now());
            if (beep) // 访问外部方法的局部变量
                Toolkit.getDefaultToolkit().beep();
        }
    }
    var listener = new TimePrinter();
    new Timer(interval, listener).start();
}
```

### 匿名内部类

- 没有名字的局部内部类，定义和实例化同时完成
- 语法：`new SuperType(参数) { 类体 }`

```java
var listener = new ActionListener() {
    public void actionPerformed(ActionEvent event) {
        System.out.println("The time is " + Instant.now());
    }
};
```

> 💡 Java 8 之后，大多数匿名内部类场景可以用 **Lambda 表达式**替代（前提是目标类型为函数式接口）
> 

### 静态内部类

- 用 `static` 修饰，**不持有**外部类对象的引用
- 只能访问外部类的**静态成员**
- 适用于不需要访问外部类实例的场景

```java
public class ArrayAlg {
    public static class Pair {
        private double first;
        private double second;
        // ...
    }

    public static Pair minmax(double[] values) {
        // ...
        return new Pair(min, max);
    }
}
```

---

## 6.5 服务加载器

- `ServiceLoader` 用于加载服务提供者（SPI 机制）
- 步骤：
    1. 定义服务接口
    2. 提供实现类
    3. 在 `META-INF/services/` 下创建配置文件
    4. 使用 `ServiceLoader.load()` 加载

```java
ServiceLoader<Cipher> loader = ServiceLoader.load(Cipher.class);
for (Cipher cipher : loader) {
    // 遍历所有实现
}
```

---

## 6.6 代理

### 代理的概念

- 代理（Proxy）可以在**运行时**创建实现了一组接口的新类
- 适用于在编译时无法确定需要实现哪些接口的场景

### 创建代理对象

```java
Object proxy = Proxy.newProxyInstance(
    classLoader,        // 类加载器
    interfaces,         // 要实现的接口数组
    invocationHandler   // 调用处理器
);
```

### InvocationHandler

- 所有方法调用都会转发到处理器的 `invoke` 方法

```java
public class TraceHandler implements InvocationHandler {
    private Object target;

    public TraceHandler(Object t) { target = t; }

    public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
        System.out.println("Calling: " + method.getName());
        return method.invoke(target, args); // 转发给目标对象
    }
}
```

### 代理的特性

- 代理类在运行时创建，一旦创建就是常规类
- 所有代理类都继承自 `Proxy`
- 同一类加载器和接口数组只会生成一个代理类
- 代理类总是 `public` 和 `final`