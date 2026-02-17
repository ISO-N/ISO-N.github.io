---
title: 第3章 语言基础
date: 2026-02-17 15:01:57
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 3.1 语法

### 区分大小写

ECMAScript 中一切都**区分大小写**——变量、函数名、操作符皆如此。`typeof` 是关键字，不能用作函数名，但 `Typeof`、`typeOf` 是合法标识符。

### 标识符

- 第一个字符必须是：字母、下划线 `_` 或美元符 `$`
- 其余字符可以是字母、下划线、美元符或**数字**
- 按惯例使用**小驼峰**（camelCase），如 `doSomething`、`myVariable`

### 注释

```jsx
// 单行注释

/* 这是
   多行注释 */
```

### 严格模式

ES5 引入了**严格模式**（strict mode），可在脚本或函数顶部添加：

```jsx
"use strict";
```

严格模式会改变一些不安全/不确定的行为，将其转为抛出错误。

### 语句

- 语句以**分号**结尾（推荐始终加分号）
- 多条语句可以用 `{ }` 组合成**代码块**
- `if` 等控制语句即使只有一行，也推荐用代码块包裹

---

## 3.2 关键字与保留字

<aside>
⚠️

关键字和保留字**不能**用作标识符或属性名。

</aside>

**关键字**（部分）：`break`、`case`、`catch`、`continue`、`debugger`、`default`、`delete`、`do`、`else`、`finally`、`for`、`function`、`if`、`in`、`instanceof`、`new`、`return`、`switch`、`this`、`throw`、`try`、`typeof`、`var`、`void`、`while`、`with`

**ES6 新增关键字**：`let`、`const`、`class`、`export`、`import`、`super`、`yield` 等

---

## 3.3 变量

### var 声明

- **函数作用域**，不是块作用域
- 存在**变量提升**（hoisting）：声明会被提升到函数顶部

```jsx
function foo() {
  console.log(age); // undefined（不会报错，因为提升了）
  var age = 26;
}
```

- 可以重复声明同一变量
- 在全局作用域中用 `var` 声明的变量会成为 `window` 的属性

### let 声明

- **块作用域**（`{ }` 内有效）
- **没有变量提升**（严格来说存在"暂时性死区" TDZ）
- 同一块中**不能重复声明**
- 全局声明时**不会**成为 `window` 属性

```jsx
if (true) {
  let age = 26;
  console.log(age); // 26
}
console.log(age); // ReferenceError
```

<aside>
💡

经典面试题：`for` 循环中 `let` vs `var`

</aside>

```jsx
// var —— 共享同一个变量
for (var i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 0);
}
// 输出: 5, 5, 5, 5, 5

// let —— 每次迭代创建新的绑定
for (let i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), 0);
}
// 输出: 0, 1, 2, 3, 4
```

### const 声明

- 与 `let` 基本相同，但声明时**必须初始化**，且**不能重新赋值**
- 如果值是对象，对象的**属性可以修改**（const 限制的是绑定，不是值本身）

```jsx
const person = { name: "Matt" };
person.name = "Nicholas"; // ✅ 允许
person = {};              // ❌ TypeError
```

<aside>
✅

**最佳实践**：优先使用 `const`，只在确实需要修改变量时使用 `let`，**不再使用 `var`**。

</aside>

---

## 3.4 数据类型

ECMAScript 有 **7 种原始类型** + **1 种复杂类型**：

| **分类** | **类型** | **说明** |
| --- | --- | --- |
| 原始类型 | `Undefined` | 声明但未赋值 |
| 原始类型 | `Null` | 空对象指针 |
| 原始类型 | `Boolean` | `true` / `false` |
| 原始类型 | `Number` | 整数和浮点数（双精度 IEEE 754） |
| 原始类型 | `String` | 不可变的字符序列 |
| 原始类型 | `Symbol` | ES6 新增，唯一且不可变的标识符 |
| 原始类型 | `BigInt` | 任意精度整数 |
| 复杂类型 | `Object` | 无序键值对集合（所有引用类型的基类） |

### typeof 操作符

