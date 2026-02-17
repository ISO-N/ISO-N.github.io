---
title: 第16章 DOM2和DOM3
date: 2026-02-17 15:02:08
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
本章介绍 DOM Level 2 和 DOM Level 3 对 DOM 的扩展，包括对 XML 命名空间的支持、新增的 DOM 操作方法、样式的访问与操作、DOM 遍历与范围等。

---

## 16.1 DOM 的演进

DOM2 和 DOM3 Core 模块在 DOM1 的基础上增加了更多交互能力，也支持了更高级的 XML 特性。

### 16.1.1 XML 命名空间

**XML 命名空间**可以实现在同一文档中混合使用不同 XML 语言，避免元素命名冲突。命名空间使用 `xmlns` 特性指定：

```xml
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <title>Example</title>
  </head>
  <body>
    <svg xmlns="http://www.w3.org/2000/svg" ...>
      <!-- SVG 内容 -->
    </svg>
  </body>
</html>
```

也可以使用**命名空间前缀**：

```xml
<xhtml:html xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <xhtml:body>
    <xhtml:p>Hello!</xhtml:p>
  </xhtml:body>
</xhtml:html>
```

#### Node 的变化

DOM2 在 `Node` 类型上新增了与命名空间相关的属性：

- `localName`：不含命名空间前缀的节点名
- `namespaceURI`：节点的命名空间 URI，未指定则为 `null`
- `prefix`：命名空间前缀，未指定则为 `null`

DOM3 新增方法：

- `isDefaultNamespace(namespaceURI)`：判断是否为默认命名空间
- `lookupNamespaceURI(prefix)`：根据前缀查找命名空间 URI
- `lookupPrefix(namespaceURI)`：根据命名空间 URI 查找前缀

#### Document 的变化

DOM2 新增与命名空间相关的方法：

- `createElementNS(namespaceURI, tagName)`
- `createAttributeNS(namespaceURI, attributeName)`
- `getElementsByTagNameNS(namespaceURI, tagName)`

#### Element 的变化

DOM2 在 Element 上新增命名空间相关方法：

- `getAttributeNS(namespaceURI, localName)`
- `getAttributeNodeNS(namespaceURI, localName)`
- `getElementsByTagNameNS(namespaceURI, tagName)`
- `hasAttributeNS(namespaceURI, localName)`
- `removeAttributeNS(namespaceURI, localName)`
- `setAttributeNS(namespaceURI, qualifiedName, value)`
- `setAttributeNodeNS(attNode)`

#### NamedNodeMap 的变化

- `getNamedItemNS(namespaceURI, localName)`
- `removeNamedItemNS(namespaceURI, localName)`
- `setNamedItemNS(node)`

### 16.1.2 其他变化

#### DocumentType 的变化

新增属性：

- `publicId`：文档类型声明中的公共标识符
- `systemId`：文档类型声明中的系统标识符
- `internalSubset`：文档类型声明中的内部子集（字符串）

#### Document 的变化

- `importNode(node, deep)`：从其他文档导入节点到当前文档，类似 `cloneNode()`，但导入后归属当前文档。`deep` 为 `true` 表示深复制
- `defaultView`：指向拥有该文档的窗口（或窗格），即 `window` 对象
- DOM3 新增 `document.implementation.hasFeature()` 的替代：`document.implementation.getFeature()`

#### Node 的变化

DOM3 新增方法：

- `isSameNode(otherNode)`：引用同一个对象时返回 `true`
- `isEqualNode(otherNode)`：节点类型、属性及子节点都相等时返回 `true`

```jsx
let div1 = document.createElement("div");
div1.setAttribute("class", "box");

let div2 = document.createElement("div");
div2.setAttribute("class", "box");

console.log(div1.isSameNode(div1));   // true
console.log(div1.isEqualNode(div2));  // true
console.log(div1.isSameNode(div2));   // false
```

- `setUserData(key, value, handler)`：为节点附加额外数据（DOM3）
- `getUserData(key)`：获取附加数据

#### 框架的变化

框架（`<iframe>`）对应的 `HTMLIFrameElement` 新增 `contentDocument` 属性，指向框架内容的 `document` 对象。DOM2 还定义了 `contentWindow` 属性。

---

## 16.2 样式

DOM2 Style 模块定义了以编程方式访问和操控 CSS 样式的 API。

### 16.2.1 存取元素样式

任何支持 `style` 特性的 HTML 元素在 JavaScript 中都有一个对应的 `style` 属性，它是 `CSSStyleDeclaration` 的实例，包含通过 HTML **`style` 特性**（内联样式）指定的所有样式。

