---
title: 第19章 表单脚本
date: 2026-02-17 15:02:11
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 19.1 表单基础

Web 表单在 HTML 中以 `<form>` 元素表示，在 JavaScript 中以 `HTMLFormElement` 类型表示。

### 19.1.1 获取表单引用

```jsx
// 1. 通过 getElementById
let form = document.getElementById("myForm");

// 2. 通过 document.forms 集合
let firstForm = document.forms[0];       // 取得页面中第一个表单
let myForm = document.forms["form2"];     // 取得名字为 "form2" 的表单
```

### 19.1.2 HTMLFormElement 的属性和方法

| **属性 / 方法** | **说明** |
| --- | --- |
| `acceptCharset` | 服务器可以接收的字符集，等价于 HTML 的 `accept-charset` 属性 |
| `action` | 请求的 URL，等价于 HTML 的 `action` 属性 |
| `elements` | 表单中所有控件的 `HTMLCollection` |
| `enctype` | 请求的编码类型，等价于 HTML 的 `enctype` 属性 |
| `length` | 表单中控件的数量 |
| `method` | HTTP 请求的方法类型，通常是 `"get"` 或 `"post"` |
| `name` | 表单的名字 |
| `reset()` | 把表单字段重置为各自的默认值 |
| `submit()` | 提交表单 |
| `target` | 用于发送请求和接收响应的窗口名字 |

### 19.1.3 提交表单

```html
<!-- 通用提交按钮 -->
<input type="submit" value="Submit Form">

<!-- 自定义提交按钮 -->
<button type="submit">Submit Form</button>

<!-- 图片按钮 -->
<input type="image" src="graphic.gif">
```

以这种方式提交表单时，会在请求发送到服务器之前触发 **`submit` 事件**，可以在此时验证表单数据并决定是否允许提交：

```jsx
let form = document.getElementById("myForm");
form.addEventListener("submit", (event) => {
  // 验证表单数据...
  event.preventDefault(); // 阻止表单提交
});
```

也可以通过 JavaScript 直接调用 `submit()` 方法提交表单（**不会触发 submit 事件**）：

```jsx
form.submit();
```

<aside>
⚠️

**防止重复提交**：表单提交的一个常见问题是用户多次点击提交按钮。解决方案：

1. 在第一次提交后**禁用提交按钮**
2. 利用 `submit` 事件处理程序**取消之后的表单提交**
</aside>

### 19.1.4 重置表单

```html
<!-- 通用重置按钮 -->
<input type="reset" value="Reset Form">

<!-- 自定义重置按钮 -->
<button type="reset">Reset Form</button>
```

重置按钮会触发 **`reset` 事件**，可以取消重置操作：

```jsx
form.addEventListener("reset", (event) => {
  event.preventDefault(); // 阻止表单重置
});
```

也可以通过 JavaScript 调用 `reset()` 方法（**会触发 reset 事件**，与 `submit()` 不同）。

### 19.1.5 表单字段

通过 `form.elements` 集合访问表单中的所有字段元素：

```jsx
let form = document.getElementById("form1");

let field1 = form.elements[0];          // 第一个字段
let field2 = form.elements["textbox1"]; // 名为 "textbox1" 的字段
let fieldCount = form.elements.length;  // 字段数量
```

<aside>
💡

如果多个表单控件使用了同一个 `name`（如单选按钮），则 `form.elements[name]` 会返回一个 **NodeList**。

</aside>

### 表单字段的公共属性

- `disabled` — 布尔值，表示表单字段是否禁用
- `form` — 指向表单字段所属的表单的指针（只读）
- `name` — 字段的名字
- `readOnly` — 布尔值，表示字段是否只读
- `tabIndex` — Tab 序号
- `type` — 字段类型，如 `"checkbox"`、`"radio"` 等
- `value` — 要提交给服务器的字段值

```jsx
// 避免多次提交表单：在 submit 事件中禁用提交按钮
form.addEventListener("submit", (event) => {
  let target = event.target;
  let btn = target.elements["submit-btn"];
  btn.disabled = true;
});
```

### 表单字段的公共方法

- `focus()` — 将浏览器焦点设置到该字段
- `blur()` — 从该字段移除焦点

```jsx
// 页面加载后自动聚焦第一个字段
window.addEventListener("load", () => {
  document.forms[0].elements[0].focus();
});
```

