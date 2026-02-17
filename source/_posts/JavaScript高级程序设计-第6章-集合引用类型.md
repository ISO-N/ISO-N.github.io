---
title: 第6章 集合引用类型
date: 2026-02-17 15:02:00
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 6.1 Object

Object 是 ECMAScript 中最常用的引用类型之一，适合存储和传递数据。

### 创建方式

```jsx
// 1. new 构造函数
let person = new Object();
person.name = "Nicholas";
person.age = 29;

// 2. 对象字面量（更常用）
let person = {
  name: "Nicholas",
  age: 29
};

// 属性名可以是字符串或数值
let person = {
  "name": "Nicholas",
  5: true   // 数值属性会自动转为字符串
};
```

### 属性访问

```jsx
// 点语法（推荐）
person.name

// 中括号语法（支持变量和特殊字符）
person["name"]
let key = "name";
person[key]
person["first name"]  // 包含空格时必须用中括号
```

---

## 6.2 Array

Array 是 ECMAScript 中最常用的集合类型，元素可以是**任意类型**，且数组**大小动态调整**。

### 6.2.1 创建数组

```jsx
// 1. Array 构造函数
let colors = new Array();
let colors = new Array(20);         // 创建 length 为 20 的数组
let colors = new Array("red", "blue", "green");

// 2. 数组字面量
let colors = ["red", "blue", "green"];
let values = [1, 2, ];              // 末尾逗号：length 为 2

// 3. Array.from()：将类数组结构转换为数组
Array.from("Matt");                 // ["M", "a", "t", "t"]
Array.from(new Map().set(1, 2));    // [[1, 2]]
Array.from(new Set().add(1).add(2));// [1, 2]

// Array.from() 的第二个参数：映射函数
Array.from([1, 2, 3], x => x ** 2); // [1, 4, 9]

// 4. Array.of()：将一组参数转换为数组
Array.of(1, 2, 3, 4);              // [1, 2, 3, 4]
```

### 6.2.2 数组空位

```jsx
const options = [1, , , , 5];  // 中间产生空位
options.length;   // 5
options;          // [1, undefined, undefined, undefined, 5]
```

<aside>
⚠️

ES6 新方法普遍将空位视为 `undefined`，但由于行为不一致，**应避免使用数组空位**。

</aside>

### 6.2.3 数组索引

```jsx
let colors = ["red", "blue", "green"];
colors[0];         // "red"
colors[2] = "black"; // 修改第三项
colors[3] = "brown"; // 添加第四项

// length 不是只读的
colors.length = 2;   // 截断到前两项
colors.length = 10;  // 扩展，新位置为 undefined
```

### 6.2.4 检测数组

```jsx
// 推荐方法
Array.isArray(value);

// instanceof 在多个全局执行上下文中可能出问题
value instanceof Array;
```

### 6.2.5 迭代器方法

```jsx
const a = ["foo", "bar", "baz"];

a.keys();    // [0, 1, 2]        — 索引迭代器
a.values();  // ["foo", "bar", "baz"] — 值迭代器
a.entries(); // [[0,"foo"], [1,"bar"], [2,"baz"]] — 键值对迭代器

// 解构赋值
for (const [idx, element] of a.entries()) {
  console.log(idx, element);
}
```

### 6.2.6 复制和填充

```jsx
// fill()：用固定值填充
const zeroes = [0, 0, 0, 0, 0];
zeroes.fill(5);          // [5, 5, 5, 5, 5]
zeroes.fill(6, 3);       // [5, 5, 5, 6, 6]  — 从索引 3 开始
zeroes.fill(0, 1, 3);    // [5, 0, 0, 6, 6]  — 填充索引 [1, 3)

// copyWithin()：浅复制数组内部的一部分到同一数组的另一位置
let ints = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
ints.copyWithin(5);       // [0,1,2,3,4, 0,1,2,3,4] — 从0复制到索引5
ints.copyWithin(0, 5);    // [5,6,7,8,9, 5,6,7,8,9] — 从索引5复制到索引0
```

