---
title: 第4章 变量、作用域与内存
date: 2026-02-17 15:01:58
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 4.1 原始值与引用值

ECMAScript 变量可以包含两种不同类型的数据：**原始值**（primitive value）和**引用值**（reference value）。

- **原始值**：最简单的数据，即 `Undefined`、`Null`、`Boolean`、`Number`、`String`、`Symbol`、`BigInt`。按值访问，操作的是存储在变量中的实际值。
- **引用值**：由多个值构成的对象。按引用访问，操作的是对该对象的**引用**而非对象本身。

### 动态属性

- 引用值可以随时添加、修改和删除其属性与方法：

```jsx
let person = new Object();
person.name = "Nicholas";
console.log(person.name); // "Nicholas"
```

- 原始值不能有属性，尽管给原始值添加属性不会报错：

```jsx
let name = "Nicholas";
name.age = 27;
console.log(name.age); // undefined
```

- 使用 `new` 关键字创建的原始值包装类型（如 `new String()`）是 **Object** 实例，可以添加属性。

### 复制值

- **原始值**的复制是**独立副本**，两个变量互不影响：

```jsx
let num1 = 5;
let num2 = num1; // num2 是 num1 的独立副本
num2 = 10;
console.log(num1); // 5（不受影响）
```

- **引用值**的复制是**引用的副本**（指针），两个变量指向同一个对象：

```jsx
let obj1 = new Object();
let obj2 = obj1;
obj1.name = "Nicholas";
console.log(obj2.name); // "Nicholas"（同一个对象）
```

### 传递参数

<aside>
⚠️

ECMAScript 中所有函数的参数都是 **按值传递** 的。

</aside>

- 传递原始值时，函数内部的修改不会影响外部变量。
- 传递引用值时，传递的是**引用的副本**（不是按引用传递）。函数内部可以通过这个引用修改对象属性，但重新赋值参数不会影响外部变量：

```jsx
function setName(obj) {
    obj.name = "Nicholas";
    obj = new Object();    // 重新赋值，不影响外部
    obj.name = "Greg";
}
let person = new Object();
setName(person);
console.log(person.name); // "Nicholas"（而非 "Greg"）
```

### 确定类型

- `typeof`：适合判断**原始值**类型，对对象只能返回 `"object"`。
- `instanceof`：适合判断**引用值**的具体类型：

```jsx
console.log(person instanceof Object);  // true
console.log(colors instanceof Array);   // true
console.log(pattern instanceof RegExp); // true
```

<aside>
💡

所有引用值都是 `Object` 的实例，因此 `引用值 instanceof Object` 始终返回 `true`。对原始值使用 `instanceof` 始终返回 `false`。

</aside>

---

## 4.2 执行上下文与作用域

### 执行上下文（Execution Context）

- 每个执行上下文都有一个关联的 **变量对象**（variable object），该上下文中定义的所有变量和函数都存在于这个对象上。
- **全局上下文** 是最外层的上下文。在浏览器中，全局上下文就是 `window` 对象。
- 每个函数调用都会创建自己的执行上下文。代码执行流进入函数时，函数的上下文被推入**上下文栈**；函数执行完毕后，上下文栈弹出该上下文，将控制权返还给之前的上下文。

### 作用域链（Scope Chain）

上下文中的代码在执行时，会创建一个 **作用域链**，决定了变量和函数的访问顺序：

1. 当前执行上下文的变量对象（最前端）
2. 包含上下文的变量对象
3. …逐级向上
4. 全局上下文的变量对象（最末端）

```jsx
var color = "blue";

function changeColor() {
    let anotherColor = "red";

    function swapColors() {
        let tempColor = anotherColor;
        anotherColor = color;
        color = tempColor;
        // 这里可以访问 color、anotherColor、tempColor
    }

    swapColors();
    // 这里可以访问 color、anotherColor，但不能访问 tempColor
}

changeColor();
// 这里只能访问 color
```

<aside>
🔗

内部上下文可以通过作用域链访问外部上下文中的一切，但外部上下文无法访问内部上下文中的任何东西。上下文之间的连接是**线性的、有序的**。

</aside>

### 作用域链增强

某些语句会在作用域链前端临时添加一个上下文：

- **`try/catch`** 的 `catch` 块：创建一个新的变量对象，包含抛出的错误对象的声明。
- **`with`** 语句：将指定对象添加到作用域链前端。

### 变量声明

### `var` 声明

- **函数作用域**：变量的作用域限定在最近的函数内部（不是块级）。
- **声明提升**（hoisting）：`var` 声明会被提升到函数作用域的顶部。
- 可以重复声明，不会报错。

```jsx
function foo() {
    console.log(age); // undefined（已提升，但未赋值）
    var age = 26;
}
```

### `let` 声明

- **块级作用域**：变量的作用域限定在最近的 `{}` 块内部。
- **暂时性死区**（Temporal Dead Zone, TDZ）：在声明之前不可访问，会抛出 `ReferenceError`。
- 不允许同一作用域内重复声明。
- 在全局作用域中用 `let` 声明的变量**不会**成为 `window` 对象的属性。

```jsx
if (true) {
    let age = 26;
    console.log(age); // 26
}
console.log(age);     // ReferenceError
```

### `const` 声明