> HTML5 新增了 `autofocus` 属性，可以自动将焦点移到对应的字段。
> 

### 表单字段的公共事件

- **`blur`** — 在字段失去焦点时触发
- **`change`** — 对于 `<input>` 和 `<textarea>`，在失去焦点且 value 改变时触发；对于 `<select>`，在选中项改变时触发
- **`focus`** — 在字段获得焦点时触发

---

## 19.2 文本框编程

HTML 中有两种文本框：

1. **单行文本框** `<input type="text">`：`size` 属性指定显示宽度（字符数），`maxlength` 限定最大字符数
2. **多行文本框** `<textarea>`：`rows` 和 `cols` 属性指定大小

```html
<!-- 显示25个字符宽，最多允许输入50个字符 -->
<input type="text" size="25" maxlength="50" value="initial value">

<!-- 25行 x 5列 的多行文本框 -->
<textarea rows="25" cols="5">initial value</textarea>
```

<aside>
💡

建议使用 `value` 属性而非 DOM 方法来读写文本框的值，不要使用 `setAttribute()` 设置 `value`，也不要修改 `<textarea>` 的第一个子节点。

</aside>

### 19.2.1 选择文本

`select()` 方法用于选中文本框中的全部文本，大多数浏览器会在调用时自动将焦点设置到文本框：

```jsx
let textbox = document.forms[0].elements["textbox1"];
textbox.select();
```

#### select 事件

当用户选中文本（或调用 `select()` 方法）时，会触发 **`select`** 事件。

#### 取得选中的文本

HTML5 扩展了两个属性：`selectionStart` 和 `selectionEnd`：

```jsx
function getSelectedText(textbox) {
  return textbox.value.substring(textbox.selectionStart, textbox.selectionEnd);
}
```

#### 部分选中文本

`setSelectionRange(startIndex, endIndex)` 方法可以选中文本框中的部分文本：

```jsx
textbox.value = "Hello world!";

textbox.setSelectionRange(0, textbox.value.length); // "Hello world!"
textbox.setSelectionRange(0, 3);                     // "Hel"
textbox.setSelectionRange(4, 7);                     // "o w"
```

### 19.2.2 输入过滤

#### 屏蔽字符

通过监听 `keypress` 事件并阻止默认行为，可以屏蔽某些字符的输入：

```jsx
// 只允许输入数字
textbox.addEventListener("keypress", (event) => {
  if (!/\d/.test(String.fromCharCode(event.charCode))
      && event.charCode > 9  // 不屏蔽方向键等功能键
      && !event.ctrlKey) {   // 不屏蔽 Ctrl 组合键（如 Ctrl+C）
    event.preventDefault();
  }
});
```

#### 处理剪贴板

剪贴板相关事件：

| **事件** | **触发时机** |
| --- | --- |
| `beforecopy` | 复制操作发生前 |
| `copy` | 复制操作发生时 |
| `beforecut` | 剪切操作发生前 |
| `cut` | 剪切操作发生时 |
| `beforepaste` | 粘贴操作发生前 |
| `paste` | 粘贴操作发生时 |

通过 `event.clipboardData`（IE 中为 `window.clipboardData`）对象访问剪贴板数据：

```jsx
// 只允许粘贴数字
textbox.addEventListener("paste", (event) => {
  let text = event.clipboardData.getData("text/plain");
  if (!/^\d*$/.test(text)) {
    event.preventDefault();
  }
});
```

### 19.2.3 自动切换

当一个文本框字符数达到最大长度时，自动将焦点切换到下一个文本框（常用于电话号码、验证码等分段输入）：

```jsx
function tabForward(event) {
  let target = event.target;
  if (target.value.length === target.maxLength) {
    let form = target.form;
    for (let i = 0, len = form.elements.length; i < len; i++) {
      if (form.elements[i] === target) {
        if (form.elements[i + 1]) {
          form.elements[i + 1].focus();
        }
        return;
      }
    }
  }
}
// 为每个输入框添加 keyup 事件监听
```

### 19.2.4 HTML5 约束验证 API

HTML5 为表单提供了原生的客户端验证能力。

#### 必填字段

```html
<input type="text" name="username" required>
```

