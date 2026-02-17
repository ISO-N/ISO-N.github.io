---
title: 第1章 Java SE 8的流库
date: 2026-02-17 15:12:00
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 1.1 从迭代到流的操作

- 流（Stream）提供了一种以**声明式**方式处理集合数据的方法，替代传统的 for 循环迭代
- 流的核心思想：**做什么，而非怎么做**

```java
// 传统迭代
long count = 0;
for (String w : words) {
    if (w.length() > 12) count++;
}

// 流操作
long count = words.stream()
    .filter(w -> w.length() > 12)
    .count();
```

- 流与集合的区别：
    - 流**不存储元素**，元素来自底层数据源（集合、数组、生成器等）
    - 流操作**不修改数据源**，而是返回新的流
    - 流操作是**惰性执行**的，尽可能延迟执行
    - 流可以是**无限的**

---

## 1.2 流的创建

- **从集合创建**：[`Collection.stream](http://Collection.stream)()` / `Collection.parallelStream()`
- **从数组创建**：[`Arrays.stream](http://Arrays.stream)(array)`
- **静态方法**：
    - `Stream.of(values...)` — 从可变参数创建
    - `Stream.empty()` — 创建空流
    - `Stream.generate(supplier)` — 无限流
    - `Stream.iterate(seed, f)` — 迭代生成无限流

```java
Stream<String> words = Stream.of("a", "b", "c");
Stream<Double> randoms = Stream.generate(Math::random);  // 无限流
Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
```

- **基本类型流**：`IntStream`、`LongStream`、`DoubleStream`
    - `IntStream.range(0, 100)` — 不包含上限
    - `IntStream.rangeClosed(0, 100)` — 包含上限

---

## 1.3 filter、map 与 flatMap

### filter

- 接受一个 `Predicate<T>`，保留满足条件的元素

```java
stream.filter(w -> w.length() > 5)
```

### map

- 对每个元素应用一个函数，产生一个新流

```java
stream.map(String::toLowerCase)
stream.map(s -> s.charAt(0))
```

### flatMap

- 将每个元素映射为一个流，然后将所有流**扁平化**合并为一个流
- 常用于"一对多"映射场景

```java
// 将每个单词拆为字符流
stream.flatMap(w -> w.chars().mapToObj(c -> (char) c))
```

---

## 1.4 抽取子流与组合流

- `stream.limit(n)` — 截取前 n 个元素
- `stream.skip(n)` — 跳过前 n 个元素
- `stream.takeWhile(predicate)` — 取满足条件的前缀（Java 9+）
- `stream.dropWhile(predicate)` — 丢弃满足条件的前缀（Java 9+）
- `Stream.concat(a, b)` — 连接两个流
- `stream.distinct()` — 去重
- `stream.sorted()` — 自然排序
- `stream.sorted(comparator)` — 自定义排序
- `stream.peek(action)` — 对每个元素执行操作（常用于调试）

---

## 1.5 简单约简（终结操作）

- 约简（Reduction）是将流中的元素组合成一个值的**终结操作**
- `count()` — 元素个数
- `max(comparator)` / `min(comparator)` — 返回 `Optional`
- `findFirst()` — 第一个元素
- `findAny()` — 任意一个元素（并行流中更高效）
- `anyMatch(predicate)` — 是否存在匹配元素
- `allMatch(predicate)` — 是否全部匹配
- `noneMatch(predicate)` — 是否全不匹配

---

## 1.6 Optional 类型

- `Optional<T>` 是一个包装器，表示值**可能存在也可能不存在**
- 正确使用 Optional 可以避免 `NullPointerException`

### 创建

```java
Optional.of(value)          // value 不能为 null
Optional.ofNullable(value)  // value 可以为 null
Optional.empty()            // 空 Optional
```

### 消费

```java
opt.ifPresent(v -> System.out.println(v));
opt.ifPresentOrElse(v -> use(v), () -> fallback());  // Java 9+
```

### 获取值

```java
opt.orElse(defaultVal)
opt.orElseGet(() -> computeDefault())
opt.orElseThrow()                         // NoSuchElementException
opt.orElseThrow(IllegalStateException::new)
```

### 转换

```java
opt.map(String::toUpperCase)
opt.flatMap(this::lookup)   // 当转换函数返回 Optional 时使用
opt.filter(predicate)
```

<aside>
⚠️

避免使用 `opt.get()`，它在值不存在时会抛异常。优先使用 `orElse` / `orElseThrow` 系列方法。

</aside>

---

## 1.7 收集结果

### 常用收集方式

```java
stream.toArray()                          // Object[]
stream.toArray(String[]::new)             // String[]
stream.collect(Collectors.toList())       // List
stream.collect(Collectors.toSet())        // Set
stream.toList()                           // Java 16+ 不可变 List
stream.collect(Collectors.joining(", "))  // 字符串拼接
```

### Collectors 工具类

| 方法 | 说明 |
| --- | --- |
| `toList()` | 收集到 List |
| `toSet()` | 收集到 Set |
| `toMap(keyFunc, valueFunc)` | 收集到 Map |
| `joining(delimiter)` | 字符串拼接 |
| `groupingBy(classifier)` | 分组 |
| `partitioningBy(predicate)` | 二分（true/false 分组） |
| `summarizingInt/Long/Double` | 统计摘要（count, sum, min, avg, max） |

---

## 1.8 收集到映射表（Map）

```java
// 基本用法
Map<Integer, String> idToName = people.stream()
    .collect(Collectors.toMap(Person::getId, Person::getName));

// 处理键冲突
Map<String, String> map = stream.collect(
    Collectors.toMap(keyFunc, valueFunc, (old, newVal) -> old));

// 指定 Map 类型
TreeMap<K, V> map = stream.collect(
    Collectors.toMap(keyFunc, valueFunc, mergeFunc, TreeMap::new));
```

<aside>
💡

如果存在重复键且未提供合并函数，`toMap` 会抛出 `IllegalStateException`。

</aside>

---

## 1.9 群组和分区

### groupingBy

```java
// 按首字母分组
Map<Character, List<String>> groups = words.stream()
    .collect(Collectors.groupingBy(w -> w.charAt(0)));

// 分组 + 下游收集器
Map<String, Long> cityCount = people.stream()
    .collect(Collectors.groupingBy(Person::getCity, Collectors.counting()));
```

### partitioningBy

```java
// 按条件分为 true/false 两组
Map<Boolean, List<String>> parts = words.stream()
    .collect(Collectors.partitioningBy(w -> w.length() > 5));
```

### 常用下游收集器

- `counting()` — 计数
- `summing[Int|Long|Double]()` — 求和
- `maxBy(comparator)` / `minBy(comparator)`
- `mapping(func, downstream)` — 先转换再收集
- `toSet()` — 收集到 Set 而非 List

---

## 1.10 基本类型流

- `IntStream`、`LongStream`、`DoubleStream` 避免自动装箱开销
- 转换方法：
    - `mapToInt()` / `mapToLong()` / `mapToDouble()` — 对象流 → 基本类型流
    - `boxed()` — 基本类型流 → 对象流
- 额外方法：`sum()`、`average()`、`max()`、`min()`、`summaryStatistics()`

```java
IntStream ints = IntStream.rangeClosed(1, 100);
int sum = ints.sum();  // 5050

IntSummaryStatistics stats = stream.mapToInt(String::length).summaryStatistics();
// stats.getAverage(), stats.getMax(), stats.getCount() ...
```

---

## 1.11 并行流

- `collection.parallelStream()` 或 `stream.parallel()` 创建并行流
- 底层使用 **Fork/Join 框架**，利用多核并行计算
- 注意事项：
    - 传递给并行流的操作必须是**无状态**和**不干扰**的
    - 操作顺序可能不确定（使用 `forEachOrdered` 保证顺序）
    - 小数据集或简单操作可能**更慢**（线程调度开销）
    - 避免在并行流中使用**共享可变状态**

```java
long count = words.parallelStream()
    .filter(w -> w.length() > 12)
    .count();
```

<aside>
⚠️

不要盲目使用并行流。只有数据量大、操作耗时、无共享状态时，并行流才能带来性能提升。先用基准测试验证。

</aside>

---

## 1.12 小结

| 类别 | 关键方法 | 说明 |
| --- | --- | --- |
| 创建 | `stream()`, `of()`, `generate()`, `iterate()` | 从集合、数组或函数创建流 |
| 中间操作 | `filter`, `map`, `flatMap`, `sorted`, `distinct`, `limit`, `skip` | 惰性执行，返回新流 |
| 终结操作 | `count`, `min`, `max`, `findFirst`, `forEach`, `collect`, `reduce` | 触发实际计算，产生结果 |
| 收集器 | `toList`, `toSet`, `toMap`, `groupingBy`, `joining` | 将流结果收集到集合或字符串 |
| 并行 | `parallelStream()`, `parallel()` | 利用多核加速，需注意线程安全 |