**CSS 属性名到 JavaScript 属性名的转换**遵循驼峰命名：

| **CSS 属性** | **JavaScript 属性** |
| --- | --- |
| `background-color` | `style.backgroundColor` |
| `font-size` | `style.fontSize` |
| `border-left-width` | `style.borderLeftWidth` |
| `float` | `style.cssFloat`（保留字，加前缀） |

```jsx
let myDiv = document.getElementById("myDiv");
// 设置样式
myDiv.style.backgroundColor = "red";
myDiv.style.width = "100px";
myDiv.style.border = "1px solid black";
```

<aside>
⚠️

`style` 属性**只包含内联样式**，不反映通过 `<style>` 标签或外部样式表设置的样式。

</aside>

#### DOM 样式属性和方法

`CSSStyleDeclaration` 对象还提供以下属性和方法：

- `cssText`：读写 style 特性中的全部 CSS 代码
- `length`：CSS 属性数量
- `parentRule`：表示 CSS 信息的 `CSSRule` 对象
- `getPropertyValue(propertyName)`：返回属性值的字符串
- `getPropertyPriority(propertyName)`：如果使用了 `!important` 则返回 `"important"`，否则返回空串
- `item(index)`：返回指定索引的属性名
- `removeProperty(propertyName)`：移除指定属性
- `setProperty(propertyName, value, priority)`：设置属性值

```jsx
// 使用 cssText 一次性设置多个样式
myDiv.style.cssText = "width: 25px; height: 100px; background-color: green;";

// 遍历所有内联样式
for (let i = 0, len = myDiv.style.length; i < len; i++) {
  let prop = myDiv.style[i]; // 或 myDiv.style.item(i)
  let value = myDiv.style.getPropertyValue(prop);
  console.log(`${prop}: ${value}`);
}
```

#### 计算样式

`document.defaultView.getComputedStyle(element, pseudoElement)` 返回一个 `CSSStyleDeclaration` 对象，包含**所有计算样式**（内联 + 样式表 + 继承）：

```jsx
let computedStyle = document.defaultView.getComputedStyle(myDiv, null);
console.log(computedStyle.backgroundColor);
console.log(computedStyle.width);
console.log(computedStyle.border);
```

<aside>
💡

计算样式是**只读**的，不能通过 `getComputedStyle()` 修改样式。不同浏览器对默认值的处理可能不同。

</aside>

### 16.2.2 操作样式表

`CSSStyleSheet` 类型表示 CSS 样式表，包括通过 `<link>` 引入的和 `<style>` 定义的。

**`document.styleSheets`** 集合包含文档中所有可用的样式表。

`CSSStyleSheet` 继承自 `StyleSheet`，属性包括：

- `disabled`：布尔值，是否禁用，可读写
- `href`：样式表 URL（`<style>` 定义的为 `null`）
- `media`：支持的媒体类型集合
- `ownerNode`：指向拥有该样式表的 DOM 节点
- `ownerRule`：如果通过 `@import` 导入，指向对应的 `CSSImportRule`
- `parentStyleSheet`：如果通过 `@import` 导入，指向父样式表
- `title`：`ownerNode` 的 `title` 属性值
- `type`：样式表类型，CSS 样式表为 `"text/css"`
- `cssRules`：样式表中 CSS 规则的集合
- `deleteRule(index)`：删除指定位置的规则
- `insertRule(rule, index)`：在指定位置插入规则

#### CSS 规则

`CSSRule` 表示样式表中的一条规则，最常用的是 `CSSStyleRule`：

- `cssText`：整条规则的文本（只读）
- `selectorText`：选择器文本
- `style`：`CSSStyleDeclaration` 对象，可读写

```jsx
let sheet = document.styleSheets[0];
let rules = sheet.cssRules;
let rule = rules[0];

console.log(rule.selectorText);      // 如 "div.box"
console.log(rule.style.cssText);     // 样式内容
console.log(rule.style.backgroundColor);

// 修改规则
rule.style.backgroundColor = "red";
```

#### 创建和删除规则

```jsx
// 插入规则
sheet.insertRule("body { background-color: silver }", 0);

// 删除第一条规则
sheet.deleteRule(0);
```

### 16.2.3 元素尺寸

以下属性并非 DOM2 Style 规范的一部分，但所有主流浏览器均支持。

#### 偏移尺寸（Offset Dimensions）

包含元素在屏幕上占据的所有可视空间（内容 + 内边距 + 滚动条 + 边框）：