- 与 `let` 基本相同，但声明时**必须初始化**，且之后**不能重新赋值**。
- 如果 `const` 变量引用的是对象，对象的属性**可以修改**：

```jsx
const person = {};
person.name = "Nicholas"; // OK，修改对象属性
// person = {};           // TypeError，不能重新赋值
```

- 若要使对象也不可修改，可使用 `Object.freeze()`。

### 三者对比

| 特性 | `var` | `let` | `const` |
| --- | --- | --- | --- |
| 作用域 | 函数作用域 | 块级作用域 | 块级作用域 |
| 声明提升 | ✅ 是 | ❌ 否（TDZ） | ❌ 否（TDZ） |
| 重复声明 | ✅ 允许 | ❌ 不允许 | ❌ 不允许 |
| 重新赋值 | ✅ 允许 | ✅ 允许 | ❌ 不允许 |
| 全局声明为 window 属性 | ✅ 是 | ❌ 否 | ❌ 否 |

<aside>
✅

**最佳实践**：优先使用 `const`，只在确实需要修改变量值时使用 `let`，不再使用 `var`。

</aside>

---

## 4.3 垃圾回收

JavaScript 是使用 **垃圾回收** 的语言，执行环境负责在代码执行时管理内存。基本思路：确定哪个变量不会再使用，然后释放它占用的内存。

### 标记清理（Mark-and-Sweep）

最常用的垃圾回收策略：

1. 垃圾回收程序运行时，会标记内存中存储的所有变量。
2. 将所有在上下文中的变量，以及被上下文中变量引用的变量的标记去掉。
3. 之后仍有标记的变量就是待删除的（因为任何在上下文中的变量都无法访问到它们）。
4. 随后垃圾回收程序进行一次**内存清理**，销毁带标记的值并回收内存。

### 引用计数（Reference Counting）

另一种不太常见的策略：

- 对每个值跟踪它被引用的次数。
- 被引用 +1，引用被覆盖 -1，计数为 0 时回收。
- **致命问题**：**循环引用** 导致内存永远不能释放：

```jsx
function problem() {
    let objA = new Object();
    let objB = new Object();
    objA.someOtherObject = objB;
    objB.anotherObject = objA;
    // 即使函数执行完，引用计数都不会为 0
}
```

### 性能

- 垃圾回收是周期性运行的，调度策略影响性能。
- 现代浏览器采用**动态分配**策略：根据对象分配的大小和数量动态调整垃圾回收的频率。

### 内存管理

<aside>
🧠

由于分配给浏览器的内存通常比桌面应用少得多，良好的内存管理习惯至关重要。

</aside>

### 解除引用

- 将不再需要的全局变量、全局对象属性设置为 `null`（**解除引用**），以便垃圾回收：

```jsx
function createPerson(name) {
    let localPerson = new Object();
    localPerson.name = name;
    return localPerson;
}
let globalPerson = createPerson("Nicholas");
// 使用完毕后
globalPerson = null; // 解除引用，等待垃圾回收
```

### 使用 `const` 和 `let` 提升性能

`const` 和 `let` 是块级作用域，会让垃圾回收器更早介入并回收内存（当块级作用域结束时），比函数作用域的 `var` 更有利。

### 隐藏类与删除操作

V8 引擎利用 **隐藏类**（Hidden Class）优化对象属性访问。保持对象结构一致有助于引擎共享隐藏类：

- ✅ 在构造函数中一次性声明所有属性。
- ❌ 避免动态添加/删除属性（`delete`），否则会导致引擎创建新的隐藏类，性能下降。

```jsx
// 推荐：结构一致
function Article(title) {
    this.title = title;
    this.author = null; // 先用 null 占位
}

// 不推荐：用 delete 删除属性
let a = new Article("foo");
delete a.author; // 导致隐藏类变化
```

### 内存泄漏常见场景

- **意外的全局变量**：函数内未使用声明关键字，变量自动成为全局变量。
- **定时器引用**：`setInterval` 引用外部变量，导致无法回收。
- **闭包**：闭包会保留其外部函数的整个变量对象。

```jsx
// 意外全局变量
function setName() {
    name = "Nicholas"; // 没有声明，成为全局变量 window.name
}

// 定时器引用
let name = "Nicholas";
setInterval(() => {
    console.log(name);
}, 100); // 只要定时器在运行，name 就不会被回收
```

### 静态分配与对象池

为了减少垃圾回收的频率，可以使用 **对象池**（Object Pool）：

- 预先创建一组对象，使用时从池中取出，用完后归还，而不是反复创建和销毁。
- 适用于需要大量频繁创建/销毁相同类型对象的场景（如游戏开发中的向量计算）。

---

## 4.4 小结

<aside>
📌

- JavaScript 变量值分为 **原始值** 和 **引用值**，原始值按值访问，引用值按引用访问。
- 原始值有 7 种类型；引用值是 Object 类型及其子类。
- 所有函数参数都是 **按值传递** 的。
- 确定原始值类型用 `typeof`，确定引用值类型用 `instanceof`。
- 执行上下文分为全局上下文和函数上下文，通过 **作用域链** 逐级查找变量。
- `var` 是函数作用域，`let` 和 `const` 是块级作用域。优先使用 `const`，其次 `let`。
- 垃圾回收主要采用 **标记清理** 策略；关注内存泄漏场景，及时解除不再需要的引用。
</aside>