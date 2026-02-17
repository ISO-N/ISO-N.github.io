---
title: 第21章 错误处理与调试
date: 2026-02-17 15:02:13
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
本章内容涵盖**浏览器错误报告**、**错误处理机制**、**调试技术**以及**常见错误类型**，是编写健壮 JavaScript 应用的核心知识。

---

## 21.1 浏览器错误报告

### 桌面控制台

所有主流浏览器都通过**开发者工具**的控制台报告 JavaScript 错误，包括：

- 错误消息（message）
- 发生错误的 URL
- 行号（line number）

> 开发时应**始终打开控制台**，以便第一时间发现运行时错误。
> 

### 移动端控制台

- iOS Safari：**设置 → Safari → 高级 → Web 检查器**，然后通过 Mac Safari 的"开发"菜单远程调试。
- Android Chrome：通过 `chrome://inspect` 在桌面 Chrome 进行远程调试。

---

## 21.2 错误处理

### 21.2.1 try/catch 语句

`try/catch` 是 JavaScript 中处理错误的**基本机制**：

```jsx
try {
  // 可能出错的代码
  someFunction();
} catch (error) {
  // 错误处理代码
  console.log(error.message);
}
```

**catch 块中的错误对象**至少包含一个 `message` 属性，此外不同浏览器还可能提供：

- `name`：错误类型名称
- `stack`：栈追踪信息（非标准，但所有主流浏览器都支持）

### 21.2.2 finally 子句

`finally` 子句**无论是否发生错误都会执行**，即使 `try` 或 `catch` 中包含 `return` 语句：

```jsx
function testFinally() {
  try {
    return 1;
  } catch (error) {
    return 2;
  } finally {
    return 0; // 始终返回 0，覆盖前面的 return
  }
}
```

<aside>
⚠️

如果 `finally` 中包含 `return`，会**覆盖** `try` 和 `catch` 中的 `return`。因此应避免在 `finally` 中使用 `return`。

</aside>

### 21.2.3 错误类型

ECMA-262 定义了以下 **8 种内置错误类型**：

| **错误类型** | **说明** |
| --- | --- |
| `Error` | 基类型，其他错误类型都继承自它，一般不会直接抛出 |
| `InternalError` | 浏览器底层引擎抛出的错误，如递归过深导致栈溢出 |
| `EvalError` | 在非直接调用 `eval()` 时抛出（极少见） |
| `RangeError` | 数值超出有效范围时抛出，如 `new Array(-1)` |
| `ReferenceError` | 访问不存在的变量时抛出 |
| `SyntaxError` | 传给 `eval()` 的代码有语法错误时抛出 |
| `TypeError` | 变量不是预期类型时抛出，**最常见**的错误类型 |
| `URIError` | 使用 `encodeURI()` 或 `decodeURI()` 时 URI 格式错误 |

可通过 `instanceof` 判断具体的错误类型：

```jsx
try {
  someFunction();
} catch (error) {
  if (error instanceof TypeError) {
    // 处理类型错误
  } else if (error instanceof RangeError) {
    // 处理范围错误
  } else {
    // 处理其他错误
  }
}
```

### 21.2.4 try/catch 的用法

<aside>
✅

**适合使用 try/catch 的场景**：处理你**无法控制**的错误，例如调用第三方库或浏览器原生 API 时可能抛出的异常。

</aside>

<aside>
❌

**不适合使用 try/catch 的场景**：已知某段代码会出错时，应该在代码层面**提前检查和预防**，而不是用 try/catch 来捕获。

</aside>

### 21.2.5 抛出错误（throw）

使用 `throw` 操作符可以随时抛出自定义错误：

```jsx
throw new Error("Something bad happened.");
throw new TypeError("Expected a string.");
throw new RangeError("Value must be between 1 and 10.");
```

#### 自定义错误类型

可以通过继承 `Error` 来创建自定义错误，便于区分不同的错误来源：

```jsx
class CustomError extends Error {
  constructor(message) {
    super(message);
    this.name = "CustomError";
  }
}

throw new CustomError("My custom error message");
```

#### 何时抛出 vs 何时捕获？

- **抛出错误**：在你能明确知道错误原因的**底层函数**中，抛出带有详细信息的错误。
- **捕获错误**：在**调用方**或应用的入口处捕获并处理错误。

> 关键原则：**抛出错误的目的是提供错误发生原因的具体信息**，而不只是让程序不崩溃。
> 

### 21.2.6 error 事件

任何未被 `try/catch` 处理的错误都会触发 `window` 对象的 `error` 事件：

```jsx
window.onerror = function (message, url, line) {
  console.log(`Error: ${message}\nURL: ${url}\nLine: ${line}`);
  return false; // 返回 false 不阻止默认行为, 返回 true 则阻止
};
```

<aside>
💡

图片加载失败也会触发 `error` 事件，但这个事件在**元素级别**触发，不会冒泡到 `window`：

</aside>

```jsx
const img = new Image();
img.addEventListener("error", (event) => {
  console.log("Image failed to load!");
});
img.src = "doesnotexist.png";
```

### 21.2.7 错误处理策略

对于大型 Web 应用，需要一套**系统化的错误处理策略**，核心思路是：

1. **识别可能发生错误的地方**
2. 对这些地方进行适当的错误处理
3. **记录和上报**错误，以便后续修复

### 21.2.8 识别可能的错误

需要重点关注以下容易出错的模式：

#### 1. 类型转换错误

