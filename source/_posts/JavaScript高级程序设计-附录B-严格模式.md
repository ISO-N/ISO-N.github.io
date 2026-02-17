---
title: 附录B 严格模式
date: 2026-02-17 15:02:20
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## B.1 严格模式的历史

- 严格模式（Strict Mode）在 **ES5** 中首次引入
- 目的是让 JavaScript 在更严格的条件下运行，消除一些不安全和不合理的语法行为
- 严格模式可以应用于**整个脚本**或**单个函数**

---

## B.2 选择使用严格模式

### B.2.1 脚本级别

- 在脚本顶部添加编译指示（pragma）：

```jsx
"use strict";

// 整个脚本都在严格模式下运行
```

### B.2.2 函数级别

- 在函数体开头添加编译指示，仅该函数在严格模式下运行：

```jsx
function doSomething() {
  "use strict";
  // 函数体在严格模式下运行
}
```

<aside>
💡

ES6 的**模块**和**类**中的代码默认在严格模式下运行，无需手动声明。

</aside>

---

## B.3 变量

严格模式对变量的创建和使用做了更严格的限制：

- **不允许意外创建全局变量** — 未使用 `var`/`let`/`const` 声明的变量赋值会抛出 `ReferenceError`

```jsx
"use strict";
message = "hello"; // ReferenceError: message is not defined
```

- **不能对变量调用 `delete`** — 删除变量会抛出 `SyntaxError`

```jsx
"use strict";
let color = "red";
delete color; // SyntaxError
```

- **保留字不能用作变量名** — 以下关键字不能用作变量名或函数参数名：
    - `implements`、`interface`、`let`、`package`、`private`、`protected`、`public`、`static`、`yield`

---

## B.4 对象

严格模式下操作对象会更容易抛出错误：

- **给只读属性赋值**会抛出 `TypeError`

```jsx
"use strict";
const obj = {};
Object.defineProperty(obj, 'name', { value: 'Matt', writable: false });
obj.name = 'John'; // TypeError
```

- **给不可配置的属性使用 `delete`** 会抛出 `TypeError`

```jsx
"use strict";
const obj = {};
Object.defineProperty(obj, 'name', { value: 'Matt', configurable: false });
delete obj.name; // TypeError
```

- **给不可扩展的对象添加属性**会抛出 `TypeError`

```jsx
"use strict";
const obj = { name: 'Matt' };
Object.preventExtensions(obj);
obj.age = 27; // TypeError
```

- **对象字面量中属性名重复**在早期严格模式中会报错（ES6 后已放宽此限制）

---

## B.5 函数

### B.5.1 参数相关限制

- **命名参数不能重复** — 重复的参数名会抛出 `SyntaxError`

```jsx
"use strict";
// SyntaxError: Duplicate parameter name not allowed
function sum(a, a, b) {
  return a + b;
}
```

- **`arguments` 对象的行为改变**：
    - `arguments` 不再跟踪命名参数的变化（非严格模式下两者是同步的）
    - 无法对 `arguments` 赋值

```jsx
"use strict";
function showValue(value) {
  value = "Foo";
  console.log(value);          // "Foo"
  console.log(arguments[0]);   // 原始传入的值，不会变成 "Foo"
}
showValue("Bar");
```

### B.5.2 `arguments.callee` 和 `arguments.caller`

- **不能使用 `arguments.callee`** — 访问会抛出 `TypeError`
- **不能使用 `arguments.caller`** — 访问会抛出 `TypeError`

```jsx
"use strict";
function factorial(n) {
  if (n <= 1) return 1;
  return n * arguments.callee(n - 1); // TypeError
}
```

### B.5.3 函数声明的限制

- **函数声明只能在脚本或函数的顶级作用域**中使用
- 在 `if`、`for` 等块中声明函数在严格模式下会抛出 `SyntaxError`

```jsx
"use strict";
// SyntaxError
if (true) {
  function doSomething() {}
}
```

---

## B.6 `eval()`

- **`eval()` 拥有自己独立的作用域** — 在 `eval()` 中声明的变量和函数不会泄漏到外部
- 可以在 `eval()` 中引用外部变量，但内部定义的变量只在 `eval()` 执行期间存在

```jsx
"use strict";
eval("var x = 10;");
console.log(typeof x); // "undefined"（非严格模式下为 "number"）
```

- **不能使用 `eval` 作为标识符**（变量名、函数名等）

```jsx
"use strict";
// 以下都是 SyntaxError
let eval = 10;
```

---

## B.7 `this` 强制转换

- 非严格模式下，`this` 值为 `null` 或 `undefined` 时会被强制转换为全局对象（`window`）
- 严格模式下，`this` 值**不会被强制转换**，保持 `null` 或 `undefined`

```jsx
"use strict";
function getColor() {
  return this.color; // 如果 this 是 undefined，会抛出 TypeError
}

// 非严格模式下，this 指向 window
// 严格模式下，this 是 undefined
getColor(); // TypeError: Cannot read property 'color' of undefined
```

<aside>
⚠️

严格模式下通过 `call()`、`apply()` 或 `bind()` 传入的 `this` 值不会被包装为对象。例如传入原始值 `42`，在非严格模式下会被包装为 `Number(42)`，而在严格模式下保持为原始值 `42`。

</aside>

---

## B.8 其他变化

- **去掉了 `with` 语句** — 使用 `with` 会抛出 `SyntaxError`

```jsx
"use strict";
// SyntaxError
with (Math) {
  console.log(sqrt(4));
}
```

- **八进制字面量的改变** — 不允许使用前导零的八进制字面量

```jsx
"use strict";
let num = 010; // SyntaxError
// 应使用 0o 前缀
let num2 = 0o10; // 正确，值为 8
```

- **`parseInt()` 的八进制解析改变** — 严格模式下 `parseInt()` 不再解析以 `0` 开头的字符串为八进制

```jsx
"use strict";
console.log(parseInt("010")); // 10（非严格模式某些引擎下可能为 8）
```

---

## B.9 小结

| **类别** | **非严格模式行为** | **严格模式行为** |
| --- | --- | --- |
| 未声明变量赋值 | 创建全局变量 | 抛出 `ReferenceError` |
| `this` 为 `undefined` | 强制转换为全局对象 | 保持 `undefined` |
| 重复参数名 | 允许 | 抛出 `SyntaxError` |
| `arguments` 同步 | 与命名参数同步 | 不同步 |
| `eval()` 作用域 | 变量泄漏到外部 | 独立作用域 |
| `with` 语句 | 允许 | 抛出 `SyntaxError` |
| 八进制字面量 `010` | 解析为 8 | 抛出 `SyntaxError` |
| 只读属性赋值 | 静默失败 | 抛出 `TypeError` |
| `delete` 变量 | 静默失败 | 抛出 `SyntaxError` |
| `arguments.callee` | 允许 | 抛出 `TypeError` |