---
title: 第15章 DOM扩展
date: 2026-02-17 15:02:07
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
尽管 DOM API 已经非常完善，但仍有不少规范对其进行了扩展，以提供更多原生功能，减少对 JavaScript 库的依赖。本章主要介绍三个方面的 DOM 扩展：**Selectors API**、**Element Traversal** 和 **HTML5 DOM 扩展**。

---

## 15.1 Selectors API

Selectors API 是 W3C 推荐标准，允许通过 **CSS 选择符** 来匹配 DOM 元素，核心方法有两个：`querySelector()` 和 `querySelectorAll()`。它们可在 `Document` 和 `Element` 类型上调用。

### 15.1.1 querySelector()

- 接收一个 **CSS 选择符** 参数，返回匹配的 **第一个** 后代元素；没有匹配则返回 `null`
- 在 `Document` 上调用时搜索整个文档；在 `Element` 上调用时只搜索该元素的后代

```jsx
// 获取 body 元素
let body = document.querySelector("body");

// 获取 id 为 myDiv 的元素
let myDiv = document.querySelector("#myDiv");

// 获取第一个类名为 selected 的元素
let selected = document.querySelector(".selected");

// 获取第一个类名为 button 的 img 元素
let img = document.body.querySelector("img.button");
```

### 15.1.2 querySelectorAll()

- 接收同样的 CSS 选择符参数，但返回 **所有** 匹配元素，结果是一个 **静态的 NodeList**（快照，不会实时更新）
- 返回值可通过 `for-of`、索引、`item()` 方法访问

```jsx
// 获取 id 为 myDiv 中所有 em 元素
let ems = document.getElementById("myDiv").querySelectorAll("em");

// 获取所有类名为 selected 的元素
let selecteds = document.querySelectorAll(".selected");

// 获取所有 p 元素中的 strong 元素
let strongs = document.querySelectorAll("p strong");
```

<aside>
⚠️

`querySelectorAll()` 返回的 NodeList 是 **静态快照**，与 `getElementsByTagName()` 等返回的**动态 NodeList** 不同。DOM 结构变化不会自动反映到该结果中。

</aside>

### 15.1.3 matches()

- 接收一个 CSS 选择符参数，如果调用元素**匹配**该选择符则返回 `true`，否则返回 `false`
- 常用于检测某个元素是否会被 `querySelector()` / `querySelectorAll()` 返回

```jsx
if (document.body.matches("body.page1")) {
  // body 元素具有 page1 类
}
```

---

## 15.2 元素遍历（Element Traversal）

`childNodes` 属性在不同浏览器中对空白文本节点的处理不一致。Element Traversal API 提供了一组只关注 **Element 节点** 的属性，避免了空白文本节点的干扰：

| **属性** | **说明** |
| --- | --- |
| `childElementCount` | 返回子元素数量（不含文本节点和注释） |
| `firstElementChild` | 指向第一个 Element 子节点 |
| `lastElementChild` | 指向最后一个 Element 子节点 |
| `previousElementSibling` | 指向前一个 Element 同胞节点 |
| `nextElementSibling` | 指向后一个 Element 同胞节点 |

```jsx
// 以前的写法——需要检查 nodeType
let child = parentElement.firstChild;
while (child) {
  if (child.nodeType === 1) { // Element 节点
    processChild(child);
  }
  child = child.nextSibling;
}

// 使用 Element Traversal——更简洁
let child = parentElement.firstElementChild;
while (child) {
  processChild(child);
  child = child.nextElementSibling;
}
```

---

## 15.3 HTML5

HTML5 规范包含了大量与 DOM 相关的扩展，极大地增强了对标记的程序化操控能力。

### 15.3.1 CSS 类扩展

#### 1. getElementsByClassName()

- 接收一个或多个类名的字符串（空格分隔），返回包含对应类名的所有元素的 **动态 NodeList**
- 可在 `document` 或任意元素上调用

```jsx
// 获取所有同时包含 username 和 current 类的元素
let allCurrentUsernames = document.getElementsByClassName("username current");

// 在特定元素的后代中查找
let selected = myDiv.getElementsByClassName("selected");
```

#### 2. classList 属性

