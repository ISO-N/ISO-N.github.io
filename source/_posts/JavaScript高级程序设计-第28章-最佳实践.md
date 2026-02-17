---
title: 第28章 最佳实践
date: 2026-02-17 15:02:19
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
本章从 **可维护性**、**性能** 和 **部署** 三个维度，总结了编写专业级 JavaScript 代码的最佳实践。

---

## 28.1 可维护性

可维护的代码具备以下特征：**可理解、直觉性、可适配、可扩展、可调试**。

### 28.1.1 编码规范

### 1. 可读性

- 合理使用 **缩进**（推荐 4 个空格）和 **注释**
- 需要注释的地方：
    - 函数和方法（描述用途、参数、返回值）
    - 大段代码（描述任务目的）
    - 复杂算法（解释实现思路）
    - Hack（说明为什么这样做以及浏览器兼容信息）

### 2. 变量和函数命名

- 变量名应该是 **名词**（如 `car`、`person`）
- 函数名应以 **动词** 开头（如 `getName()`、`isEnabled()`）
- 布尔值变量以 `is`、`can`、`has` 等开头
- 命名要尽量具有描述性和详细性，不怕名称长，只怕含义不清
- 避免无意义命名如 `foo`、`bar`、`temp`

### 3. 变量类型透明化

三种方式让变量类型一目了然：

- **初始化**：通过初始值暗示类型，如 `let found = false;`（布尔值）、`let count = -1;`（数值）、`let name = "";`（字符串）
- **匈牙利标记法**：在变量名前加前缀表示类型，如 `bFound`（布尔）、`sName`（字符串）、`oElement`（对象）
- **类型注释**：使用注释标注类型，如 `let found /*:Boolean*/ = false;`

### 28.1.2 松散耦合

### 1. 解耦 HTML/JavaScript

- **避免在 HTML 中写 JavaScript**：不用内联事件处理器（如 `onclick`），不用内联 `<script>` 执行重要逻辑
- **避免在 JavaScript 中写 HTML**：尽量不用 `innerHTML` 拼接大段 HTML，推荐使用模板引擎或从服务端加载
- 理想情况：HTML 负责结构，JavaScript 从外部文件引入

### 2. 解耦 CSS/JavaScript

- JavaScript 不要直接修改 `style` 属性（如 `element.style.color = "red"`）
- 推荐方式：通过 **增删 CSS 类名** 来改变样式

```jsx
// 不好的做法
element.style.color = "red";
element.style.backgroundColor = "blue";

// 好的做法
element.className = "highlight";
```

### 3. 解耦应用逻辑/事件处理程序

- 事件处理程序应只负责接收事件并调用应用逻辑，自身不应包含业务逻辑
- 规则：
    - 不要把 `event` 对象传给其他方法，只传所需数据
    - 应用逻辑函数应能在不触发事件的情况下独立运行

```jsx
// 不好的做法
function handleClick(event) {
    let popup = document.getElementById("popup");
    popup.style.left = event.clientX + "px";
    popup.style.top = event.clientY + "px";
    popup.className = "reveal";
}

// 好的做法
function showPopup(x, y) {
    let popup = document.getElementById("popup");
    popup.style.left = x + "px";
    popup.style.top = y + "px";
    popup.className = "reveal";
}
function handleClick(event) {
    showPopup(event.clientX, event.clientY);
}
```

### 28.1.3 编程实践

### 1. 尊重对象所有权

不要修改不属于你的对象，包括：

- **不要给实例或原型添加属性/方法**
- **不要重定义已有方法**
- 涉及原生对象（`Object`、`Array`、`Document` 等）、DOM 对象、BOM 对象以及类库对象
- 如果需要扩展功能，推荐：创建新类型继承已有类型，或创建自定义工具类

### 2. 避免全局变量

- 最多创建 **一个全局对象** 作为命名空间

```jsx
// 单一全局变量（命名空间模式）
var MyApp = {
    name: "MyApplication",
    version: "1.0.0",
    getUser: function() { /* ... */ },
    setUser: function() { /* ... */ }
};
```

### 3. 避免与 null 比较

直接与 `null` 比较的代码不够健壮，应改用更精确的检测：

- 基本类型 → `typeof`
- 引用类型 → `instanceof`
- 函数 → `typeof`

```jsx
// 不好的做法
if (values != null) { ... }

// 好的做法
if (values instanceof Array) { ... }
```

### 4. 使用常量

将反复使用的值提取为常量，便于集中管理和修改：

- URL
- 任何可能变化的字符串值（如 UI 文本）
- 重复使用的数值
- 配置项

```jsx
const Constants = {
    INVALID_VALUE_MSG: "Input not valid.",
    API_URL: "/api/data/"
};
```