### 6.2.7 转换方法

```jsx
let colors = ["red", "blue", "green"];

colors.toString();       // "red,blue,green"
colors.valueOf();        // ["red", "blue", "green"]（返回数组本身）
colors.join("||");       // "red||blue||green"
```

### 6.2.8 栈方法（LIFO）

```jsx
let stack = [];
stack.push("red", "green");  // 压入末尾，返回新长度 2
stack.pop();                  // 弹出末尾 "green"，返回该项
```

### 6.2.9 队列方法（FIFO）

```jsx
let queue = [];
queue.push("red", "green");
queue.shift();                // 移除并返回首项 "red"

// 反向队列
queue.unshift("black");       // 在开头插入，返回新长度
queue.pop();                  // 移除末尾
```

### 6.2.10 排序方法

```jsx
let values = [1, 2, 3, 4, 5];
values.reverse();             // [5, 4, 3, 2, 1]

// sort() 默认按字符串排序（会出问题）
let values = [0, 1, 5, 10, 15];
values.sort();                // [0, 1, 10, 15, 5]  ← 字符串比较

// 使用比较函数
values.sort((a, b) => a - b); // [0, 1, 5, 10, 15]  ← 升序
values.sort((a, b) => b - a); // [15, 10, 5, 1, 0]   ← 降序
```

### 6.2.11 操作方法

```jsx
// concat()：拼接数组（不修改原数组）
let colors = ["red", "green", "blue"];
let colors2 = colors.concat("yellow", ["black", "brown"]);
// ["red","green","blue","yellow","black","brown"]

// slice()：切片（不修改原数组）
let colors = ["red","green","blue","yellow","purple"];
colors.slice(1);       // ["green","blue","yellow","purple"]
colors.slice(1, 4);    // ["green","blue","yellow"]

// splice()：最强大的数组方法（修改原数组）
// 删除
let removed = colors.splice(0, 2);     // 删除前 2 项
// 插入
colors.splice(2, 0, "red", "purple");  // 在索引 2 处插入
// 替换
colors.splice(1, 1, "red", "purple");  // 替换索引 1 处的 1 项
```

### 6.2.12 搜索和位置方法

**严格相等搜索：**

```jsx
let numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];
numbers.indexOf(4);        // 3（从头搜索）
numbers.lastIndexOf(4);    // 5（从尾搜索）
numbers.includes(4);       // true（ES7）
```

**断言函数搜索：**

```jsx
const people = [
  { name: "Matt", age: 27 },
  { name: "Nicholas", age: 29 }
];

people.find((el, idx, arr) => el.age < 28);
// { name: "Matt", age: 27 }

people.findIndex((el, idx, arr) => el.age < 28);
// 0
```

### 6.2.13 迭代方法

```jsx
let numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];

// every()：每一项都返回 true 则返回 true
numbers.every((item) => item > 2);   // false

// some()：任意一项返回 true 则返回 true
numbers.some((item) => item > 2);    // true

// filter()：返回 true 的项组成新数组
numbers.filter((item) => item > 2);  // [3, 4, 5, 4, 3]

// map()：返回每次调用的结果组成新数组
numbers.map((item) => item * 2);     // [2, 4, 6, 8, 10, 8, 6, 4, 2]

// forEach()：仅遍历，无返回值
numbers.forEach((item) => { /* ... */ });
```

### 6.2.14 归并方法

```jsx
let values = [1, 2, 3, 4, 5];

// reduce()：从左向右归并
let sum = values.reduce((prev, cur) => prev + cur, 0);
// 15

// reduceRight()：从右向左归并
let sum = values.reduceRight((prev, cur) => prev + cur, 0);
// 15
```

---

## 6.3 定型数组（Typed Array）

定型数组是 ECMAScript 为提升向原生 API（如 WebGL）传递数据效率而新增的结构。