以往操作类名需要通过 `className` 属性拼接字符串，非常繁琐。HTML5 新增了 `classList` 属性，类型为 **DOMTokenList**，提供以下方法：

| **方法** | **说明** |
| --- | --- |
| `add(value)` | 添加类名（已存在则忽略） |
| `contains(value)` | 检查是否包含指定类名，返回布尔值 |
| `remove(value)` | 移除指定类名 |
| `toggle(value)` | 切换类名：存在则移除，不存在则添加 |

```jsx
// 移除 disabled 类
div.classList.remove("disabled");

// 添加 current 类
div.classList.add("current");

// 切换 user 类
div.classList.toggle("user");

// 检查是否包含 disabled 类
if (div.classList.contains("disabled")) {
  // ...
}
```

### 15.3.2 焦点管理

- `document.activeElement`：始终指向当前拥有焦点的 DOM 元素；页面刚加载完成时为 `document.body`
- `document.hasFocus()`：返回布尔值，表示文档是否拥有焦点

```jsx
let button = document.getElementById("myButton");
button.focus();

console.log(document.activeElement === button); // true
console.log(document.hasFocus()); // true
```

### 15.3.3 HTMLDocument 扩展

#### 1. readyState 属性

`document.readyState` 有两个可能的值：

- `"loading"`：文档正在加载
- `"complete"`：文档加载完成

```jsx
if (document.readyState === "complete") {
  // 文档加载完毕，可以安全操作 DOM
}
```

#### 2. compatMode 属性

指示浏览器当前的渲染模式：

- `"CSS1Compat"`：标准模式
- `"BackCompat"`：混杂（怪异）模式

#### 3. head 属性

```jsx
let head = document.head; // 直接获取 <head> 元素
```

### 15.3.4 字符集属性

- `document.characterSet`：获取/设置文档使用的字符集，默认为 `"UTF-16"`
- 可通过 `<meta>` 标签、响应头或该属性直接设置

```jsx
console.log(document.characterSet); // "UTF-8"
document.characterSet = "UTF-8";
```

### 15.3.5 自定义数据属性

HTML5 允许在元素上使用 `data-` 前缀定义自定义非标准属性，通过 `dataset` 属性访问：

```html
<div id="myDiv" data-appId="12345" data-myname="Nicholas"></div>
```

```jsx
let div = document.getElementById("myDiv");

// 读取
let appId = div.dataset.appid;  // 注意：名称全部小写
let myName = div.dataset.myname;

// 写入
div.dataset.appid = "23456";
div.dataset.myname = "Michael";
```

### 15.3.6 插入标记

#### 1. innerHTML 属性

- **读取**时：返回元素所有后代的 HTML 字符串（包含标签）
- **写入**时：根据提供的字符串解析为 DOM 子树，替换元素的所有子节点

```jsx
div.innerHTML = "Hello & welcome, <b>\"reader\"!</b>";
```

<aside>
⚠️

将用户输入直接赋值给 `innerHTML` 存在 **XSS（跨站脚本攻击）** 风险。虽然现代浏览器会阻止通过 `innerHTML` 插入的 `<script>` 执行，但其他方式（如 `<img onerror>`）仍可触发脚本。

</aside>

#### 2. outerHTML 属性

- **读取**时：返回调用元素自身及其所有后代的 HTML 字符串
- **写入**时：用新的 DOM 子树**替换调用元素本身**

```jsx
// 读取：包含元素自身的标签
console.log(div.outerHTML);

// 写入：整个 div 被新内容替换
div.outerHTML = "<p>This is a paragraph.</p>";
// 等同于：
let p = document.createElement("p");
p.appendChild(document.createTextNode("This is a paragraph."));
div.parentNode.replaceChild(p, div);
```

#### 3. insertAdjacentHTML() 与 insertAdjacentText()

接收两个参数：**插入位置**和要插入的 HTML/文本字符串。位置参数（不区分大小写）：

| **位置值** | **含义** |
| --- | --- |
| `"beforebegin"` | 元素自身的前面（作为前一个同胞节点） |
| `"afterbegin"` | 元素的第一个子节点之前 |
| `"beforeend"` | 元素的最后一个子节点之后 |
| `"afterend"` | 元素自身的后面（作为后一个同胞节点） |