```jsx
// 检测浏览器是否支持 required 属性
let isRequiredSupported = "required" in document.createElement("input");
```

#### 更多输入类型

HTML5 新增的 `type` 值：`"email"` 和 `"url"` 等，浏览器会自动进行格式验证。

```html
<input type="email" name="email">
<input type="url" name="homepage">
```

#### 数值范围

`"number"`、`"range"`、`"datetime"`、`"datetime-local"`、`"date"`、`"month"`、`"week"`、`"time"` 类型都可以指定 `min`、`max`、`step`：

```html
<input type="number" min="0" max="100" step="5" name="count">
```

#### 输入模式

`pattern` 属性用于指定一个正则表达式，输入值必须匹配该模式：

```html
<!-- 只允许输入数字 -->
<input type="text" pattern="\d+" name="count">
```

#### 检测有效性

`checkValidity()` 方法可以检测表单字段是否有效：

```jsx
if (document.forms[0].elements[0].checkValidity()) {
  // 字段有效，继续处理
} else {
  // 字段无效
}

// 也可以在表单级别检测
if (document.forms[0].checkValidity()) {
  // 所有字段都有效
}
```

`validity` 属性是一个对象，包含一系列布尔属性，详细说明字段为何有效或无效：

- `customError` — 是否设置了自定义错误
- `patternMismatch` — 值是否与 `pattern` 不匹配
- `rangeOverflow` — 值是否大于 `max`
- `rangeUnderflow` — 值是否小于 `min`
- `stepMisalignment` — 值是否不符合 `step` 的规则
- `tooLong` — 值是否超过 `maxlength`
- `typeMismatch` — 值是否不是 `"email"` 或 `"url"` 要求的格式
- `valid` — 是否有效（其他所有属性都为 `false` 时为 `true`）
- `valueMissing` — 字段为 `required` 但没有值

#### 禁用验证

```html
<!-- 在表单级别禁用验证 -->
<form method="post" action="/signup" novalidate>
  <!-- 表单内容 -->
</form>

<!-- 指定某个提交按钮不验证 -->
<input type="submit" formnovalidate name="btnNoValidate" value="Non-validating Submit">
```

```jsx
// 通过 JavaScript 设置
document.forms[0].noValidate = true;
```

---

## 19.3 选择框编程

选择框由 `<select>` 和 `<option>` 元素创建，在 JavaScript 中用 `HTMLSelectElement` 类型表示。

### HTMLSelectElement 的特殊属性和方法

| **属性 / 方法** | **说明** |
| --- | --- |
| `add(newOption, relOption)` | 在 `relOption` 之前向控件添加新的 `<option>` |
| `multiple` | 布尔值，是否允许多选（等价于 HTML 的 `multiple` 属性） |
| `options` | 控件中所有 `<option>` 元素的 HTMLCollection |
| `remove(index)` | 移除给定位置的选项 |
| `selectedIndex` | 基于 0 的选中项索引（没有选中项时为 -1）；多选时只保存第一个选中项的索引 |
| `size` | 选择框中可见的行数（等价于 HTML 的 `size` 属性） |

每个 `<option>` 元素在 JavaScript 中用 `HTMLOptionElement` 表示，拥有以下属性：

- `index` — 选项在 `options` 集合中的索引
- `label` — 选项的标签
- `selected` — 布尔值，是否选中
- `text` — 选项的文本
- `value` — 选项的值

```jsx
// 推荐的访问方式（不使用 DOM）
let selectbox = document.forms[0].elements["location"];
let text = selectbox.options[0].text;   // 选项文本
let value = selectbox.options[0].value; // 选项值
```

### 19.3.1 选项处理

**单选**选择框中，获取选中项最简单的方式是使用 `selectedIndex`：

```jsx
let selectedOption = selectbox.options[selectbox.selectedIndex];
console.log(`Selected index: ${selectbox.selectedIndex}`);
console.log(`Selected text: ${selectedOption.text}`);
console.log(`Selected value: ${selectedOption.value}`);
```

**多选**选择框中，需要遍历 `options` 集合检查每个选项的 `selected` 属性：

```jsx
function getSelectedOptions(selectbox) {
  let result = [];
  for (let option of selectbox.options) {
    if (option.selected) {
      result.push(option);
    }
  }
  return result;
}
```

### 19.3.2 添加选项