- `offsetHeight`：元素总高度（border + padding + content + 水平滚动条）
- `offsetWidth`：元素总宽度
- `offsetTop`：元素上外边框到 `offsetParent` 上内边框的距离
- `offsetLeft`：元素左外边框到 `offsetParent` 左内边框的距离
- `offsetParent`：最近的有定位的祖先元素

<aside>
📌

偏移尺寸属性都是**只读**的，每次访问都会重新计算，频繁访问应缓存为局部变量。

</aside>

#### 客户端尺寸（Client Dimensions）

元素内容区加上内边距的空间（不包括边框和滚动条）：

- `clientWidth`：内容宽度 + 左右内边距
- `clientHeight`：内容高度 + 上下内边距

常见用法——获取浏览器视口大小：

```jsx
let viewportWidth = document.documentElement.clientWidth;
let viewportHeight = document.documentElement.clientHeight;
```

#### 滚动尺寸（Scroll Dimensions）

包含可滚动内容的元素信息：

- `scrollHeight`：没有滚动条时内容的总高度
- `scrollWidth`：没有滚动条时内容的总宽度
- `scrollTop`：内容区被隐藏在上方的像素数（可写）
- `scrollLeft`：内容区被隐藏在左侧的像素数（可写）

```jsx
// 回到顶部
function scrollToTop(element) {
  if (element.scrollTop !== 0) {
    element.scrollTop = 0;
  }
}
```

#### 确定元素尺寸

`getBoundingClientRect()` 返回一个 `DOMRect` 对象，包含元素在页面中相对于**视口**的位置：

- `left`、`top`、`right`、`bottom`
- `width`、`height`

```jsx
let rect = element.getBoundingClientRect();
console.log(rect.left, rect.top, rect.width, rect.height);
```

---

## 16.3 遍历

DOM2 Traversal and Range 模块定义了两个用于辅助**顺序遍历** DOM 的类型：`NodeIterator` 和 `TreeWalker`。它们都从给定起始节点开始，以**深度优先**方式遍历 DOM 树。

### 16.3.1 NodeIterator

通过 `document.createNodeIterator()` 创建，接收 4 个参数：

1. **`root`**：遍历起始节点
2. **`whatToShow`**：位掩码，表示要访问的节点类型
3. **`filter`**：`NodeFilter` 对象或过滤函数
4. **`entityReferenceExpansion`**：是否扩展实体引用（HTML 中无效）

**`whatToShow`** 常用值：

- `NodeFilter.SHOW_ALL`：所有节点
- `NodeFilter.SHOW_ELEMENT`：元素节点
- `NodeFilter.SHOW_TEXT`：文本节点
- `NodeFilter.SHOW_COMMENT`：注释节点

```jsx
// 遍历 <div id="div1"> 下的所有元素
let div = document.getElementById("div1");
let iterator = document.createNodeIterator(
  div,
  NodeFilter.SHOW_ELEMENT,
  null,
  false
);

let node = iterator.nextNode();
while (node !== null) {
  console.log(node.tagName);
  node = iterator.nextNode();
}
```

**使用过滤器**：

```jsx
let filter = function(node) {
  return node.tagName.toLowerCase() === "p"
    ? NodeFilter.FILTER_ACCEPT
    : NodeFilter.FILTER_SKIP;
};

let iterator = document.createNodeIterator(
  div, NodeFilter.SHOW_ELEMENT, filter, false
);
```

主要方法：

- `nextNode()`：向前移动一步，返回下一个节点
- `previousNode()`：向后移动一步，返回上一个节点

### 16.3.2 TreeWalker

`TreeWalker` 是 `NodeIterator` 的高级版本，通过 `document.createTreeWalker()` 创建，参数与 `createNodeIterator()` 相同。

**额外的遍历方法**：

- `parentNode()`：遍历到当前节点的父节点
- `firstChild()`：遍历到第一个子节点
- `lastChild()`：遍历到最后一个子节点
- `nextSibling()`：遍历到下一个同胞节点
- `previousSibling()`：遍历到上一个同胞节点

**过滤器的区别**：对 `TreeWalker` 来说，`NodeFilter.FILTER_REJECT` 会跳过该节点**及其整个子树**，而 `NodeFilter.FILTER_SKIP` 只跳过该节点但仍遍历其子树。

**`currentNode` 属性**：表示遍历过程中上一次返回的节点，可以设置来修改遍历的起点：

```jsx
let walker = document.createTreeWalker(
  div,
  NodeFilter.SHOW_ELEMENT,
  null,
  false
);

walker.firstChild();       // 到 <p>
walker.nextSibling();      // 到 <ul>

let node = walker.firstChild();  // 到第一个 <li>
while (node !== null) {
  console.log(node.tagName);
  node = walker.nextSibling();
}
```