```jsx
// 在元素前面插入
element.insertAdjacentHTML("beforebegin", "<p>Hello world!</p>");

// 在元素内部的开头插入
element.insertAdjacentHTML("afterbegin", "<p>Hello world!</p>");

// 在元素内部的末尾插入
element.insertAdjacentHTML("beforeend", "<p>Hello world!</p>");

// 在元素后面插入
element.insertAdjacentHTML("afterend", "<p>Hello world!</p>");
```

#### 4. 内存与性能问题

<aside>
💡

**性能建议：**

- 频繁使用 `innerHTML` 做小修改效率不高；最好**构建好完整字符串后一次性赋值**
- 被 `innerHTML` 替换掉的子树如果有绑定事件处理程序或引用了 JS 对象，可能导致**内存泄漏**。替换前应先手动移除相关引用
</aside>

### 15.3.7 scrollIntoView()

将元素滚动到浏览器视口中可见的位置，接收一个参数：

- `true` 或 `{ block: "start" }`：元素顶部与视口顶部对齐（默认）
- `false` 或 `{ block: "end" }`：元素底部与视口底部对齐
- 还支持 `{ behavior: "smooth" }` 实现平滑滚动

```jsx
// 元素顶部对齐视口顶部
document.forms[0].scrollIntoView();

// 平滑滚动，底部对齐
document.forms[0].scrollIntoView({ behavior: "smooth", block: "end" });
```

---

## 15.4 专有扩展

除了标准规范以外，部分浏览器还提供了一些专有的 DOM 扩展，其中一些后来被纳入标准。

### 15.4.1 children 属性

- `HTMLCollection` 类型，只包含元素的 **Element 类型子节点**（与 `childNodes` 不同，不包含文本节点、注释等）
- 各主流浏览器均已支持

### 15.4.2 contains() 方法

用于判断一个节点是否是另一个节点的后代：

```jsx
// 检查 body 是否包含某个元素
console.log(document.documentElement.contains(document.body)); // true
```

也可使用 DOM Level 3 的 `compareDocumentPosition()` 方法，该方法返回一个位掩码：

| **掩码值** | **关系** |
| --- | --- |
| 0x1 | 断连（不在同一文档中） |
| 0x2 | 参考节点在前 |
| 0x4 | 参考节点在后 |
| 0x8 | 参考节点包含传入节点 |
| 0x10 | 传入节点包含参考节点 |

```jsx
let result = document.documentElement.compareDocumentPosition(document.body);
console.log(!!(result & 0x10)); // true，body 被 documentElement 包含
```

### 15.4.3 插入标记 — innerText 与 outerText

#### innerText

- **读取**时：按深度优先拼接子树中所有文本节点的值
- **写入**时：移除所有子节点，替换为纯文本（HTML 标签会被**转义**，不会被解析）

```jsx
div.innerText = "Hello & welcome, <b>reader</b>!";
// 实际渲染为纯文本: Hello & welcome, <b>reader</b>!
// 等同于设置了一个文本节点
```

<aside>
💡

`innerText` 可以用于快速**去除 HTML 标签**：`div.innerText = div.innerText;`

</aside>

#### outerText

- **读取**时：与 `innerText` 相同
- **写入**时：替换整个元素（包括元素自身）为文本节点，使用时需谨慎

---

## 15.5 小结

本章介绍了以下主要 DOM 扩展：

- **Selectors API**：通过 CSS 选择符查询 DOM，包括 `querySelector()`、`querySelectorAll()` 和 `matches()`
- **Element Traversal**：提供纯元素遍历属性，避免空白文本节点的干扰
- **HTML5 DOM 扩展**：
    - `classList` 简化类名操作
    - 焦点管理（`activeElement`、`hasFocus()`）
    - `HTMLDocument` 扩展（`readyState`、`compatMode`、`head`）
    - 自定义数据属性（`dataset`）
    - 插入标记（`innerHTML`、`outerHTML`、`insertAdjacentHTML()`）
    - `scrollIntoView()` 滚动控制
- **专有扩展**：`children`、`contains()`、`innerText`、`outerText` 等