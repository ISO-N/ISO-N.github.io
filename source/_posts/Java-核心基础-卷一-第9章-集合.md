---
title: 第9章 集合
date: 2026-02-17 03:20:31
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷一
---
## 9.1 Java 集合框架

Java 集合框架（Java Collections Framework）是一组用于表示和操作集合的统一架构，位于 `java.util` 包中。

### 核心接口层次结构

```
Collection
├── List（有序、可重复）
├── Set（无序、不可重复）
│   └── SortedSet
│       └── NavigableSet
└── Queue（队列）
    └── Deque（双端队列）

Map（键值对映射，独立于 Collection）
├── SortedMap
│   └── NavigableMap
```

### 迭代器（Iterator）

- `Iterator<E>` 接口用于遍历集合元素
- 核心方法：
    - `hasNext()` — 是否还有下一个元素
    - `next()` — 返回下一个元素
    - `remove()` — 删除上一次 `next()` 返回的元素
- `for-each` 循环底层就是使用迭代器实现的

```java
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
    String element = iter.next();
    // 处理 element
}
```

<aside>
⚠️

调用 `remove()` 之前必须先调用 `next()`，否则会抛出 `IllegalStateException`。

</aside>

---

## 9.2 集合框架中的接口

### Collection 接口

所有集合类的根接口，定义了通用操作：

- `add(E)` / `addAll(Collection)`
- `remove(Object)` / `removeAll(Collection)`
- `contains(Object)` / `containsAll(Collection)`
- `size()` / `isEmpty()` / `clear()`
- `iterator()` / `toArray()`

### List 接口

有序集合，允许重复元素，支持按索引访问：

- `get(int index)` / `set(int index, E element)`
- `add(int index, E element)` / `remove(int index)`
- `indexOf(Object)` / `lastIndexOf(Object)`
- `subList(int fromIndex, int toIndex)`
- `ListIterator` — 支持双向遍历和添加操作

### Set 接口

不允许重复元素（依据 `equals()` 判断）

### Map 接口

键值对映射，键不能重复：

- `put(K, V)` / `get(K)` / `remove(K)`
- `containsKey(K)` / `containsValue(V)`
- `keySet()` / `values()` / `entrySet()`
- `getOrDefault(K, V)` / `putIfAbsent(K, V)`
- `merge()` / `compute()` / `computeIfAbsent()`

---

## 9.3 具体集合类

### 常用集合类一览

| **集合类** | **接口** | **底层结构** | **特点** |
| --- | --- | --- | --- |
| `ArrayList` | List | 动态数组 | 随机访问快，增删慢 |
| `LinkedList` | List / Deque | 双向链表 | 增删快，随机访问慢 |
| `HashSet` | Set | 哈希表 | 无序，查找 O(1) |
| `TreeSet` | SortedSet | 红黑树 | 有序，查找 O(log n) |
| `LinkedHashSet` | Set | 哈希表 + 链表 | 保持插入顺序 |
| `HashMap` | Map | 哈希表 | 无序，允许 null 键/值 |
| `TreeMap` | SortedMap | 红黑树 | 按键排序 |
| `LinkedHashMap` | Map | 哈希表 + 链表 | 保持插入/访问顺序 |
| `PriorityQueue` | Queue | 堆（最小堆） | 按优先级出队 |
| `ArrayDeque` | Deque | 循环数组 | 双端队列，栈和队列的首选 |

### ArrayList

- 底层为可扩容的 `Object[]` 数组
- 默认初始容量为 **10**，扩容为原来的 **1.5 倍**
- 适合频繁随机访问的场景

```java
ArrayList<String> list = new ArrayList<>();
list.add("Alice");
list.add("Bob");
list.get(0); // "Alice"
```

### LinkedList

- 实现了 `List` 和 `Deque` 接口
- 适合频繁插入/删除的场景
- 可用作栈、队列、双端队列

```java
LinkedList<String> list = new LinkedList<>();
list.addFirst("A");
list.addLast("B");
list.removeFirst();
```

---

## 9.4 散列集（HashSet）

- 基于 **哈希表**（实际是 `HashMap` 的封装）
- 元素无序，不允许重复
- `add()`、`contains()`、`remove()` 平均时间复杂度 **O(1)**

### hashCode 与 equals 的契约

<aside>
📌

- 如果 `a.equals(b)` 为 `true`，则 `a.hashCode() == b.hashCode()` **必须**成立
- 如果 `hashCode` 相同，`equals` **不一定**为 `true`（哈希冲突）
- 重写 `equals()` 时**必须**同时重写 `hashCode()`
</aside>

### 哈希冲突解决

- Java 8 之前：**链地址法**（链表）
- Java 8 及之后：链表长度超过 **8** 时转为**红黑树**，低于 **6** 时退化回链表

---

## 9.5 树集（TreeSet）

- 基于**红黑树**实现，元素自动排序
- 插入、删除、查找时间复杂度 **O(log n)**
- 元素必须实现 `Comparable` 接口，或在构造时提供 `Comparator`

```java
TreeSet<String> set = new TreeSet<>();
set.add("Charlie");
set.add("Alice");
set.add("Bob");
// 遍历结果：Alice, Bob, Charlie（自然排序）
```

### Comparable vs Comparator

