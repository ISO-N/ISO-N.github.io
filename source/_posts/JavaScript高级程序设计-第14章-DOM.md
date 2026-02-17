---
title: 第14章 DOM
date: 2026-02-17 15:02:07
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
DOM（文档对象模型）是 HTML 和 XML 文档的编程接口。DOM 表示由多层节点构成的文档，通过它可以添加、删除和修改页面的各个部分。

## 14.1 节点层级

DOM 将任何 HTML 或 XML 文档描绘成一个由多层节点构成的结构。每个节点都有自己的特点、数据和方法，并与其他节点存在某种关系。整个文档就是一个 **文档节点**，是每个文档的根节点。

以下面 HTML 为例：

```html
<html>
  <head>
    <title>Sample Page</title>
  </head>
  <body>
    <p>Hello World!</p>
  </body>
</html>
```

其中 `document` 节点是文档的根节点，`<html>` 元素是文档元素（**documentElement**），也是文档最外层的元素，所有其他元素都存在于这个元素之内。

### 14.1.1 Node 类型

DOM Level 1 定义了一个 `Node` 接口，所有 DOM 节点类型都实现了这个接口。在 JavaScript 中，所有节点类型都继承自 `Node` 类型，因此所有节点共享相同的基本属性和方法。

每个节点都有 `nodeType` 属性，用于表明节点类型。常用节点类型常量如下：

| **常量** | **值** | **说明** |
| --- | --- | --- |
| `Node.ELEMENT_NODE` | 1 | 元素节点 |
| `Node.ATTRIBUTE_NODE` | 2 | 属性节点 |
| `Node.TEXT_NODE` | 3 | 文本节点 |
| `Node.CDATA_SECTION_NODE` | 4 | CDATA 区段 |
| `Node.COMMENT_NODE` | 8 | 注释节点 |
| `Node.DOCUMENT_NODE` | 9 | 文档节点 |
| `Node.DOCUMENT_TYPE_NODE` | 10 | 文档类型节点 |
| `Node.DOCUMENT_FRAGMENT_NODE` | 11 | 文档片段节点 |

#### nodeName 与 nodeValue

对于元素节点，`nodeName` 始终等于元素的标签名，`nodeValue` 始终为 `null`。

```jsx
if (someNode.nodeType == 1) {
  let name = someNode.nodeName;  // 元素的标签名
  let value = someNode.nodeValue; // null
}
```

#### 节点关系

每个节点都有一个 `childNodes` 属性，包含一个 **NodeList** 对象。NodeList 是一个类数组对象，用于存储有序的节点集合，它是**实时的**——DOM 结构的变化会自动反映在 NodeList 中。

```jsx
let firstChild = someNode.childNodes[0];
let secondChild = someNode.childNodes.item(1);
let count = someNode.childNodes.length;

// 转换为真正的数组
let arrayOfNodes = Array.from(someNode.childNodes);
```

其他关系指针：

- `parentNode`：指向父节点
- `previousSibling` / `nextSibling`：前一个 / 后一个同胞节点
- `firstChild` / `lastChild`：第一个 / 最后一个子节点
- `hasChildNodes()`：如果有一个或多个子节点返回 `true`
- `ownerDocument`：指向整个文档的文档节点（`document`）

#### 操纵节点

- **`appendChild(newNode)`**：在 `childNodes` 末尾添加节点。如果该节点已经是文档的一部分，会从原位置移动到新位置。
- **`insertBefore(newNode, referenceNode)`**：在参考节点前插入。如果参考节点为 `null`，则等同于 `appendChild`。
- **`replaceChild(newNode, oldNode)`**：用新节点替换旧节点，旧节点被移除。
- **`removeChild(node)`**：移除节点。

