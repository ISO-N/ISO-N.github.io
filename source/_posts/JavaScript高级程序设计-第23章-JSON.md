---
title: 第23章 JSON
date: 2026-02-17 15:02:14
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 23.1 语法

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，由 Douglas Crockford 发明。JSON 语法支持三种类型的值：

### 简单值

JSON 可以表示字符串、数值、布尔值和 `null`，但 **不支持** `undefined`。

- 字符串**必须使用双引号**（单引号会导致语法错误）
- 数值无需引号
- 布尔值和 `null` 直接书写

```json
"Hello World"
42
true
null
```

### 对象

JSON 对象与 JavaScript 对象字面量有两个重要区别：

1. **属性名必须加双引号**
2. **没有变量声明**（JSON 中没有变量的概念）
3. **末尾没有分号**

```json
{
  "name": "Nicholas",
  "age": 29,
  "school": {
    "name": "Merrimack College",
    "location": "North Andover, MA"
  }
}
```

### 数组

JSON 数组使用 JavaScript 的数组字面量形式，同样**没有变量和分号**：

```json
[
  {
    "title": "Professional JavaScript",
    "authors": ["Nicholas C. Zakas"],
    "edition": 4,
    "year": 2020
  },
  {
    "title": "Professional Ajax",
    "authors": ["Nicholas C. Zakas", "Jeremy McPeak"],
    "edition": 2,
    "year": 2008
  }
]
```

---

## 23.2 解析与序列化

JSON 的流行并非因为语法与 JavaScript 类似，而是因为 **可以直接解析为可用的 JavaScript 对象**，相比 XML 需要 DOM 解析要方便得多。

### JSON 对象

ECMAScript 5 定义了全局对象 `JSON`，所有主流浏览器都支持。JSON 对象有两个方法：

- `JSON.stringify()`：将 JavaScript 对象序列化为 JSON 字符串
- `JSON.parse()`：将 JSON 字符串解析为原生 JavaScript 值

```jsx
let book = {
  title: "Professional JavaScript",
  authors: ["Nicholas C. Zakas"],
  edition: 4,
  year: 2020
};

let jsonText = JSON.stringify(book);
// '{"title":"Professional JavaScript","authors":["Nicholas C. Zakas"],"edition":4,"year":2020}'
```

<aside>
⚠️

**注意**：在序列化 JavaScript 对象时，所有**函数**和**原型成员**都会被有意省略。值为 `undefined` 的属性也会被跳过。最终结果中只保留有效的 JSON 数据类型的实例属性。

</aside>

---

## 23.3 序列化选项

`JSON.stringify()` 除了接收要序列化的对象外，还可以接收两个额外参数：

```jsx
JSON.stringify(value, replacer, space)
```

### 过滤结果（replacer）

**第二个参数**可以是**数组**或**函数**。

**数组过滤器**——只包含数组中列出的属性：

```jsx
let jsonText = JSON.stringify(book, ["title", "edition"]);
// '{"title":"Professional JavaScript","edition":4}'
```

**替换函数**——对每个键值对进行更精细的控制：

```jsx
let jsonText = JSON.stringify(book, (key, value) => {
  switch (key) {
    case "authors":
      return value.join(",");
    case "year":
      return 5000;
    case "edition":
      return undefined; // 返回 undefined 会删除该属性
    default:
      return value;
  }
});
// '{"title":"Professional JavaScript","authors":"Nicholas C. Zakas","year":5000}'
```

<aside>
💡

替换函数第一次调用时，`key` 是**空字符串**，`value` 是要序列化的整个对象。务必在 `default` 分支中返回 `value`，否则所有未显式处理的属性都会被忽略。

</aside>

### 字符串缩进（space）

**第三个参数**控制缩进和空白符，用于美化输出。

**数值缩进**（最大值为 10）：

```jsx
let jsonText = JSON.stringify(book, null, 4);
/*
{
    "title": "Professional JavaScript",
    "authors": [
        "Nicholas C. Zakas"
    ],
    "edition": 4,
    "year": 2020
}
*/
```

**字符串缩进**（最长取前 10 个字符）：

```jsx
let jsonText = JSON.stringify(book, null, "--");
/*
{
--"title": "Professional JavaScript",
--"authors": [
----"Nicholas C. Zakas"
--],
--"edition": 4,
--"year": 2020
}
*/
```

### toJSON() 方法

可以在对象上自定义 `toJSON()` 方法来定义其 JSON 序列化行为：

```jsx
let book = {
  title: "Professional JavaScript",
  authors: ["Nicholas C. Zakas"],
  edition: 4,
  year: 2020,
  toJSON: function () {
    return this.title;
  }
};

let jsonText = JSON.stringify(book);
// '"Professional JavaScript"'
```

<aside>
📌

**序列化对象的顺序**（非常重要）：

1. 如果对象有 `toJSON()` 方法且能取到有效值，则调用该方法，否则使用默认序列化
2. 如果提供了第二个参数（替换函数），则应用它（传入的值是第 1 步返回的值）
3. 对第 2 步返回的每个值进行相应的序列化
4. 如果提供了第三个参数，执行相应的格式化
</aside>

---

## 23.4 解析选项

`JSON.parse()` 也可以接收一个额外参数——**还原函数（reviver）**，其签名与 `JSON.stringify()` 的替换函数完全相同：

```jsx
JSON.parse(jsonText, reviver)
```

### 还原函数

还原函数接收 `key` 和 `value` 两个参数，需要返回一个值。返回 `undefined` 会从结果中删除对应的键。

**经典用例**——将日期字符串还原为 `Date` 对象：

```jsx
let book = {
  title: "Professional JavaScript",
  authors: ["Nicholas C. Zakas"],
  edition: 4,
  year: 2020,
  releaseDate: new Date(2019, 11, 1)
};

let jsonText = JSON.stringify(book);
// releaseDate 被序列化为 "2019-12-01T00:00:00.000Z"

let bookCopy = JSON.parse(jsonText, (key, value) => {
  if (key === "releaseDate") {
    return new Date(value);
  }
  return value;
});

console.log(bookCopy.releaseDate.getFullYear()); // 2019
```

<aside>
💡

如果不使用还原函数，`releaseDate` 只是一个普通字符串而非 `Date` 对象，无法调用 `getFullYear()` 等方法。

</aside>

---

## 23.5 小结

| **要点** | **说明** |
| --- | --- |
| JSON 数据类型 | 简单值（字符串、数值、布尔值、`null`）、对象、数组；不支持 `undefined`、函数、Symbol |
| 字符串规范 | **必须使用双引号**，对象的属性名也必须加双引号 |
| `JSON.stringify()` | 将 JS 值序列化为 JSON 字符串；忽略函数、原型成员和值为 `undefined` 的属性 |
| `JSON.parse()` | 将 JSON 字符串解析为 JS 值 |
| replacer（替换函数/数组） | `stringify` 的第二个参数，控制哪些属性被序列化及如何序列化 |
| space（缩进） | `stringify` 的第三个参数，控制输出格式，数值或字符串，最多 10 |
| `toJSON()` | 自定义对象的序列化行为，优先级最高 |
| reviver（还原函数） | `parse` 的第二个参数，常用于将日期字符串还原为 `Date` 对象 |