| **对比项** | **Comparable** | **Comparator** |
| --- | --- | --- |
| 所在包 | `java.lang` | `java.util` |
| 方法 | `compareTo(T o)` | `compare(T o1, T o2)` |
| 侵入性 | 需修改类本身 | 外部定义，无需修改类 |
| 排序策略 | 单一自然排序 | 可定义多种排序策略 |

---

## 9.6 队列与双端队列

### Queue 接口

| **操作** | **抛异常** | **返回特殊值** |
| --- | --- | --- |
| 入队 | `add(e)` | `offer(e)` |
| 出队 | `remove()` | `poll()` |
| 查看队首 | `element()` | `peek()` |

### PriorityQueue（优先级队列）

- 底层为**最小堆**，每次出队的是优先级最高（最小）的元素
- 不是 FIFO，而是按优先级排列
- 插入和删除时间复杂度 **O(log n)**

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.add(3);
pq.add(1);
pq.add(2);
pq.poll(); // 返回 1
```

### Deque（双端队列）

- `ArrayDeque` 是首选实现（比 `LinkedList` 更高效）
- 可用作**栈**（`push` / `pop`）和**队列**（`offer` / `poll`）

---

## 9.7 映射（Map）

### HashMap

- 基于哈希表，允许 `null` 键和 `null` 值
- 无序，线程不安全
- 默认初始容量 **16**，负载因子 **0.75**，扩容为原来的 **2 倍**

```java
Map<String, Integer> map = new HashMap<>();
map.put("Alice", 90);
map.put("Bob", 85);
map.getOrDefault("Charlie", 0); // 返回 0
```

### TreeMap

- 基于红黑树，按键的自然顺序或自定义 `Comparator` 排序
- 不允许 `null` 键

### LinkedHashMap

- 保持**插入顺序**或**访问顺序**
- 访问顺序模式可用于实现 **LRU 缓存**

```java
LinkedHashMap<String, Integer> lhm = new LinkedHashMap<>(16, 0.75f, true);
// 第三个参数 true 表示按访问顺序排列
```

### Map 的遍历方式

```java
// 方式一：entrySet（推荐）
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

// 方式二：keySet
for (String key : map.keySet()) {
    System.out.println(key + " = " + map.get(key));
}

// 方式三：forEach（Java 8+）
map.forEach((k, v) -> System.out.println(k + " = " + v));
```

---

## 9.8 视图与包装器

### 不可修改视图

- `Collections.unmodifiableList(list)` — 返回只读视图
- `Collections.unmodifiableMap(map)`
- `Collections.unmodifiableSet(set)`
- 对视图的修改操作会抛出 `UnsupportedOperationException`

### 同步视图

- `Collections.synchronizedList(list)`
- `Collections.synchronizedMap(map)`
- 为集合添加线程安全的包装

### 子范围视图

- `list.subList(from, to)` — 返回原列表的一部分视图
- `sortedSet.headSet(to)` / `tailSet(from)` / `subSet(from, to)`
- 对视图的修改会**反映到原集合**

<aside>
💡

视图并不创建新的集合副本，而是引用原集合。对视图的修改会直接影响底层集合。

</aside>

---

## 9.9 算法（Collections 工具类）

`java.util.Collections` 提供了大量静态工具方法：

### 排序与查找

- `Collections.sort(list)` — 归并排序（稳定排序）
- `Collections.binarySearch(list, key)` — 二分查找（要求有序）
- `Collections.reverse(list)` — 反转
- `Collections.shuffle(list)` — 随机打乱

### 极值

- `Collections.min(collection)`
- `Collections.max(collection)`

### 批量操作

- `Collections.fill(list, value)` — 用指定值填充
- `Collections.copy(dest, src)` — 复制
- `Collections.replaceAll(list, oldVal, newVal)` — 替换
- `Collections.frequency(collection, obj)` — 统计出现次数
- `Collections.disjoint(c1, c2)` — 判断是否无交集

---

## 9.10 遗留的集合类

<aside>
📜

以下类属于早期 Java 遗留实现，**不推荐在新代码中使用**，了解即可。

</aside>

### Hashtable

- 线程安全（方法加 `synchronized`），但性能差
- 不允许 `null` 键和 `null` 值
- 现代替代方案：`ConcurrentHashMap`

### Vector

- 线程安全的动态数组
- 现代替代方案：`ArrayList`（配合 `Collections.synchronizedList`）

### Stack

- 继承自 `Vector`
- 现代替代方案：`ArrayDeque`

### Enumeration

- 早期的迭代器接口
- 现代替代方案：`Iterator`

### Properties

- 继承自 `Hashtable<Object, Object>`
- 用于读写 `.properties` 配置文件
- 这是唯一仍常用的遗留类

---

## 📝 本章核心要点速记

<aside>
🎯

- **ArrayList** vs **LinkedList**：随机访问多选 ArrayList，频繁增删选 LinkedList
- **HashSet** 要求正确重写 `hashCode()` 和 `equals()`
- **TreeSet / TreeMap** 元素需可比较（Comparable 或 Comparator）
- **HashMap** 默认容量 16，负载因子 0.75，Java 8 后链表长度 > 8 转红黑树
- **PriorityQueue** 是堆实现，不是 FIFO
- **ArrayDeque** 优于 `Stack` 和 `LinkedList` 作为栈/队列使用
- 遍历 Map 推荐使用 `entrySet()`
- 遗留类（Hashtable、Vector、Stack）应避免使用
</aside>