```jsx
// 在末尾添加
someNode.appendChild(newNode);
// 插入到第一个子节点之前
someNode.insertBefore(newNode, someNode.firstChild);
// 替换第一个子节点
someNode.replaceChild(newNode, someNode.firstChild);
// 移除第一个子节点
someNode.removeChild(someNode.firstChild);
```

#### 其他方法

- **`cloneNode(deep)`**：复制节点。参数为 `true` 时深复制（含所有子树），`false` 时只复制节点本身。
    - 克隆不会复制添加到 DOM 节点的 JavaScript 属性（如事件处理程序），只复制 HTML 属性以及可选的子节点。
- **`normalize()`**：处理文档子树中的文本节点——删除空文本节点，合并相邻文本节点。

### 14.1.2 Document 类型

`Document` 类型是 JavaScript 中表示文档节点的类型。在浏览器中，`document` 对象是 `HTMLDocument` 的实例，表示整个 HTML 页面。`document` 也是 `window` 对象的属性，可以作为全局对象访问。

特征：

- `nodeType` 等于 **9**
- `nodeName` 等于 `"#document"`
- `nodeValue` 为 `null`
- `parentNode` 为 `null`
- 子节点可以是 `DocumentType`（最多一个）、`Element`（最多一个）、`Comment` 等

#### 文档子节点

```jsx
let html = document.documentElement; // 取得 <html> 的引用
let body = document.body;            // 取得 <body> 的引用
let doctype = document.doctype;      // 取得 <!DOCTYPE> 的引用
```

#### 文档信息

```jsx
let title = document.title;      // 取得文档标题
document.title = "New Title";    // 设置文档标题

let url = document.URL;          // 完整 URL
let domain = document.domain;    // 域名
let referrer = document.referrer; // 来源页面 URL
```

> `domain` 属性是可以设置的，但有安全限制：只能设置为 URL 中包含的域。例如 `p2p.wrox.com` 可以设置为 `wrox.com`，但不能再设回子域。这一特性在跨子域的 frame/iframe 通信中很有用。
> 

#### 定位元素

```jsx
// 按 ID 获取
let div = document.getElementById("myDiv");

// 按标签名获取（返回 HTMLCollection）
let images = document.getElementsByTagName("img");
images.length;        // 图片数量
images[0].src;        // 第一张图片的 src
images.item(0);       // 同上
images.namedItem("myImage"); // 按 name 属性获取
images["myImage"];    // 简写

// 获取所有元素
let allElements = document.getElementsByTagName("*");

// 按 name 属性获取（常用于单选按钮）
let radios = document.getElementsByName("color");
```

> `HTMLCollection` 和 `NodeList` 类似，也是**实时**的，访问时都会在文档中进行动态查询。
> 

#### 特殊集合

`document` 对象上有几个特殊集合，也都是 `HTMLCollection` 的实例：

- `document.anchors`：所有带 `name` 属性的 `<a>` 元素
- `document.forms`：所有 `<form>` 元素（同 `document.getElementsByTagName("form")`）
- `document.images`：所有 `<img>` 元素
- `document.links`：所有带 `href` 属性的 `<a>` 元素

#### 文档写入

```jsx
document.write("<strong>Hello</strong>");    // 写入文本
document.writeln("<strong>Hello</strong>");  // 写入文本并换行
document.open();   // 打开文档输出流
document.close();  // 关闭文档输出流
```

<aside>
⚠️

在页面加载完成后调用 `document.write()` 会重写整个页面。此外，在严格的 XHTML 文档中不能使用 `document.write()`。

</aside>

### 14.1.3 Element 类型

`Element` 类型是除 `Document` 类型之外最常用的类型。

特征：

- `nodeType` 等于 **1**
- `nodeName` 等于元素的标签名
- `nodeValue` 为 `null`
- `parentNode` 可能是 `Document` 或 `Element`

```jsx
let div = document.getElementById("myDiv");
div.tagName;  // "DIV"（HTML 中始终全大写）
div.nodeName; // "DIV"（同 tagName）

// 推荐比较时转为小写
if (element.tagName.toLowerCase() == "div") { ... }
```

