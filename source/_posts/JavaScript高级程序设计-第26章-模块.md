---
title: 第26章 模块
date: 2026-02-17 15:02:17
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 26.1 理解模块模式

模块模式的核心思想是将逻辑分块，各自封装，相互独立，每个块自行决定对外暴露什么，同时自行决定引入执行哪些外部代码。

### 模块标识符

模块系统本质上是**键/值**实体，每个模块都有一个可用于引用它的标识符。原生实现中，标识符通常是**文件的实际路径**（如 Node.js）或**相对于文件的路径**（如浏览器中的 URL）。

### 模块依赖

模块系统的核心是**依赖管理**。指定依赖的模块会在本地模块执行前先加载好。

### 模块加载

模块加载取决于运行环境：

- **浏览器**：通过 `<script>` 标签加载，或者使用网络请求动态加载
- **Node.js**：通过文件系统加载

### 入口

程序需要一个**入口（entry point）**，即代码执行的起点。依赖图是通过入口模块的依赖递归解析形成的。

### 异步依赖

因为 JavaScript 可以异步执行，所以模块也可以异步加载。可以按需加载模块，让程序在提供模块之后再把这些模块加载进来。

### 动态依赖

有些模块系统允许在程序执行时才确定依赖关系：

```jsx
if (condition) {
  const moduleA = require('./moduleA');
}
```

### 静态分析

模块中包含的信息可以在**不实际执行模块**的情况下被推断出来。静态分析可以做到**摇树优化（tree-shaking）**——从打包结果中移除未引用的代码。

### 循环依赖

模块之间**可能存在循环依赖**，例如 A 依赖 B，B 依赖 A。大多数模块系统能处理循环依赖，但需要注意可能导致的**部分初始化**问题。

---

## 26.2 凑合的模块系统

在 ES6 原生模块规范出现之前，开发者使用各种方式实现模块化：

### IIFE（立即调用函数表达式）

```jsx
var Foo = (function() {
  // 私有变量
  var bar = 'baz';

  return {
    // 公有接口
    getBar: function() {
      return bar;
    }
  };
})();
```

### 揭示模块模式

```jsx
var Foo = (function() {
  var bar = 'baz';
  var qux = function() {
    console.log(bar);
  };

  return {
    qux: qux
  };
})();
```

> 💡 这些模式的不足之处在于：需要**手动管理依赖顺序**，且无法做到按需加载。
> 

---

## 26.3 使用预 ES6 模块加载器

### 26.3.1 CommonJS

CommonJS 主要用于 **Node.js**，使用同步方式加载模块。

**导出：**

```jsx
// moduleA.js
module.exports = {
  stuff: 'hello'
};

// 或者单个导出
module.exports.stuff = 'hello';
```

**导入：**

```jsx
var moduleA = require('./moduleA');
console.log(moduleA.stuff); // 'hello'
```

<aside>
⚠️

CommonJS 以**同步方式**加载模块，即 `require()` 调用时模块会被立即加载并执行。这在服务器端没问题，但不适合浏览器环境。

</aside>

**核心特点：**

- 模块是**单例**：无论 `require()` 多少次，实际只加载一次
- 模块加载后会被**缓存**，后续引用返回缓存的模块
- 导出的是值的**拷贝/引用**（基本类型是拷贝，对象是引用）
- 支持动态依赖，`require()` 可以出现在条件语句中

### 26.3.2 异步模块定义（AMD）

AMD（Asynchronous Module Definition）专为**浏览器**环境设计，支持异步加载。RequireJS 是其代表实现。

```jsx
// 定义模块，ID 为 'moduleA'，依赖 'moduleB'
define('moduleA', ['moduleB'], function(moduleB) {
  return {
    stuff: moduleB.doStuff()
  };
});
```

**核心特点：**

- 异步加载，不会阻塞浏览器
- 可以指定模块 ID（字符串标识符）
- 函数参数与依赖数组按序对应

### 26.3.3 通用模块定义（UMD）

UMD 是一种兼容多种模块系统的**统一模式**，可以让同一份代码在 CommonJS、AMD 和全局环境中都能使用。

```jsx
(function(root, factory) {
  if (typeof define === 'function' && define.amd) {
    // AMD
    define(['moduleB'], factory);
  } else if (typeof module === 'object' && module.exports) {
    // CommonJS / Node
    module.exports = factory(require('moduleB'));
  } else {
    // 浏览器全局变量
    root.returnExports = factory(root.moduleB);
  }
}(this, function(moduleB) {
  return {
    stuff: moduleB.doStuff()
  };
}));
```

> 本质上就是运行时检测当前环境，然后选择对应的方式注册模块。
> 

---

## 26.4 使用 ES6 模块

ES6 模块是 JavaScript 原生支持的模块系统，是未来的标准。

### 26.4.1 模块标签

在浏览器中，需要通过 `type="module"` 来告诉浏览器这是一个模块脚本：

