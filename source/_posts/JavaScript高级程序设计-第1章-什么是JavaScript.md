---
title: 第1章 什么是JavaScript
date: 2026-02-17 15:01:55
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
本章主要介绍 JavaScript 的**起源、发展历程**以及它由哪些部分组成，帮助建立对这门语言的整体认识。

---

## 1.1 简短的历史回顾

- **1995 年**，Netscape 公司的 **Brendan Eich** 开发了 JavaScript，最初名为 **Mocha**，后改名为 **LiveScript**，最终为了蹭 Java 的热度而命名为 **JavaScript**
- 最初的目的是在**客户端浏览器**中处理一些简单的输入验证，避免与服务器的无谓往返通信
- **1996 年**，微软在 IE3 中加入了名为 **JScript** 的 JavaScript 实现，由此出现了两个版本的 JavaScript 并存的局面
- **1997 年**，JavaScript 1.1 作为提案被提交给 **ECMA（欧洲计算机制造商协会）**，由第 39 技术委员会（**TC39**）负责标准化，最终形成了 **ECMA-262** 标准，即 **ECMAScript**
- **1998 年**，ISO/IEC 也采纳了 ECMAScript 作为标准（ISO/IEC-16262）

## 1.2 JavaScript 实现

JavaScript 的实现远不止 ECMAScript，完整的 JavaScript 包含以下**三个部分**：

<aside>
💡

**JavaScript = ECMAScript + DOM + BOM**

</aside>

### 1.2.1 ECMAScript

- ECMAScript 即 **ECMA-262 定义的语言**，并不局限于浏览器。浏览器只是 ECMAScript 的一个**宿主环境（host environment）**
- 其他宿主环境包括：**Node.js**、Adobe Flash（已淘汰）等
- ECMAScript 定义了：
    - **语法**
    - **类型**
    - **语句**
    - **关键字**
    - **保留字**
    - **操作符**
    - **全局对象**

### ECMAScript 版本演进

| 版本 | 年份 | 重要变化 |
| --- | --- | --- |
| **第 1 版** | 1997 | 与 Netscape 的 JavaScript 1.1 基本相同，做了少量编辑性修改 |
| **第 2 版** | 1998 | 编辑性修订，与 ISO/IEC-16262 保持一致，无实质变化 |
| **第 3 版** | 1999 | 首次真正的修改：新增字符串处理、错误定义、数值输出、正则表达式、`try/catch` 等 |
| **第 4 版** | 废弃 | 改动过于激进（强类型变量、类、接口等），最终被废弃 |
| **第 5 版（ES5）** | 2009 | 新增 `JSON` 对象、严格模式、`Array` 新方法、`Object` 元编程方法等 |
| **第 6 版（ES6 / ES2015）** | 2015 | 里程碑版本：`class`、模块、箭头函数、`Promise`、`let/const`、迭代器、生成器、解构、模板字面量等 |
| **第 7 版（ES2016）** | 2016 | `Array.prototype.includes()`、指数操作符 `**` |
| **第 8 版（ES2017）** | 2017 | `async/await`、`Object.values()/entries()`、字符串填充、`SharedArrayBuffer` 等 |
| **第 9 版（ES2018）** | 2018 | 异步迭代、剩余/扩展属性、`Promise.finally()`、正则表达式增强等 |
| **第 10 版（ES2019）** | 2019 | `Array.prototype.flat()/flatMap()`、`Object.fromEntries()`、`String.prototype.trimStart()/trimEnd()` 等 |

> 自 ES6 起，ECMA 采用**年度发布**节奏，每年 6 月发布新版本。
> 

### 1.2.2 DOM

- **DOM（文档对象模型，Document Object Model）** 是一个应用编程接口（API），用于在 HTML 中使用扩展的 XML
- DOM 将整个页面抽象为一个**分层节点树**，允许开发者通过 API **增删改查**页面的内容和结构

#### 为什么 DOM 是必需的？

在 IE4 和 Netscape Navigator 4 各自支持不同的**动态 HTML（DHTML）** 之后，开发者面临严重的跨浏览器兼容问题。为了保持 Web 的**跨平台特性**，**W3C** 开始制定 DOM 标准。

#### DOM 级别

- **DOM Level 1（1998 年）**
    - **DOM Core**：提供映射文档结构的方式，方便访问和操作文档任意部分
    - **DOM HTML**：在 DOM Core 基础上扩展了 HTML 特有的对象和方法
- **DOM Level 2**
    - **DOM 视图**：描述追踪文档不同视图的接口（如 CSS 样式应用前后）
    - **DOM 事件**：描述事件及事件处理的接口
    - **DOM 样式**：描述基于 CSS 为元素应用样式的接口
    - **DOM 遍历和范围**：描述遍历和操作 DOM 树的接口
- **DOM Level 3**
    - 增加了以统一方式**加载和保存文档**的方法（DOM Load and Save）
    - 新增**验证文档**的方法（DOM Validation）
    - 扩展支持了所有 **XML 1.0** 特性
- **DOM Level 4（DOM Living Standard）**
    - 新增 `Mutation Observers` 等

### 1.2.3 BOM

- **BOM（浏览器对象模型，Browser Object Model）** 用于访问和操作**浏览器窗口**
- BOM 主要针对**浏览器窗口和子窗口（frame）**，但通常也包含以下扩展：
    - 弹出新浏览器窗口的能力
    - 移动、缩放和关闭浏览器窗口的能力
    - `navigator` 对象：提供浏览器的详尽信息
    - `location` 对象：提供浏览器加载页面的详尽信息
    - `screen` 对象：提供屏幕分辨率的详尽信息
    - `performance` 对象：提供浏览器内存占用、导航行为和时间统计的详尽信息
    - 对 **cookie** 的支持
    - 其他自定义对象，如 `XMLHttpRequest`

<aside>
⚠️

BOM 长期没有正式标准，每个浏览器各自实现。直到 **HTML5** 的出现，BOM 的实现细节才被正式纳入规范，浏览器间的差异大大减少。

</aside>

## 1.3 JavaScript 版本

- **Mozilla** 是唯一仍在延续最初 JavaScript **版本编号**的浏览器厂商
- Firefox 的 JavaScript 版本号对应的是其 Gecko 渲染引擎中的 JavaScript 实现版本
- 但在实际开发中，我们更多关注的是 **ECMAScript 的版本兼容性**，而非浏览器的 JavaScript 版本号

## 1.4 小结

<aside>
📝

**核心要点**

- JavaScript 由三部分组成：**ECMAScript**（核心语言）、**DOM**（文档操作）、**BOM**（浏览器交互）
- ECMAScript 由 ECMA-262 定义，提供核心语言功能，**不依赖**于任何特定宿主环境
- DOM 提供与**网页内容**交互的方法和接口，由 W3C 制定标准
- BOM 提供与**浏览器**交互的方法和接口，HTML5 之后趋于标准化
- 这三部分在**不同浏览器**中的支持程度不同，了解各浏览器的兼容性对实际开发至关重要
</aside>