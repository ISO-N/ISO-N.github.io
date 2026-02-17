---
title: 第2章 HTML中的JavaScript
date: 2026-02-17 15:01:56
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
本章主要介绍如何将 JavaScript 引入 HTML 页面，包括 `<script>` 元素的使用方式、脚本加载策略、文档模式以及 `<noscript>` 元素。

## 2.1 <script> 元素

`<script>` 是向 HTML 页面中插入 JavaScript 的主要方法，由 Netscape 创造并在 Netscape Navigator 2 中首次实现，后被加入 HTML 规范。

### 主要属性

| **属性** | **说明** |
| --- | --- |
| `src` | 表示要执行的外部脚本文件的 URL，可以是同域或跨域的 |
| `type` | 脚本语言的 MIME 类型，默认值为 `text/javascript`；若值为 `module`，则代码被当作 ES6 模块处理 |
| `async` | 立即开始下载脚本，**不阻塞**页面解析，下载完成后立即执行（仅对外部脚本有效） |
| `defer` | 立即开始下载脚本，**不阻塞**页面解析，延迟到文档完全被解析和显示之后再执行（仅对外部脚本有效） |
| `crossorigin` | 配置跨域请求的 CORS 设置。`anonymous` 不携带凭据，`use-credentials` 携带凭据 |
| `integrity` | 用于验证子资源完整性（SRI），确保 CDN 资源未被篡改 |
| `nomodule` | 标记脚本在支持 ES6 模块的浏览器中不执行，用于向不支持模块的旧浏览器提供回退脚本 |

### 2.1.1 使用方式

**行内脚本：** 直接在 `<script>` 标签内编写代码

```jsx
<script>
  function sayHi() {
    console.log("Hi!");
  }
</script>
```

<aside>
⚠️

行内脚本中不能出现字符串 `</script>`，否则浏览器会将其解释为结束标签。需要使用转义字符 `<\/script>` 来避免。

</aside>

**外部脚本：** 通过 `src` 属性引入外部 `.js` 文件

```jsx
<script src="example.js"></script>
```

<aside>
💡

使用了 `src` 属性的 `<script>` 标签，内部的行内代码会被**忽略**。浏览器只会下载并执行外部文件。

</aside>

### 2.1.2 标签位置

**传统做法：** 放在 `<head>` 中

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="example1.js"></script>
    <script src="example2.js"></script>
  </head>
  <body>
    <!-- 页面内容 -->
  </body>
</html>
```

- 问题：所有脚本下载、解析和执行完毕后才开始渲染页面，导致**白屏**

**推荐做法：** 放在 `<body>` 末尾

```html
<!DOCTYPE html>
<html>
  <head></head>
  <body>
    <!-- 页面内容 -->
    <script src="example1.js"></script>
    <script src="example2.js"></script>
  </body>
</html>
```

- 页面内容先渲染，用户感知到的加载速度更快

### 2.1.3 推迟执行脚本（defer）

```html
<script defer src="example.js"></script>
```

- 脚本**立即开始下载**，但**延迟到整个页面解析完毕后执行**（在 `DOMContentLoaded` 事件之前）
- 多个 `defer` 脚本理论上会**按顺序执行**（但实践中不一定保证，最好只有一个 `defer` 脚本）
- 仅对**外部脚本**有效

### 2.1.4 异步执行脚本（async）

```html
<script async src="example.js"></script>
```

- 脚本**立即开始下载**，下载完成后**立即执行**，不等待其他脚本或 DOM 解析
- 多个 `async` 脚本的**执行顺序不确定**，取决于谁先下载完
- 一定会在页面的 `load` 事件之前执行，但可能在 `DOMContentLoaded` 之前或之后
- 适合与 DOM 无关且彼此独立的脚本（如统计分析脚本）

### defer vs async 对比

| **特性** | **defer** | **async** |
| --- | --- | --- |
| 下载时机 | 立即并行下载 | 立即并行下载 |
| 执行时机 | 文档解析完成后、`DOMContentLoaded` 前 | 下载完成后立即执行 |
| 执行顺序 | 按标签出现顺序 | 不保证顺序 |
| 是否阻塞解析 | 不阻塞 | 下载不阻塞，执行时阻塞 |
| 适用场景 | 依赖 DOM 或有顺序依赖的脚本 | 独立无依赖的脚本（如统计、广告） |

### 2.1.5 动态加载脚本

可以通过 DOM API 动态创建 `<script>` 元素来加载脚本：

```jsx
let script = document.createElement('script');
script.src = 'example.js';
document.head.appendChild(script);
```

- 默认以 `async` 方式加载
- 可设置 `script.async = false` 使其按顺序执行
- 这种方式对浏览器**预加载器不可见**，可能影响性能

<aside>
💡

为了让预加载器知道这些动态请求脚本的存在，可以在 `<head>` 中显式声明：
`<link rel="preload" href="example.js">`

</aside>

## 2.2 行内代码与外部文件

虽然可以直接在 HTML 中嵌入 JavaScript 代码，但**最佳实践是使用外部文件**，原因如下：

- **可维护性**：JavaScript 代码集中管理，不与 HTML 混在一起
- **缓存**：浏览器可以缓存外部 JavaScript 文件。如果多个页面引用同一个文件，只需下载一次
- **适应未来**：外部文件不存在 XHTML 或其他标记语言兼容性问题

## 2.3 文档模式

通过 `<!DOCTYPE>` 声明来切换文档模式，主要影响 CSS 渲染，某些情况下也会影响 JavaScript 解释。

### 三种模式

1. **混杂模式（Quirks Mode）**
    - 省略 `<!DOCTYPE>` 声明即进入
    - 浏览器以向后兼容的方式渲染，不同浏览器差异大
2. **标准模式（Standards Mode）**
    - 使用严格的 DOCTYPE，如：
    - `<!DOCTYPE html>`（HTML5，**推荐**）
3. **准标准模式（Almost Standards Mode）**
    - 与标准模式非常接近，主要区别在于处理图片间隙（table 中的空白）

<aside>
✅

现代开发中，始终使用 `<!DOCTYPE html>` 开启**标准模式**即可。

</aside>

## 2.4 <noscript> 元素

`<noscript>` 元素用于在以下情况向用户展示替代内容：

- 浏览器**不支持脚本**
- 浏览器的脚本功能被**禁用**

```html
<noscript>
  <p>本页面需要启用 JavaScript 才能正常使用。</p>
</noscript>
```

如果浏览器支持并启用了脚本，`<noscript>` 中的内容**不会被渲染**。

## 2.5 小结

<aside>
📝

1. 使用 `<script>` 元素将 JavaScript 插入 HTML，可以是行内代码或外部文件引用
2. 推荐使用**外部文件**来组织 JavaScript 代码（可维护性、缓存、兼容性）
3. 不使用 `defer` 和 `async` 时，`<script>` 按照在页面中出现的**顺序**被解释执行
4. `defer`：延迟到文档解析完成后按顺序执行
5. `async`：下载完立即执行，不保证顺序
6. 使用 `<!DOCTYPE html>` 确保标准模式
7. `<noscript>` 提供脚本不可用时的降级方案
</aside>