```jsx
// 方式1：使用 DOM 方法
let newOption = document.createElement("option");
newOption.appendChild(document.createTextNode("Option text"));
newOption.setAttribute("value", "Option value");
selectbox.appendChild(newOption);

// 方式2：使用 Option 构造函数
let newOption2 = new Option("Option text", "Option value");
selectbox.appendChild(newOption2);

// 方式3：使用 add() 方法（添加到末尾传 undefined）
let newOption3 = new Option("Option text", "Option value");
selectbox.add(newOption3, undefined);
```

### 19.3.3 移除选项

```jsx
// 方式1：DOM 的 removeChild()
selectbox.removeChild(selectbox.options[0]);

// 方式2：使用 remove() 方法
selectbox.remove(0);

// 方式3：设置为 null
selectbox.options[0] = null;
```

<aside>
💡

要清除选择框的所有选项，可以从后往前迭代移除：

```
function clearSelectbox(selectbox) {
  for (let i = selectbox.options.length - 1; i >= 0; i--) {
    selectbox.remove(i);
  }
}
```

</aside>

### 19.3.4 移动和重排选项

使用 DOM 的 `appendChild()` 方法可以将一个选择框中的选项移动到另一个选择框（会自动从原位置移除）：

```jsx
// 将第一个选择框的第一个选项移动到第二个选择框
let selectbox1 = document.getElementById("selLocations1");
let selectbox2 = document.getElementById("selLocations2");
selectbox2.appendChild(selectbox1.options[0]);
```

使用 `insertBefore()` 方法可以重排选项的顺序：

```jsx
// 将选项在选择框中向前移动一个位置
let optionToMove = selectbox.options[1];
selectbox.insertBefore(optionToMove, selectbox.options[optionToMove.index - 1]);
```

---

## 19.4 表单序列化

在提交表单之前，浏览器会按如下规则对表单数据进行序列化：

1. 字段名和值使用 URL 编码（`encodeURIComponent()`），并以 `&` 分隔
2. 禁用的字段不会发送
3. 只发送勾选的复选框和单选按钮的值
4. 类型为 `"reset"` 和 `"button"` 的按钮不会发送
5. 多选框中每个选中值单独一个条目
6. 点击提交按钮时，该提交按钮也会被发送；通过 `submit()` 方法提交时则不包含
7. `<select>` 元素的值就是选中 `<option>` 的 `value` 属性值（没有 `value` 则是 `text`）

```jsx
function serialize(form) {
  let parts = [];
  for (let field of form.elements) {
    switch (field.type) {
      case "select-one":
      case "select-multiple":
        if (field.name.length) {
          for (let option of field.options) {
            if (option.selected) {
              let value = option.value || option.text;
              parts.push(
                `${encodeURIComponent(field.name)}=${encodeURIComponent(value)}`
              );
            }
          }
        }
        break;
      case undefined:  // 字段集
      case "file":     // 文件输入
      case "submit":   // 提交按钮
      case "reset":    // 重置按钮
      case "button":   // 自定义按钮
        break;
      case "radio":
      case "checkbox":
        if (!field.checked) {
          break;
        }
        // 注意：这里 fall through
      default:
        if (field.name.length) {
          parts.push(
            `${encodeURIComponent(field.name)}=${encodeURIComponent(field.value)}`
          );
        }
    }
  }
  return parts.join("&");
}
```

---

## 19.5 富文本编辑

富文本编辑（又称 "WYSIWYG"——What You See Is What You Get）的基本技术就是在页面中嵌入一个空白 HTML 页面，通过设置 `designMode` 或使用 `contenteditable` 属性使其可编辑。

### 19.5.1 使用 contenteditable 属性

```html
<div class="editable" id="richedit" contenteditable></div>
```

`contenteditable` 属性有三个可能的值：

- `"true"` — 开启编辑
- `"false"` — 关闭编辑
- `"inherit"` — 继承父元素的设置

```jsx
let div = document.getElementById("richedit");
div.contentEditable = "true";
```

### 19.5.2 与富文本交互

使用 `document.execCommand()` 方法与富文本编辑器交互，可以执行预定义的命令来操纵富文本内容：