### 6.3.1 ArrayBuffer

`ArrayBuffer` 是所有定型数组的基础，表示一块**预分配的内存**。

```jsx
const buf = new ArrayBuffer(16);  // 分配 16 字节内存
buf.byteLength;                   // 16
```

<aside>
💡

`ArrayBuffer` 一经创建就不能调整大小，但可以用 `slice()` 复制部分到新的 `ArrayBuffer`。

</aside>

### 6.3.2 DataView

`DataView` 是对 `ArrayBuffer` 的底层读写接口，支持多种数值类型和**字节序控制**。

```jsx
const buf = new ArrayBuffer(16);
const view = new DataView(buf);

view.setInt32(0, 25);             // 在偏移量 0 写入 Int32 值 25
view.getInt32(0);                 // 25

// 支持的类型：Int8, Uint8, Int16, Uint16, Int32, Uint32, Float32, Float64
// 第三个参数控制字节序（true 为小端序）
view.getInt32(0, true);           // 小端序读取
```

### 6.3.3 定型数组类型

| 类型 | 字节数 | 描述 |
| --- | --- | --- |
| `Int8Array` | 1 | 有符号 8 位整数 |
| `Uint8Array` | 1 | 无符号 8 位整数 |
| `Uint8ClampedArray` | 1 | 无符号 8 位（溢出钳制） |
| `Int16Array` | 2 | 有符号 16 位整数 |
| `Uint16Array` | 2 | 无符号 16 位整数 |
| `Int32Array` | 4 | 有符号 32 位整数 |
| `Uint32Array` | 4 | 无符号 32 位整数 |
| `Float32Array` | 4 | 32 位 IEEE-754 浮点 |
| `Float64Array` | 8 | 64 位 IEEE-754 浮点 |

```jsx
// 创建方式
const ints = new Int16Array(2);         // 创建长度 2 的空数组
const ints2 = new Int16Array([2, 4]);   // 从数组初始化

// 定型数组支持大部分普通数组方法
ints.length;
ints.find(v => v > 0);
ints.map(v => v * 2);
// 但不支持：concat, pop, push, shift, splice, unshift
```

<aside>
⚠️

定型数组**长度不可变**，不能使用会改变数组长度的方法。可以使用 `set()` 和 `subarray()` 来操作。

</aside>

---

## 6.4 Map

`Map` 是 ES6 新增的键值对集合，与 `Object` 的主要区别是**键可以是任意类型**。

### 基本 API

```jsx
// 创建
const m = new Map();

// 也可通过可迭代对象初始化
const m = new Map([
  ["key1", "val1"],
  ["key2", "val2"],
  ["key3", "val3"]
]);

// 增删查改
m.set("name", "Matt");   // 设置键值对，返回 Map 自身（可链式调用）
m.get("name");            // "Matt"
m.has("name");            // true
m.delete("name");         // 删除单个
m.clear();                // 清空
m.size;                   // 键值对数量
```

### 顺序与迭代

Map 会维护**插入顺序**，支持多种迭代方式：

```jsx
const m = new Map([["key1", "val1"], ["key2", "val2"]]);

for (const [key, value] of m.entries()) {  // 或 m[Symbol.iterator]()
  console.log(key, value);
}

m.forEach((val, key) => console.log(key, val));
```

### Map vs Object 如何选择

| **特性** | **Object** | **Map** |
| --- | --- | --- |
| 键的类型 | 字符串 / Symbol | 任意类型 |
| 插入顺序 | 不保证（整数键除外） | 保证 |
| 大小 | 需手动计算 | `size` 属性 |
| 性能 | 频繁增删性能较差 | 频繁增删性能更优 |
| 内存占用 | 相同键值对数量下更多 | 更少（约少 50%） |
| 序列化 | 原生支持 JSON | 不支持 JSON |

---

## 6.5 WeakMap