---

## 28.2 性能

### 28.2.1 作用域意识

### 1. 避免全局查找

- 全局变量/函数的查找需要遍历整个作用域链，开销较大
- 如果函数中多次引用 `document` 等全局对象，应将其 **缓存到局部变量**

```jsx
// 不好的做法
function updateUI() {
    let imgs = document.getElementsByTagName("img");
    for (let i = 0, len = imgs.length; i < len; i++) {
        imgs[i].title = document.title + " image " + i;
    }
    let msg = document.getElementById("msg");
    msg.innerHTML = "Update complete.";
}

// 好的做法：缓存 document 引用
function updateUI() {
    let doc = document;
    let imgs = doc.getElementsByTagName("img");
    for (let i = 0, len = imgs.length; i < len; i++) {
        imgs[i].title = doc.title + " image " + i;
    }
    let msg = doc.getElementById("msg");
    msg.innerHTML = "Update complete.";
}
```

### 2. 不使用 with 语句

- `with` 语句会创建额外的作用域，使内部代码的性能下降
- 和全局查找同理，用 **局部变量** 代替

### 28.2.2 选择正确的方法

### 1. 避免不必要的属性查找

算法复杂度概念（O 记号）：

| **记号** | **名称** | **说明** |
| --- | --- | --- |
| O(1) | 常量 | 无论多少值，执行时间恒定（如访问变量或数组元素） |
| O(log n) | 对数 | 总执行时间与值的数量相关，但不必完整遍历（如二分搜索） |
| O(n) | 线性 | 总执行时间与值的数量直接相关（如遍历数组） |
| O(n²) | 二次 | 总执行时间与值数量的平方相关（如冒泡排序） |
- 对象属性和数组元素的查找比变量访问慢
- 多次使用对象属性（尤其是多层嵌套如 `obj.a.b.c`）应 **缓存到局部变量**

### 2. 优化循环

- **简化终止条件**：避免每次循环都进行属性查找，提前缓存 `length`
- **简化循环体**：确保没有可以抽到循环外的运算
- **使用后测试循环**（`do-while`）：避免对终止条件的初始评估（适用于确定至少有一次迭代时）

```jsx
// 优化前
for (let i = 0; i < values.length; i++) {
    process(values[i]);
}

// 优化后
for (let i = values.length - 1; i >= 0; i--) {
    process(values[i]);
}
```

### 3. 展开循环（达夫设备）

当循环次数确定且非常大时，可以通过展开循环体来减少循环控制开销：

```jsx
// 达夫设备（Duff's Device）
let iterations = Math.ceil(values.length / 8);
let startAt = values.length % 8;
let i = 0;

if (startAt > 0) {
    do {
        process(values[i++]);
    } while (--startAt > 0);
}
do {
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
    process(values[i++]);
} while (--iterations > 0);
```

> 对于大数据量的迭代，展开循环可以显著提高性能
> 

### 4. 避免重复解释

避免在代码中使用 `eval()`、`Function()` 构造器或给 `setTimeout`/`setInterval` 传字符串：

```jsx
// 不好的做法 — 需要对字符串代码进行二次解释
eval("alert('Hello!')");
let sayHi = new Function("alert('Hello!')");
setTimeout("alert('Hello!')", 500);

// 好的做法 — 直接使用函数
alert("Hello!");
let sayHi = function() { alert("Hello!"); };
setTimeout(function() { alert("Hello!"); }, 500);
```

### 5. switch 优于 if-else

- 当有超过两个条件分支时，`switch` 通常比 `if-else` 更快
- 把最可能出现的条件放在前面

### 6. 位运算

- 位运算在底层操作，比布尔运算快
- 对于使用 `% 2` 判断奇偶，可用 `& 1` 代替

```jsx
// 使用位运算判断奇偶
if (i & 1) {
    // 奇数
} else {
    // 偶数
}
```

### 28.2.3 语句最少化

### 1. 多个变量声明合并

```jsx
// 不好的做法
let count = 5;
let color = "blue";
let values = [1, 2, 3];
let now = new Date();

// 好的做法
let count = 5,
    color = "blue",
    values = [1, 2, 3],
    now = new Date();
```

### 2. 使用数组和对象字面量

```jsx
// 不好的做法
let values = new Array();
values[0] = 123;
values[1] = 456;
values[2] = 789;

let person = new Object();
person.name = "Nicholas";
person.age = 29;

// 好的做法
let values = [123, 456, 789];
let person = { name: "Nicholas", age: 29 };
```

### 28.2.4 优化 DOM 交互

### 1. 实时（Live）集合最小化