#### HTML 元素

所有 HTML 元素都是 `HTMLElement` 或其子类的实例。`HTMLElement` 直接继承 `Element` 并增加了下列标准属性：

- `id`：元素在文档中的唯一标识符
- `title`：元素的额外信息（通常作为提示条显示）
- `lang`：元素内容的语言代码
- `dir`：语言的书写方向（`"ltr"` 或 `"rtl"`）
- `className`：与 `class` 属性对应（因为 `class` 是保留字）

```jsx
let div = document.getElementById("myDiv");
div.id;         // "myDiv"
div.className;  // "bd"
div.title;      // "Body text"
div.lang;       // "en"
div.dir;        // "ltr"
```

#### 属性操作

每个元素都有零或多个属性。DOM 提供了三个方法来操纵属性：

```jsx
// 获取属性
let value = div.getAttribute("id");
let cls = div.getAttribute("class"); // 注意是 "class" 不是 "className"

// 设置属性
div.setAttribute("id", "someOtherId");
div.setAttribute("class", "ft");

// 删除属性
div.removeAttribute("class");
```

<aside>
💡

`getAttribute()` 通常用于获取**自定义属性**（如 `data-*`）。对于 `style` 属性，`getAttribute()` 返回 CSS 字符串，而 DOM 属性返回 `CSSStyleDeclaration` 对象；对于事件处理程序（如 `onclick`），`getAttribute()` 返回字符串，而 DOM 属性返回函数。因此实际开发中**更推荐通过 DOM 属性**来读写标准属性。

</aside>

#### attributes 属性

`Element` 类型是唯一使用 `attributes` 属性的 DOM 节点类型。`attributes` 包含一个 `NamedNodeMap`，其中每个属性都以 `Attr` 节点的形式存在。

```jsx
// 获取属性
let id = element.attributes.getNamedItem("id").nodeValue;
let id2 = element.attributes["id"].nodeValue; // 简写

// 设置属性
element.attributes["id"].nodeValue = "someOtherId";

// 删除属性
let oldAttr = element.attributes.removeNamedItem("id");

// 遍历所有属性
for (let i = 0, len = element.attributes.length; i < len; ++i) {
  let attr = element.attributes[i];
  console.log(`${attr.nodeName}="${attr.nodeValue}"`);
}
```

#### 创建元素

```jsx
let div = document.createElement("div");
div.id = "myNewDiv";
div.className = "box";

// 添加到文档树中（此时元素才会被渲染）
document.body.appendChild(div);
```

#### 元素的子节点

元素可以包含任意多个子节点和后代节点。`childNodes` 包含所有子节点，包括元素、文本（空白）和注释节点。

```jsx
// 遍历子元素时，应过滤 nodeType
for (let i = 0, len = element.childNodes.length; i < len; ++i) {
  if (element.childNodes[i].nodeType == 1) {
    // 处理元素节点
  }
}
```

### 14.1.4 Text 类型

`Text` 节点包含按字面解释的纯文本，也可能包含转义后的 HTML 字符（但不能包含 HTML 代码）。

特征：

- `nodeType` 等于 **3**
- `nodeName` 等于 `"#text"`
- `nodeValue` 等于节点中包含的文本
- `parentNode` 是一个 `Element`
- 不支持子节点

操纵文本的方法：

| **方法** | **说明** |
| --- | --- |
| `appendData(text)` | 在末尾追加文本 |
| `deleteData(offset, count)` | 从 offset 位置删除 count 个字符 |
| `insertData(offset, text)` | 在 offset 位置插入文本 |
| `replaceData(offset, count, text)` | 替换从 offset 到 offset+count 的文本 |
| `splitText(offset)` | 在 offset 处拆分为两个文本节点 |
| `substringData(offset, count)` | 提取从 offset 到 offset+count 的文本 |