---

## 16.4 范围

DOM2 Traversal and Range 模块定义了 **Range 接口**，可以更精细地控制文档，实现选择文档中某个区域而不用考虑节点边界。

### 16.4.1 DOM 范围

通过 `document.createRange()` 创建 `Range` 实例。

Range 属性：

- `startContainer`：范围起点所在的节点
- `startOffset`：起点在 `startContainer` 中的偏移量
- `endContainer`：范围终点所在的节点
- `endOffset`：终点在 `endContainer` 中的偏移量
- `commonAncestorContainer`：同时包含起点和终点的最深层共同祖先节点

### 16.4.2 简单选择

- `selectNode(node)`：选择整个节点及其后代
- `selectNodeContents(node)`：选择节点的所有后代内容

```jsx
let range1 = document.createRange();
let range2 = document.createRange();
let p1 = document.getElementById("p1");

range1.selectNode(p1);          // 包含 <p> 本身
range2.selectNodeContents(p1);  // 只包含 <p> 的内容
```

更精细的边界设置：

- `setStartBefore(refNode)`
- `setStartAfter(refNode)`
- `setEndBefore(refNode)`
- `setEndAfter(refNode)`

### 16.4.3 复杂选择

使用 `setStart(node, offset)` 和 `setEnd(node, offset)` 设定任意边界：

```jsx
let range = document.createRange();
let p1 = document.getElementById("p1");
let p1Index = -1;
for (let i = 0, len = p1.parentNode.childNodes.length; i < len; i++) {
  if (p1.parentNode.childNodes[i] === p1) {
    p1Index = i;
    break;
  }
}
// 选择从 p1 的第2个字符到第5个字符
let textNode = p1.firstChild;
range.setStart(textNode, 2);
range.setEnd(textNode, 5);
```

### 16.4.4 操作范围内容

创建范围后可以对选区执行操作：

- **`deleteContents()`**：删除范围包含的文档片段
- **`extractContents()`**：删除并返回范围中的文档片段（`DocumentFragment`）
- **`cloneContents()`**：返回范围内容的副本（`DocumentFragment`）

```jsx
// 删除范围内容
range.deleteContents();

// 提取范围内容
let fragment = range.extractContents();
document.body.appendChild(fragment);

// 复制范围内容
let clone = range.cloneContents();
document.body.appendChild(clone);
```

### 16.4.5 范围插入

- **`insertNode(node)`**：在范围的起始位置插入一个节点

```jsx
let span = document.createElement("span");
span.style.color = "red";
span.appendChild(document.createTextNode("Inserted text"));
range.insertNode(span);
```

- **`surroundContents(node)`**：用给定节点包围范围内容

```jsx
let span = document.createElement("span");
span.style.backgroundColor = "yellow";
range.surroundContents(span);
```

### 16.4.6 范围折叠

**折叠（Collapse）** 指范围的起点和终点指向同一位置（即没有选中任何内容）：

```jsx
range.collapse(true);   // 折叠到起点
range.collapse(false);  // 折叠到终点
console.log(range.collapsed);  // true
```

### 16.4.7 范围比较

`compareBoundaryPoints(how, sourceRange)` 用于比较两个范围的边界：

- `Range.START_TO_START`（0）：比较两个范围的起点
- `Range.START_TO_END`（1）：sourceRange 的起点与当前范围的终点比较
- `Range.END_TO_END`（2）：比较两个范围的终点
- `Range.END_TO_START`（3）：sourceRange 的终点与当前范围的起点比较

返回值：`-1`（在前）、`0`（相等）、`1`（在后）

### 16.4.8 复制范围

```jsx
let newRange = range.cloneRange();
```

### 16.4.9 清理

使用完范围后应手动解除引用：

```jsx
range.detach();   // 从文档中分离
range = null;     // 解除引用
```

---

## 16.5 小结

DOM2 和 DOM3 规范对 DOM 进行了重要扩展：

- **DOM2 Core** 引入了 XML 命名空间支持、新增节点操作方法（`importNode()`、`isSameNode()`、`isEqualNode()` 等）
- **DOM2 Style** 提供了以编程方式访问和修改 CSS 样式的能力，包括内联样式、计算样式和样式表操作
- **DOM2 Traversal and Range** 提供了 `NodeIterator`、`TreeWalker` 用于 DOM 遍历，以及 `Range` 用于精细的文档区域操作