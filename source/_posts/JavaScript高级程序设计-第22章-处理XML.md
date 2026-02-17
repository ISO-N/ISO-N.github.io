---
title: 第22章 处理XML
date: 2026-02-17 15:02:13
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
本章介绍浏览器中处理 XML 数据的几种技术，包括 **XML DOM**、**XPath** 和 **XSLT**。

---

## 22.1 浏览器对 XML DOM 的支持

### 22.1.1 DOM Level 2 Core

DOM Level 2 增加了 `document.implementation.createDocument()` 方法，可以创建空白 XML 文档：

```jsx
// 创建一个空的 XML 文档
// 参数：命名空间 URI、文档元素的标签名、文档类型
let xmldom = document.implementation.createDocument("", "root", null);

console.log(xmldom.documentElement.tagName); // "root"

let child = xmldom.createElement("child");
xmldom.documentElement.appendChild(child);
```

- 第一个参数是命名空间 URI（通常传空字符串）
- 第二个参数是文档元素的标签名
- 第三个参数是文档类型（通常为 `null`）

可以通过 `document.implementation.hasFeature()` 检测 DOM Level 2 XML 支持：

```jsx
let hasXmlDom = document.implementation.hasFeature("XML", "2.0");
```

### 22.1.2 DOMParser 类型

`DOMParser` 可以将 **XML 字符串解析为 DOM 文档**：

```jsx
let parser = new DOMParser();
let xmldom = parser.parseFromString("<root><child/></root>", "text/xml");

console.log(xmldom.documentElement.tagName); // "root"
console.log(xmldom.documentElement.firstChild.tagName); // "child"

let anotherChild = xmldom.createElement("child");
xmldom.documentElement.appendChild(anotherChild);
```

- `parseFromString()` 接受两个参数：**XML 字符串** 和 **内容类型**（`"text/xml"`）
- 返回 `Document` 的实例，可以执行 DOM 操作

**解析错误处理：**

解析出错时，`DOMParser` 不会抛出错误，而是返回一个包含 `<parsererror>` 元素的文档：

```jsx
let parser = new DOMParser();
let xmldom = parser.parseFromString("<root>", "text/xml");

let errors = xmldom.getElementsByTagName("parsererror");
if (errors.length > 0) {
    console.log("Parsing error!");
}
```

### 22.1.3 XMLSerializer 类型

`XMLSerializer` 与 `DOMParser` 相反，将 **DOM 文档序列化为 XML 字符串**：

```jsx
let serializer = new XMLSerializer();
let xml = serializer.serializeToString(xmldom);
console.log(xml);
```

- `serializeToString()` 接受一个 DOM 节点参数
- 可以传入任何有效的 DOM 节点，包括文档节点和单个元素节点
- 如果传入非 DOM 节点，会抛出错误

---

## 22.2 浏览器对 XPath 的支持

**XPath** 是一种用于在 XML 文档中定位节点的语言，能够替代 DOM 遍历实现更精确高效的节点查找。

### 22.2.1 DOM Level 3 XPath

DOM Level 3 XPath 规范定义了在 DOM 中求值 XPath 表达式的接口。检测是否支持：

```jsx
let supportsXPath = document.implementation.hasFeature("XPath", "3.0");
```

**核心类型与方法：**

最重要的类型是 `XPathEvaluator`，包含以下方法：

- `createExpression(expression, nsresolver)` — 预编译 XPath 表达式，返回 `XPathExpression`
- `createNSResolver(node)` — 基于指定节点创建命名空间解析器
- `evaluate(expression, context, nsresolver, type, result)` — 求值 XPath 表达式

**`evaluate()` 方法参数：**

| 参数 | 说明 |
| --- | --- |
| `expression` | XPath 表达式字符串 |
| `context` | 上下文节点（从哪个节点开始查找） |
| `nsresolver` | 命名空间解析器（无命名空间时为 `null`） |
| `type` | 期望的返回结果类型（`XPathResult` 常量） |
| `result` | 用于存放结果的 `XPathResult` 对象（通常为 `null`） |

**XPathResult 常量：**

| 常量 | 说明 |
| --- | --- |
| `XPathResult.ANY_TYPE` | 返回与 XPath 表达式匹配的自然类型 |
| `XPathResult.NUMBER_TYPE` | 数值结果 |
| `XPathResult.STRING_TYPE` | 字符串结果 |
| `XPathResult.BOOLEAN_TYPE` | 布尔值结果 |
| `XPathResult.UNORDERED_NODE_ITERATOR_TYPE` | 无序节点迭代器 |
| `XPathResult.ORDERED_NODE_ITERATOR_TYPE` | 有序节点迭代器 |
| `XPathResult.UNORDERED_NODE_SNAPSHOT_TYPE` | 无序节点快照 |
| `XPathResult.ORDERED_NODE_SNAPSHOT_TYPE` | 有序节点快照 |
| `XPathResult.ANY_UNORDERED_NODE_TYPE` | 匹配的任意单个节点（无序） |
| `XPathResult.FIRST_ORDERED_NODE_TYPE` | 匹配的第一个节点（有序） |