```jsx
// 创建文本节点
let textNode = document.createTextNode("Hello World!");
let anotherTextNode = document.createTextNode("Yippee!");

let element = document.createElement("div");
element.appendChild(textNode);
element.appendChild(anotherTextNode);
document.body.appendChild(element);

// 此时 element 有两个文本子节点
element.normalize(); // 合并为一个文本节点
console.log(element.childNodes.length); // 1

// 拆分文本节点
let newNode = element.firstChild.splitText(5);
console.log(element.firstChild.nodeValue); // "Hello"
console.log(newNode.nodeValue);            // " World!Yippee!"
```

### 14.1.5 Comment 类型

注释在 DOM 中以 `Comment` 类型表示。

特征：

- `nodeType` 等于 **8**
- `nodeName` 等于 `"#comment"`
- `nodeValue` 等于注释的内容

```jsx
// <div id="myDiv"><!-- A comment --></div>
let div = document.getElementById("myDiv");
let comment = div.firstChild;
console.log(comment.data); // " A comment "
```

`Comment` 类型与 `Text` 类型继承同一个基类（`CharacterData`），所以拥有除 `splitText()` 以外的所有字符串操作方法。

### 14.1.6 CDATASection 类型

`CDATASection` 类型只在 XML 文档中有效，表示 CDATA 区段。

- `nodeType` 等于 **4**
- `nodeName` 等于 `"#cdata-section"`

### 14.1.7 DocumentType 类型

`DocumentType` 对象包含文档的 doctype 信息。

- `nodeType` 等于 **10**
- `nodeName` 等于 doctype 的名称

```jsx
console.log(document.doctype.name); // "html"
```

### 14.1.8 DocumentFragment 类型

`DocumentFragment` 是一种"轻量级"文档，能够包含和操作节点，但本身不会被添加到文档树中。它常用来作为临时仓库，批量操作节点后一次性添加到文档，从而**减少浏览器重排和重绘次数**。

- `nodeType` 等于 **11**
- `nodeName` 等于 `"#document-fragment"`

```jsx
let fragment = document.createDocumentFragment();
let ul = document.getElementById("myList");

for (let i = 0; i < 3; ++i) {
  let li = document.createElement("li");
  li.appendChild(document.createTextNode(`Item ${i + 1}`));
  fragment.appendChild(li);
}

// 一次性添加，只触发一次重排
ul.appendChild(fragment);
```

<aside>
✅

**性能优化**：当需要向文档中添加大量节点时，先将节点添加到 `DocumentFragment`，然后再一次性把 `DocumentFragment` 添加到文档，可以避免多次重排。

</aside>

### 14.1.9 Attr 类型

元素的属性在 DOM 中以 `Attr` 类型来表示。

- `nodeType` 等于 **2**
- `nodeName` 等于属性名
- `nodeValue` 等于属性值

`Attr` 对象有三个属性：`name`（属性名）、`value`（属性值）和 `specified`（布尔值，区分是在代码中指定的还是默认值）。

```jsx
let attr = document.createAttribute("align");
attr.value = "left";
element.setAttributeNode(attr);

console.log(element.attributes["align"].value);    // "left"
console.log(element.getAttributeNode("align").value); // "left"
console.log(element.getAttribute("align"));           // "left"
```

> 实际开发中推荐使用 `getAttribute()`、`setAttribute()` 和 `removeAttribute()` 方法，而不是直接操作 `Attr` 对象。
> 

---

## 14.2 DOM 编程

### 14.2.1 动态脚本

动态脚本指的是在页面初始加载时不存在、之后通过 DOM 动态添加的脚本。