使用 `==` / `!=` 时会发生**隐式类型转换**，应尽量使用 `===` / `!==`：

```jsx
// ❌ 隐式转换，容易出错
if (value == 0) { ... }

// ✅ 严格比较
if (value === 0) { ... }
```

在流程控制语句中，非布尔值会被自动转换，应**显式判断**：

```jsx
// ❌ 不明确
function process(values) {
  if (values) {  // 非 null/undefined 但也可能不是数组
    values.sort();
  }
}

// ✅ 明确检查类型
function process(values) {
  if (values instanceof Array) {
    values.sort();
  }
}
```

#### 2. 数据类型错误

在函数中应当**验证参数的类型**，尤其是在非类型安全的 JavaScript 中：

```jsx
function reverseSort(values) {
  if (values instanceof Array) {
    values.sort();
    values.reverse();
  }
}
```

- 对于**基本类型**，使用 `typeof` 检查
- 对于**对象类型**，使用 `instanceof` 检查

#### 3. 通信错误

与服务器通信时常见的错误包括：

- URL 格式不正确或参数**未编码**（应使用 `encodeURIComponent()`）
- 响应数据格式不符合预期
- 网络中断导致请求失败

```jsx
// ✅ 对查询参数进行编码
function addQueryParam(url, name, value) {
  url += (url.indexOf("?") === -1 ? "?" : "&");
  url += `${encodeURIComponent(name)}=${encodeURIComponent(value)}`;
  return url;
}
```

### 21.2.9 区分致命错误与非致命错误

> 非致命错误可以用**局部提示**的方式通知用户；致命错误则需要**明确告知**并引导用户刷新或重试。
> 

### 21.2.10 把错误记录到服务器

建立集中的**错误日志系统**，将客户端错误发送到服务器：

```jsx
function logError(severity, message) {
  const img = new Image();
  const encodedSev = encodeURIComponent(severity);
  const encodedMsg = encodeURIComponent(message);
  img.src = `log.php?sev=${encodedSev}&msg=${encodedMsg}`;
}
```

<aside>
💡

使用 `Image` 对象发送日志的好处：

- **所有浏览器**都支持 `Image` 对象
- 可以**跨域**发送请求
- 不受页面卸载的影响（使用 `navigator.sendBeacon()` 更佳）
</aside>

---

## 21.3 调试技术

### 21.3.1 把消息记录到控制台

`console` 对象提供多种级别的日志方法：

| **方法** | **用途** |
| --- | --- |
| `console.log()` | 通用日志输出 |
| `console.info()` | 信息性消息 |
| `console.warn()` | 警告消息（黄色图标） |
| `console.error()` | 错误消息（红色图标） |

### 21.3.2 理解控制台的运行时

在开发者工具的控制台中输入代码时，是在**当前页面的全局作用域**中执行的，可以直接访问页面中定义的变量和函数。

### 21.3.3 使用 JavaScript 调试器

在代码中插入 `debugger` 关键字，可以在运行时触发断点：

```jsx
function complexOperation() {
  debugger; // 如果开发者工具已打开，会在此暂停执行
  // ... 后续代码
}
```

也可以在浏览器开发者工具中**直接设置断点**，无需修改源代码。

### 21.3.4 在页面中打印消息

在开发阶段，可以在页面中创建一个专门用于显示调试信息的区域：

```jsx
function log(message) {
  const console = document.getElementById("debuginfo");
  if (console === null) {
    const el = document.createElement("div");
    el.id = "debuginfo";
    el.style.cssText =
      "position:fixed;bottom:0;width:100%;background:#eee;border-top:1px solid #ccc;padding:5px;font:12px monospace;max-height:200px;overflow:auto";
    document.body.appendChild(el);
  }
  document.getElementById("debuginfo").innerHTML += `<p>${message}</p>`;
}
```

### 21.3.5 补充控制台方法

如果某些浏览器不支持 `console` 对象，可以自定义一个：

```jsx
if (typeof console === "undefined") {
  console = {
    log() {},
    info() {},
    warn() {},
    error() {},
  };
}
```

### 21.3.6 抛出错误

在开发阶段，对关键函数参数使用 `assert` 模式可以尽早发现问题：

```jsx
function assert(condition, message) {
  if (!condition) {
    throw new Error(message);
  }
}

function divide(num1, num2) {
  assert(typeof num1 === "number" && typeof num2 === "number",
    "divide(): Both arguments must be numbers.");
  assert(num2 !== 0, "divide(): Cannot divide by zero.");
  return num1 / num2;
}
```

---

## 21.4 旧版 IE 的常见错误

<aside>
📝

本节内容主要涉及 IE 浏览器的历史遗留问题，在现代开发中已基本不会遇到。了解即可：

- 无效字符错误
- 未找到成员
- 未知运行时错误
- 语法错误
- 系统找不到指定资源
</aside>

---

## 21.5 小结

<aside>
📌

**本章核心要点：**

- `try/catch/finally` 是处理运行时错误的基本机制，`finally` 始终执行
- JavaScript 定义了 8 种内置错误类型，可通过继承 `Error` 创建自定义错误
- 应合理区分**致命错误**与**非致命错误**，采取不同的处理策略
- 在底层函数中**抛出**有意义的错误，在调用方**捕获**并处理
- 使用 `typeof`、`instanceof` 等做**防御性编程**，提前预防常见错误
- 利用 `console` 方法、`debugger` 关键字和浏览器开发者工具进行调试
- 建立**错误日志上报机制**，将客户端错误集中记录到服务器
</aside>