```jsx
typeof undefined   // "undefined"
typeof true        // "boolean"
typeof "hello"     // "string"
typeof 42          // "number"
typeof Symbol()    // "symbol"
typeof null        // "object"  ⚠️ 历史遗留 Bug
typeof {}          // "object"
typeof function(){} // "function"
```

### Undefined 类型

- 只有一个值：`undefined`
- 变量声明未赋值默认为 `undefined`
- **未声明的变量**只能执行 `typeof`（返回 `"undefined"`），其余操作会报 `ReferenceError`

### Null 类型

- 只有一个值：`null`
- `null == undefined` 为 `true`（宽松相等）
- 当变量将来要保存对象时，建议初始化为 `null`

### Boolean 类型

各类型转换为 Boolean 的规则：

| **数据类型** | **转为 true** | **转为 false** |
| --- | --- | --- |
| String | 非空字符串 | `""`（空字符串） |
| Number | 非零数值（包括 Infinity） | `0`、`NaN` |
| Object | 任何对象 | `null` |
| Undefined | — | `undefined` |

### Number 类型

**整数**：十进制、八进制（`0o70`）、十六进制（`0xFF`）

**浮点数**：

- 存储空间是整数的两倍，JS 引擎会尽量转为整数
- 经典精度问题：`0.1 + 0.2 !== 0.3`（IEEE 754 通病）

**特殊值**：

- `Number.MIN_VALUE` / `Number.MAX_VALUE`
- `Infinity` / `-Infinity`
- **`NaN`**：不等于任何值，包括它自身；用 `isNaN()` 或 `Number.isNaN()` 检测

**数值转换**：

| **函数** | **特点** |
| --- | --- |
| `Number()` | 通用转换；`""` → `0`，`null` → `0`，`undefined` → `NaN` |
| `parseInt()` | 解析整数；忽略前导空格，遇到非数字停止；**建议始终传第二个参数（基数）** |
| `parseFloat()` | 解析浮点数；只解析十进制 |

### String 类型

- 字符串是**不可变**的（immutable）
- 转换方式：`toString()` 方法、`String()` 函数、`"" + value`

**模板字面量**（Template Literals）：

```jsx
let name = "World";
let greeting = `Hello, ${name}!`; // 支持插值
let multiLine = `第一行
第二行`; // 支持多行
```

**标签函数**（Tagged Templates）：

```jsx
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) =>
    `${result}${str}<b>${values[i] || ""}</b>`, "");
}
let x = 10, y = 20;
highlight`${x} + ${y} = ${x + y}`;
```

**原始字符串**：`String.raw` 可以获取转义前的原始内容：

```jsx
String.raw`\n`  // "\\n"（长度为2）
```

### Symbol 类型

- 用 `Symbol()` 创建，每次调用都产生**唯一**的符号
- 可以传入描述字符串：`Symbol("desc")`，仅用于调试
- **不能** 与 `new` 一起使用

```jsx
let s1 = Symbol("foo");
let s2 = Symbol("foo");
s1 === s2; // false
```

**全局符号注册表**：`Symbol.for()` 和 `Symbol.keyFor()`

```jsx
let s1 = Symbol.for("app.id"); // 创建/复用全局符号
let s2 = Symbol.for("app.id");
s1 === s2; // true
Symbol.keyFor(s1); // "app.id"
```

**常用内置符号**（Well-Known Symbols）：

- `Symbol.iterator` — 定义默认迭代器
- `Symbol.toPrimitive` — 对象转原始值
- `Symbol.hasInstance` — `instanceof` 行为
- `Symbol.toStringTag` — `Object.prototype.toString` 的标签

---

## 3.5 操作符

### 一元操作符

- **递增/递减**：`++` / `--`（前缀先运算后取值，后缀先取值后运算）
- **一元加/减**：`+value` 相当于 `Number(value)`；`-value` 取负

### 位操作符

- 按 **32 位整数** 操作
- `~`（按位非）、`&`（按位与）、`|`（按位或）、`^`（按位异或）
- `<<`（左移）、`>>`（有符号右移）、`>>>`（无符号右移）

### 布尔操作符