```html
<script type="module" src="module.js"></script>

<!-- 也支持内联模块 -->
<script type="module">
  import { foo } from './fooModule.js';
  console.log(foo);
</script>
```

**与普通脚本的区别：**

- 模块脚本默认是 **`defer`** 的，即延迟执行
- 模块代码运行在**严格模式**下
- 模块只会被**加载一次**（即使被多个 `<script>` 标签引用）
- 模块有自己的**顶级作用域**（不是全局作用域）
- 模块中的 `this` 是 `undefined`（而非 `window`）

### 26.4.2 模块加载

ES6 模块按照**静态分析**加载依赖，加载过程：

1. 浏览器解析入口模块，确定依赖
2. 递归解析依赖的依赖
3. 所有依赖解析完成后，按照**依赖图的逆序**执行模块代码

<aside>
📌

ES6 模块的加载和链接是**异步**的，但模块内部的代码执行是**同步且有序**的。

</aside>

### 26.4.3 模块导出

使用 `export` 关键字导出模块成员。

**命名导出（Named Export）：**

```jsx
// 行内导出
export const foo = 'foo';
export function bar() {}
export class Baz {}

// 集中导出
const foo = 'foo';
function bar() {}
const baz = 'baz';
export { foo, bar, baz };

// 导出时重命名
export { foo as myFoo, bar };
```

**默认导出（Default Export）：**

```jsx
// 每个模块只能有一个默认导出
export default function() {
  console.log('default');
}

// 等价写法
const foo = 'foo';
export { foo as default };
```

**同时使用命名导出和默认导出：**

```jsx
export const bar = 'bar';
export default function() {
  console.log('default');
}
```

<aside>
⚠️

`export` 语句必须在**模块顶层**，不能嵌套在块（如 `if`、函数）中。

</aside>

### 26.4.4 模块导入

使用 `import` 关键字导入模块成员。

**导入命名导出：**

```jsx
// 导入单个
import { foo } from './module.js';

// 导入多个
import { foo, bar, baz } from './module.js';

// 导入时重命名
import { foo as myFoo } from './module.js';

// 导入整个模块的命名空间
import * as Foo from './module.js';
console.log(Foo.foo); // 访问命名导出
console.log(Foo.default); // 访问默认导出
```

**导入默认导出：**

```jsx
import myDefault from './module.js';

// 同时导入默认导出和命名导出
import myDefault, { foo, bar } from './module.js';
import { default as myDefault, foo, bar } from './module.js';
```

**仅执行模块（不导入任何绑定）：**

```jsx
import './module.js';
```

<aside>
📌

`import` 语句同样必须在**模块顶层**。导入的绑定类似于 `const` 声明——不可重新赋值（但如果导入的是对象，可以修改其属性）。

</aside>

### 26.4.5 模块转移导出

模块可以将其他模块的导出"转交"出去，常用于聚合模块：

```jsx
// 转移所有命名导出
export * from './module.js';

// 转移指定的命名导出
export { foo, bar } from './module.js';

// 转移时重命名
export { foo as myFoo } from './module.js';

// 转移默认导出
export { default } from './module.js';

// 将命名导出转为默认导出
export { foo as default } from './module.js';
```

> 注意：`export * from` **不会**转移默认导出，需要显式处理。
> 

### 26.4.6 工作者模块

Worker 也支持模块语法：

```jsx
// 创建模块类型的 Worker
const worker = new Worker('worker.js', { type: 'module' });
```

### 26.4.7 向后兼容

可以使用 `nomodule` 属性提供回退脚本，让不支持模块的浏览器加载传统脚本：

```html
<!-- 支持模块的浏览器会执行这个 -->
<script type="module" src="module.js"></script>

<!-- 不支持模块的浏览器会执行这个 -->
<script nomodule src="fallback.js"></script>
```

---

## 小结

| **特性** | **CommonJS** | **AMD** | **ES6 模块** |
| --- | --- | --- | --- |
| 加载方式 | 同步 | 异步 | 异步（静态解析） |
| 运行环境 | 服务器端（Node.js） | 浏览器 | 浏览器 + Node.js |
| 语法 | `require()` / `module.exports` | `define()` / `require()` | `import` / `export` |
| 静态分析 | ❌ 不支持 | ❌ 不支持 | ✅ 支持（可 tree-shaking） |
| 动态导入 | ✅ `require()` 可在任意位置 | ✅ 通过回调 | ✅ `import()` 动态导入 |
| 循环依赖处理 | 返回部分初始化的导出 | 通过回调延迟 | 通过实时绑定（live binding） |
- **模块模式**的核心是通过封装实现逻辑隔离和依赖管理
- **CommonJS** 是 Node.js 的模块系统，同步加载，适合服务器端
- **AMD** 为浏览器设计，异步加载，RequireJS 是典型实现
- **UMD** 是一种兼容方案，让代码在多种环境中运行
- **ES6 模块**是语言原生支持的模块系统，支持静态分析、摇树优化，是未来标准