**使用示例：**

```jsx
let parser = new DOMParser();
let xmldom = parser.parseFromString(
    "<employees><employee name='Alice'/><employee name='Bob'/></employees>",
    "text/xml"
);

// 使用迭代器模式获取节点
let result = xmldom.evaluate(
    "employee",
    xmldom.documentElement,
    null,
    XPathResult.ORDERED_NODE_ITERATOR_TYPE,
    null
);

let node = result.iterateNext();
while (node) {
    console.log(node.getAttribute("name"));
    node = result.iterateNext();
}
```

**快照模式 vs 迭代器模式：**

- **迭代器模式**：结果与文档绑定，文档修改后迭代器失效
- **快照模式**：结果是节点的快照，文档修改不影响已获取的结果

```jsx
// 快照模式
let result = xmldom.evaluate(
    "employee",
    xmldom.documentElement,
    null,
    XPathResult.ORDERED_NODE_SNAPSHOT_TYPE,
    null
);

for (let i = 0; i < result.snapshotLength; i++) {
    console.log(result.snapshotItem(i).getAttribute("name"));
}
```

**获取单个节点：**

```jsx
let result = xmldom.evaluate(
    "employee",
    xmldom.documentElement,
    null,
    XPathResult.FIRST_ORDERED_NODE_TYPE,
    null
);

console.log(result.singleNodeValue.getAttribute("name")); // "Alice"
```

### 22.2.2 单个命名空间的 XPath

如果 XML 文档使用了命名空间，XPath 查询时必须提供命名空间解析器：

```jsx
// 方式一：使用 createNSResolver()
let nsresolver = xmldom.createNSResolver(xmldom.documentElement);

let result = xmldom.evaluate(
    "wrox:book/wrox:author",
    xmldom.documentElement,
    nsresolver,
    XPathResult.ORDERED_NODE_SNAPSHOT_TYPE,
    null
);

// 方式二：自定义命名空间解析器函数
let nsresolver2 = function(prefix) {
    switch (prefix) {
        case "wrox": return "http://www.wrox.com/";
        default: return null;
    }
};
```

---

## 22.3 浏览器对 XSLT 的支持

**XSLT**（可扩展样式表语言转换）是一种将 XML 文档转换为其他格式（如 HTML）的语言。

### 22.3.1 XSLTProcessor 类型

`XSLTProcessor` 用于通过 **XSLT 样式表** 转换 XML 文档：

```jsx
// 1. 加载 XSLT 样式表
let xsltParser = new DOMParser();
let xsltdom = xsltParser.parseFromString(`
    <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
        <xsl:output method="html"/>
        <xsl:template match="/">
            <ul>
                <xsl:for-each select="employees/employee">
                    <li><xsl:value-of select="@name"/></li>
                </xsl:for-each>
            </ul>
        </xsl:template>
    </xsl:stylesheet>
`, "text/xml");

// 2. 创建 XSLTProcessor 并导入样式表
let processor = new XSLTProcessor();
processor.importStylesheet(xsltdom);

// 3. 转换 XML 文档
// transformToDocument() — 返回完整文档
let resultDocument = processor.transformToDocument(xmldom);

// transformToFragment() — 返回文档片段，可附加到现有文档
let fragment = processor.transformToFragment(xmldom, document);
document.body.appendChild(fragment);
```

**两种转换方法对比：**

| 方法 | 返回值 | 适用场景 |
| --- | --- | --- |
| `transformToDocument(xmldom)` | 完整的 `Document` 对象 | 需要对结果做进一步 DOM 操作 |
| `transformToFragment(xmldom, ownerDoc)` | `DocumentFragment` 对象 | 直接插入到现有页面 |

### 22.3.2 使用参数

`XSLTProcessor` 支持通过 `setParameter()` 向 XSLT 样式表传递参数：

```jsx
// XSLT 样式表中使用 <xsl:param name="message"/>
// 然后在 JS 中设置参数值
processor.setParameter(null, "message", "Hello World!");

// 获取参数值
let value = processor.getParameter(null, "message");

// 移除参数
processor.removeParameter(null, "message");
```

- 三个参数：命名空间 URI（通常为 `null`）、参数名、参数值

### 22.3.3 重置处理器

调用 `reset()` 方法可以移除所有参数和样式表，恢复到初始状态：

```jsx
processor.reset();
// 之后可以重新导入另一个样式表
processor.importStylesheet(anotherXsltdom);
```

---

## 22.4 小结

<aside>
📌

**核心要点回顾：**

- **DOMParser** 将 XML 字符串 → DOM 文档；**XMLSerializer** 将 DOM 文档 → XML 字符串
- **XPath** 提供了强大的 XML 节点查询能力，支持迭代器和快照两种结果模式
- **XSLT** 通过 `XSLTProcessor` 实现 XML 文档的格式转换
- 解析错误不会抛出异常，需要检查返回文档中的 `<parsererror>` 元素
- 涉及命名空间的 XPath 查询，必须提供命名空间解析器
</aside>