```jsx
// 方式一：引入外部脚本文件
function loadScript(url) {
  let script = document.createElement("script");
  script.type = "text/javascript";
  script.src = url;
  document.body.appendChild(script);
}
loadScript("client.js");

// 方式二：内联脚本
function loadScriptString(code) {
  let script = document.createElement("script");
  script.type = "text/javascript";
  try {
    script.appendChild(document.createTextNode(code));
  } catch (ex) {
    // IE 不允许常规 DOM 操作修改 <script> 元素
    script.text = code;
  }
  document.body.appendChild(script);
}
loadScriptString("function sayHi(){alert('hi');}");
```

### 14.2.2 动态样式

与动态脚本类似，动态样式是在页面加载后动态添加的样式。

```jsx
// 方式一：<link> 引入外部样式表
function loadStyles(url) {
  let link = document.createElement("link");
  link.rel = "stylesheet";
  link.type = "text/css";
  link.href = url;
  let head = document.getElementsByTagName("head")[0];
  head.appendChild(link);
}
loadStyles("styles.css");

// 方式二：<style> 嵌入样式
function loadStyleString(css) {
  let style = document.createElement("style");
  style.type = "text/css";
  try {
    style.appendChild(document.createTextNode(css));
  } catch (ex) {
    // IE 兼容
    style.styleSheet.cssText = css;
  }
  let head = document.getElementsByTagName("head")[0];
  head.appendChild(style);
}
loadStyleString("body{background-color:red}");
```

### 14.2.3 操纵表格

DOM 为 `<table>`、`<tbody>` 和 `<tr>` 提供了便利的属性和方法，简化了表格的创建和修改。

#### `<table>` 的属性和方法

- `caption`：指向 `<caption>` 元素的指针
- `tBodies`：`<tbody>` 元素的 HTMLCollection
- `tFoot`：指向 `<tfoot>` 元素的指针
- `tHead`：指向 `<thead>` 元素的指针
- `rows`：所有行的 HTMLCollection
- `createTHead()`、`createTFoot()`、`createCaption()`
- `deleteTHead()`、`deleteTFoot()`、`deleteCaption()`
- `deleteRow(pos)`、`insertRow(pos)`

#### `<tbody>` 的属性和方法

- `rows`：`<tbody>` 中行的 HTMLCollection
- `deleteRow(pos)`
- `insertRow(pos)`：在指定位置插入行，返回该行的引用

#### `<tr>` 的属性和方法

- `cells`：`<tr>` 中单元格的 HTMLCollection
- `deleteCell(pos)`
- `insertCell(pos)`：在指定位置插入单元格，返回该单元格的引用

```jsx
// 使用 DOM 便利方法创建表格
let table = document.createElement("table");
table.border = 1;
table.width = "100%";

let tbody = document.createElement("tbody");
table.appendChild(tbody);

// 创建第一行
tbody.insertRow(0);
tbody.rows[0].insertCell(0);
tbody.rows[0].cells[0].appendChild(document.createTextNode("Cell 1,1"));
tbody.rows[0].insertCell(1);
tbody.rows[0].cells[1].appendChild(document.createTextNode("Cell 2,1"));

// 创建第二行
tbody.insertRow(1);
tbody.rows[1].insertCell(0);
tbody.rows[1].cells[0].appendChild(document.createTextNode("Cell 1,2"));
tbody.rows[1].insertCell(1);
tbody.rows[1].cells[1].appendChild(document.createTextNode("Cell 2,2"));

document.body.appendChild(table);
```

### 14.2.4 使用 NodeList

理解 `NodeList`、`NamedNodeMap` 和 `HTMLCollection` 这三个集合类型是理解 DOM 的关键。这三个集合都是**"实时的"**，文档结构的变化会实时反映到它们中。

<aside>
🚨

**经典陷阱**：在迭代 NodeList 时修改 DOM 会导致无限循环：

```jsx
// ❌ 无限循环！
let divs = document.getElementsByTagName("div");
for (let i = 0; i < divs.length; ++i) {
  let div = document.createElement("div");
  document.body.appendChild(div);
}
```