| **命令** | **值** | **说明** |
| --- | --- | --- |
| `bold` | null | 切换选中文本的粗体 |
| `copy` | null | 将选中文本复制到剪贴板 |
| `createlink` | URL 字符串 | 将选中文本转换为指向 URL 的链接 |
| `cut` | null | 将选中文本剪切到剪贴板 |
| `delete` | null | 删除选中文本 |
| `fontname` | 字体名 | 将选中文本改为指定字体 |
| `fontsize` | 1~7 | 将选中文本改为指定大小 |
| `forecolor` | 颜色字符串 | 将选中文本改为指定颜色 |
| `formatblock` | HTML 标签名 | 将选中文本格式化为指定的块级元素 |
| `indent` | null | 缩进文本 |
| `inserthorizontalrule` | null | 在光标位置插入 `<hr>` 元素 |
| `insertimage` | 图片 URL | 在光标位置插入图片 |
| `insertorderedlist` | null | 在光标位置插入有序列表 |
| `insertunorderedlist` | null | 在光标位置插入无序列表 |
| `italic` | null | 切换选中文本的斜体 |
| `justifycenter` | null | 居中对齐 |
| `outdent` | null | 减少缩进 |
| `paste` | null | 将剪贴板内容粘贴到选中文本 |
| `removeformat` | null | 移除选中文本的块级格式 |
| `selectall` | null | 选中所有文本 |
| `underline` | null | 切换选中文本的下划线 |
| `unlink` | null | 移除选中文本的链接 |

```jsx
// 切换粗体
document.execCommand("bold", false, null);

// 切换斜体
document.execCommand("italic", false, null);

// 创建链接
document.execCommand("createlink", false, "http://www.example.com");

// 改变字体大小
document.execCommand("fontsize", false, 3);
```

### 19.5.3 富文本选区

使用 `window.getSelection()` 方法可以获取用户在富文本编辑器中的选区：

```jsx
let selection = window.getSelection();
```

`Selection` 对象的主要属性：

- `anchorNode` — 选区开始的节点
- `anchorOffset` — 选区在 `anchorNode` 中跳过的字符数
- `focusNode` — 选区结束的节点
- `focusOffset` — `focusNode` 中包含在选区内的字符数
- `isCollapsed` — 选区起点和终点是否重合（光标状态）
- `rangeCount` — 选区中包含的 DOM 范围数量

`Selection` 对象的常用方法：

- `addRange(range)` — 把一个 DOM 范围添加到选区
- `collapse(node, offset)` — 将选区折叠到指定节点的指定偏移
- `collapseToEnd()` — 折叠到终点
- `collapseToStart()` — 折叠到起点
- `containsNode(node)` — 确定指定节点是否在选区中
- `deleteFromDocument()` — 从文档中删除选区文本
- `getRangeAt(index)` — 返回索引对应的 DOM 范围
- `removeAllRanges()` — 移除所有范围，相当于取消选区
- `selectAllChildren(node)` — 选中指定节点的所有子节点
- `toString()` — 返回选区中的文本

```jsx
let selection = window.getSelection();

// 取得选中的文本
let selectedText = selection.toString();

// 取得代表选区的范围
let range = selection.getRangeAt(0);

// 高亮选中文本
let span = document.createElement("span");
span.style.backgroundColor = "yellow";
range.surroundContents(span);
```

### 19.5.4 通过表单提交富文本

富文本编辑器的内容并不会随表单自动提交。通常在表单的 `submit` 事件中，把富文本内容提取出来放到一个隐藏字段中：

```jsx
form.addEventListener("submit", (event) => {
  let target = event.target;
  target.elements["comments"].value =
    document.getElementById("richedit").innerHTML;
});
```

---

## 19.6 小结

<aside>
📌

**本章要点回顾**

- 使用 JavaScript 可以方便地操作表单及其字段，通过 `submit` 和 `reset` 事件对表单提交和重置进行拦截
- 文本框可以通过 `select()`、`setSelectionRange()` 方法以及剪贴板事件进行精细控制
- 选择框提供了 `add()`、`remove()` 等方法用于动态操作选项
- HTML5 约束验证 API 提供了原生的客户端验证能力，包括 `required`、`pattern`、`checkValidity()` 等
- `contenteditable` 属性和 `document.execCommand()` 方法使富文本编辑成为可能
- 富文本编辑器的内容需要手动提取到隐藏字段中才能通过表单提交
</aside>