- `getElementsByTagName()`、`getElementsByClassName()` 等返回的是 **实时集合**（HTMLCollection），每次访问都会重新查询 DOM
- 把集合长度缓存到局部变量，减少访问次数
- 如果不需要实时集合，考虑使用 `querySelectorAll()`（返回静态 NodeList）

### 2. 使用 innerHTML

- 对于大量 DOM 变更，使用 `innerHTML` 比反复调用 `createElement()` + `appendChild()` 要快
- 原因：`innerHTML` 在后台调用的是浏览器 **原生 HTML 解析器**，比 JavaScript DOM 操作快得多
- 技巧：**先拼接完字符串，再一次性赋值给 innerHTML**

```jsx
// 不好的做法：每次循环都触发 innerHTML
let list = document.getElementById("myList");
for (let i = 0; i < 10; i++) {
    list.innerHTML += `<li>Item ${i}</li>`;  // 每次赋值都触发解析
}

// 好的做法：先拼接再一次性赋值
let list = document.getElementById("myList");
let html = "";
for (let i = 0; i < 10; i++) {
    html += `<li>Item ${i}</li>`;
}
list.innerHTML = html;
```

### 3. 使用事件委托

- 利用事件冒泡机制，在 **父元素** 上统一处理子元素事件，而不是为每个子元素单独绑定
- 减少事件处理器的数量，降低内存开销，提升性能

```jsx
// 不好的做法：为每个 li 绑定事件
let items = document.querySelectorAll("li");
for (let item of items) {
    item.addEventListener("click", function() { /* ... */ });
}

// 好的做法：事件委托到父元素
document.getElementById("myList").addEventListener("click", function(event) {
    let target = event.target;
    if (target.tagName === "LI") {
        // 处理点击
    }
});
```

### 4. 注意 HTMLCollection

HTMLCollection 对象的"活性"是性能隐患，访问它相当于在实时查询 DOM。以下操作会返回 HTMLCollection：

- `getElementsByTagName()`
- `getElementsByClassName()`
- `document.images`、`document.forms` 等
- 使用 `childNodes` 属性

---

## 28.3 部署

### 28.3.1 构建流程

<aside>
⚙️

专业的 JavaScript 项目不应直接把源代码部署到生产环境，而是需要一个 **构建流程**。

</aside>

构建流程通常包括：

1. **代码检查**（Linting）：通过 ESLint 等工具检查语法错误和不良实践
2. **打包/合并**：将多个文件合并为一个或少量文件（如使用 Webpack、Rollup）
3. **转译**：使用 Babel 等工具将新语法转为旧版浏览器兼容的代码
4. **压缩/混淆**：减小文件体积

### 28.3.2 验证

### 代码检查（Linting）

- 使用 **ESLint** 等工具在构建阶段自动检查代码
- 常见检查项：
    - 使用 `eval()`
    - 使用 `with`
    - 变量声明前使用
    - `switch` 中遗漏 `break`
    - 不推荐的代码模式
    - 语法错误

### 28.3.3 压缩

### 1. 代码压缩（Minification）

通过安全的代码变换来减少文件体积：

- 删除空白（空格、换行、缩进）
- 删除注释
- 缩短变量名、函数名和其他标识符
- 常用工具：**UglifyJS**、**Terser**

> 压缩后代码不可读但功能完全一致，生产环境应始终使用压缩后的代码
> 

### 2. JavaScript 编译

编译器（如 **Google Closure Compiler**）更进一步优化代码：

- 类似于压缩工具的功能
- 额外进行代码优化（如内联函数、删除死代码、常量折叠等）

### 3. HTTP 压缩

- 服务器端启用 **Gzip** 或 **Brotli** 压缩传输 JavaScript 文件
- 大多数服务器和浏览器都支持
- 通常能将文件体积再减少 **70%** 左右
- 请求头 `Accept-Encoding: gzip, deflate`，响应头 `Content-Encoding: gzip`

---

## 28.4 小结

<aside>
📝

**可维护性**

- 遵循编码规范（可读性、命名、类型透明化）
- 松散耦合（HTML/CSS/JS 分离，应用逻辑与事件处理分离）
- 尊重对象所有权，避免全局变量，使用常量，做好类型检测

**性能**

- 注意作用域（缓存全局查找，禁用 `with`）
- 选择正确方法（缓存属性查找、优化循环、避免重复解释、善用位运算）
- 最少化语句数量（合并声明、使用字面量）
- 优化 DOM 交互（减少实时集合访问、innerHTML 一次性赋值、事件委托）

**部署**

- 使用构建流程自动化代码检查、打包、转译
- 代码压缩 + HTTP 压缩，最大程度减小传输体积
</aside>