每次循环都会添加新 `<div>`，`divs.length` 也随之增长，导致永远无法退出。

```jsx
// ✅ 解决方案：缓存 length 值
let divs = document.getElementsByTagName("div");
for (let i = 0, len = divs.length; i < len; ++i) {
  let div = document.createElement("div");
  document.body.appendChild(div);
}
```

</aside>

> 一般来说，最好限制操作 NodeList 的次数。因为每次查询都会搜索整个文档，所以最好把查询到的 NodeList 缓存起来。
> 

---

## 14.3 MutationObserver 接口

`MutationObserver` 接口可以在 DOM 被修改时异步执行回调。它取代了早期的 Mutation Events，并且可以监视整个文档、DOM 树的一部分，或者某个元素的特定属性、子节点、文本等变化。

### 14.3.1 基本用法

```jsx
// 创建 MutationObserver 实例
let observer = new MutationObserver(
  (mutationRecords) => console.log(mutationRecords)
);
```

#### observe() 方法

```jsx
observer.observe(document.body, {
  attributes: true,    // 观察属性变化
  childList: true,     // 观察子节点变化
  subtree: true        // 观察后代节点变化
});
```

`MutationObserverInit` 对象常用配置：

| **属性** | **说明** |
| --- | --- |
| `attributes` | 设为 `true` 表示观察元素属性变化 |
| `attributeFilter` | 字符串数组，指定要观察的属性名 |
| `attributeOldValue` | 设为 `true` 表示记录属性变化前的旧值 |
| `characterData` | 设为 `true` 表示观察文本内容变化 |
| `characterDataOldValue` | 设为 `true` 表示记录文本变化前的旧值 |
| `childList` | 设为 `true` 表示观察子节点增删 |
| `subtree` | 设为 `true` 表示扩展到整棵子树 |

#### 回调与 MutationRecord

每次 DOM 变化都会创建一个 `MutationRecord` 实例，包含关键信息：

- `type`：变化类型（`"attributes"` / `"characterData"` / `"childList"`）
- `target`：被修改的节点
- `addedNodes` / `removedNodes`：新增 / 移除的节点的 NodeList
- `previousSibling` / `nextSibling`：添加或删除的节点的前后同胞
- `attributeName`：变化的属性名
- `oldValue`：变化前的值（需配置相应的 `Old Value` 选项）

#### disconnect() 方法

```jsx
// 停止观察
observer.disconnect();
```

调用 `disconnect()` 后，不仅停止监听，已有但尚未投递的变更记录也会被丢弃。如果想在停止前处理残余记录，可以先调用 `takeRecords()`。

#### takeRecords() 方法

```jsx
// 获取并清空已排队但未投递的 MutationRecord
let records = observer.takeRecords();
```

### 14.3.2 异步回调与记录队列

`MutationObserver` 的回调是以**微任务**（microtask）的形式异步执行的。每次 DOM 变化都会向记录队列中添加一条 `MutationRecord`，当前同步代码执行完毕后，才会一次性将队列中所有记录传给回调函数。

```jsx
let observer = new MutationObserver(
  (records) => console.log("变化数量:", records.length)
);
observer.observe(document.body, { attributes: true });

document.body.className = "foo";
document.body.className = "bar";
document.body.className = "baz";

// 输出："变化数量: 3"
// 三次修改在回调中作为一个批次一起投递
```

### 14.3.3 性能、内存与垃圾回收

<aside>
💡

**使用建议**：

- `MutationObserver` 引用的是目标节点的**弱引用**，不会阻止目标节点被垃圾回收
- 但目标节点对 `MutationObserver` 是**强引用**，如果目标被回收，关联的 `MutationObserver` 也会被回收
- `MutationRecord` 中的 `target` 和节点引用是**强引用**，可能会阻碍垃圾回收。如果需要长时间保存变化记录，建议保存必要信息后尽早释放 `MutationRecord` 的引用
</aside>