`WeakMap` 的键**只能是对象或 Symbol**，且键是**弱引用**（不会阻止垃圾回收）。

```jsx
const wm = new WeakMap();
const key = { id: 1 };

wm.set(key, "val");
wm.get(key);            // "val"
wm.has(key);            // true
wm.delete(key);
```

<aside>
💡

**弱引用**意味着：如果键对象没有其他引用，垃圾回收器可以随时回收它，对应的键值对也会自动消失。因此 WeakMap **不可迭代**，也没有 `size` 和 `clear()`。

</aside>

### 典型用途

**1. 私有变量：**

```jsx
const privateData = new WeakMap();
class Person {
  constructor(name) {
    privateData.set(this, { name });
  }
  getName() {
    return privateData.get(this).name;
  }
}
```

**2. DOM 节点元数据：**

```jsx
const metadata = new WeakMap();
const loginButton = document.querySelector("#login");
metadata.set(loginButton, { disabled: true });
// 当 DOM 节点被移除后，关联的元数据也会被自动回收
```

---

## 6.6 Set

`Set` 是 ES6 新增的集合类型，类似于数组，但**值唯一**，不允许重复。

### 基本 API

```jsx
const s = new Set();

// 通过可迭代对象初始化
const s = new Set(["val1", "val2", "val3"]);

// 增删查
s.add("Matt");        // 返回 Set 自身（可链式调用）
s.has("Matt");        // true
s.delete("Matt");     // 删除单个
s.clear();            // 清空
s.size;               // 元素数量
```

### 顺序与迭代

Set 也维护**插入顺序**：

```jsx
const s = new Set(["val1", "val2"]);

for (const value of s.values()) {  // 或 s[Symbol.iterator]()
  console.log(value);
}

// entries() 返回 [value, value] 的键值对（为与 Map 接口统一）
for (const [k, v] of s.entries()) {
  console.log(k === v);  // true
}
```

### 常用场景

```jsx
// 数组去重
const unique = [...new Set([1, 2, 3, 3, 2, 1])]; // [1, 2, 3]

// 集合运算
const a = new Set([1, 2, 3]);
const b = new Set([2, 3, 4]);

// 并集
new Set([...a, ...b]);              // {1, 2, 3, 4}
// 交集
new Set([...a].filter(x => b.has(x)));  // {2, 3}
// 差集
new Set([...a].filter(x => !b.has(x))); // {1}
```

---

## 6.7 WeakSet

`WeakSet` 的值**只能是对象或 Symbol**，且是**弱引用**。

```jsx
const ws = new WeakSet();
const obj = { id: 1 };

ws.add(obj);
ws.has(obj);    // true
ws.delete(obj);
```

<aside>
💡

与 WeakMap 类似，WeakSet **不可迭代**，没有 `size` 和 `clear()`。主要用于标记对象（如检测循环引用、标记已处理的 DOM 节点等）。

</aside>

### 典型用途：标记对象

```jsx
const disabledElements = new WeakSet();
const loginButton = document.querySelector("#login");

// 标记禁用
disabledElements.add(loginButton);

// 检查是否禁用
if (disabledElements.has(loginButton)) {
  // ...
}
// DOM 节点移除后会自动被垃圾回收
```

---

## 本章小结

| **类型** | **特点** | **可迭代** | **适用场景** |
| --- | --- | --- | --- |
| `Object` | 键只能是字符串/Symbol | ✅ | 通用键值存储、JSON |
| `Array` | 有序、可变长、丰富的 API | ✅ | 有序数据集合 |
| `Map` | 键可为任意类型、有序、高性能 | ✅ | 频繁增删键值对 |
| `WeakMap` | 键为弱引用对象 | ❌ | 私有数据、DOM 元数据 |
| `Set` | 值唯一、有序 | ✅ | 去重、集合运算 |
| `WeakSet` | 值为弱引用对象 | ❌ | 标记/追踪对象 |