- `!`（逻辑非）：`!!value` 等价于 `Boolean(value)`
- `&&`（逻辑与）：**短路求值**，第一个为假直接返回，不再计算第二个
- `||`（逻辑或）：**短路求值**，第一个为真直接返回

### 乘性操作符

`*`、`/`、`%` — 对非数值会先调用 `Number()` 转换

### 指数操作符

```jsx
2 ** 10  // 1024（等价于 Math.pow(2, 10)）
```

### 加性操作符

- `+`：如果有一个操作数是**字符串**，则另一个也转为字符串并**拼接**
- `-`：将操作数转为数值后相减

<aside>
⚠️

常见陷阱：`"5" + 3` → `"53"`（拼接），`"5" - 3` → `2`（数值运算）

</aside>

### 关系操作符

`<`、`>`、`<=`、`>=`

- 两个字符串比较时按**字符编码**逐位比较
- 任一操作数是数值，则另一个转为数值
- 与 `NaN` 比较总返回 `false`

### 相等操作符

| **操作符** | **名称** | **特点** |
| --- | --- | --- |
| `==` / `!=` | 相等 / 不相等 | 会**类型转换**后再比较 |
| `===` / `!==` | 全等 / 不全等 | **不转换**类型，类型不同直接 `false` |

<aside>
✅

**最佳实践**：始终使用 `===` 和 `!==`，避免隐式类型转换带来的意外。

</aside>

### 条件操作符（三元）

```jsx
let max = (a > b) ? a : b;
```

### 赋值操作符

`=`、`+=`、`-=`、`*=`、`/=`、`%=`、`**=`、`<<=`、`>>=`、`>>>=`

### 逗号操作符

```jsx
let num = (1, 2, 3, 4, 5); // num 为 5（取最后一个值）
```

---

## 3.6 语句

### if 语句

```jsx
if (condition) {
  // ...
} else if (anotherCondition) {
  // ...
} else {
  // ...
}
```

### do-while 语句

**后测试循环**（至少执行一次）：

```jsx
do {
  // ...
} while (condition);
```

### while 语句

**先测试循环**：

```jsx
while (condition) {
  // ...
}
```

### for 语句

```jsx
for (let i = 0; i < 10; i++) {
  // ...
}
```

初始化、条件、循环后表达式都**可省略**（无限循环：`for(;;)`）。

### for-in 语句

遍历对象的**可枚举属性**（包括原型链上的）：

```jsx
for (const key in object) {
  console.log(key, object[key]);
}
```

- 不保证属性的顺序
- 若对象为 `null` 或 `undefined`，不执行循环体

### for-of 语句

遍历**可迭代对象**（数组、Map、Set、字符串等）：

```jsx
for (const item of [1, 2, 3]) {
  console.log(item);
}
```

### 标签语句

```jsx
outerLoop:
for (let i = 0; i < 10; i++) {
  for (let j = 0; j < 10; j++) {
    if (j === 5) break outerLoop; // 跳出外层循环
  }
}
```

### break 与 continue

- `break` — 立即退出循环
- `continue` — 跳过本次迭代，进入下一次

### switch 语句

```jsx
switch (expression) {
  case value1:
    // ...
    break;
  case value2:
    // ...
    break;
  default:
    // ...
}
```

- 使用 **`===`** 比较，不会类型转换
- 没有 `break` 会**贯穿**（fall-through）到下一个 case

---

## 3.7 函数

使用 `function` 关键字声明：

```jsx
function sayHi(name, message) {
  console.log(`Hello ${name}, ${message}`);
}
```

- 不需要指定返回值类型；`return` 可以返回任意值
- 没有 `return` 或 `return;` 时，返回 `undefined`
- **参数** 按值传递，内部通过 `arguments` 类数组对象可访问所有传入的参数
- JS 函数**没有重载**——同名函数后定义的会覆盖先定义的

<aside>
✅

**小结**：本章是 JavaScript 的语法基石。重点掌握 `let`/`const` 的块作用域与 `var` 的区别、7 种原始类型的转换规则、`===` 全等比较，以及 `for-of` 与 `for-in` 的适用场景。这些基础会贯穿后续所有章节。

</aside>