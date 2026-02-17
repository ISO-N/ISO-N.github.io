---
title: 第17章 事件
date: 2026-02-17 15:02:09
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
JavaScript 与 HTML 的交互通过**事件**实现。事件是文档或浏览器窗口中发生的特定交互瞬间，可以使用**监听器（处理程序）**来订阅事件，以便在事件发生时执行相应代码。

## 17.1 事件流

**事件流**描述了页面接收事件的顺序。IE 和 Netscape 提出了几乎完全相反的事件流方案。

### 17.1.1 事件冒泡

IE 的事件流称为**事件冒泡（Event Bubbling）**：事件从最具体的元素（文档树中最深的节点）开始触发，然后向上传播到较不具体的节点（文档）。

```
点击 <div> 后的冒泡顺序：
<div> → <body> → <html> → document
```

现代浏览器中，事件会一直冒泡到 `window` 对象。

### 17.1.2 事件捕获

Netscape 提出的**事件捕获（Event Capturing）**：最不具体的节点最先收到事件，最具体的节点最后收到事件。

```
点击 <div> 后的捕获顺序：
document → <html> → <body> → <div>
```

> 由于老版本浏览器不支持捕获，通常建议使用事件冒泡，特殊情况下才使用事件捕获。
> 

### 17.1.3 DOM 事件流

DOM2 Events 规范规定事件流分为 **3 个阶段**：

1. **事件捕获阶段**（Capturing Phase）—— 为提前拦截事件提供机会
2. **到达目标**（Target Phase）—— 事件到达实际目标元素
3. **事件冒泡阶段**（Bubbling Phase）—— 最迟在此阶段响应事件

```
document → <html> → <body> → [目标<div>] → <body> → <html> → document
         ← 捕获阶段 →     ↑目标↑     ← 冒泡阶段 →
```

---

## 17.2 事件处理程序

事件是用户或浏览器执行的某种动作，为响应事件而调用的函数称为**事件处理程序（Event Handler）**，以 `on` 开头，如 `onclick`、`onload`。

### 17.2.1 HTML 事件处理程序

直接在 HTML 属性中指定事件处理程序：

```html
<input type="button" value="Click Me" onclick="console.log('Clicked')" />
```

也可以调用在别处定义的脚本函数：

```html
<script>
  function showMessage() {
    console.log("Hello world!");
  }
</script>
<input type="button" value="Click Me" onclick="showMessage()" />
```

**特点：**

- 函数中有一个特殊的局部变量 `event`，即事件对象
- `this` 值等于事件的目标元素
- 动态创建的函数作用域链被扩展，可以直接访问 `document` 及元素自身的成员

**缺点：**

- **时差问题**：用户可能在 HTML 出现后、JS 加载前就触发事件
- **作用域链扩展**在不同浏览器中可能导致不同结果
- **HTML 与 JavaScript 强耦合**，不利于维护

### 17.2.2 DOM0 事件处理程序

通过 JavaScript 指定事件处理程序的传统方式——将一个函数赋值给 DOM 元素的事件处理程序属性：

```jsx
let btn = document.getElementById("myBtn");
btn.onclick = function () {
  console.log("Clicked");
  console.log(this.id); // "myBtn"
};
```

- 事件处理程序在元素的作用域中运行，`this` 等于元素本身
- 以这种方式添加的事件处理程序在**冒泡阶段**处理
- 移除事件处理程序：`btn.onclick = null;`

### 17.2.3 DOM2 事件处理程序

DOM2 Events 定义了两个方法：

- `addEventListener(eventType, handler, useCapture)`
- `removeEventListener(eventType, handler, useCapture)`

第三个参数 `useCapture`：`true` 表示在**捕获阶段**调用，`false`（默认）表示在**冒泡阶段**调用。

```jsx
let btn = document.getElementById("myBtn");

btn.addEventListener("click", () => {
  console.log(this.id);
}, false);
```

**优势**：可以为同一个事件添加**多个**处理程序，按添加顺序触发。

```jsx
btn.addEventListener("click", () => { console.log(this.id); }, false);
btn.addEventListener("click", () => { console.log("Hello world!"); }, false);
```

> ⚠️ 通过 `addEventListener()` 添加的事件处理程序只能使用 `removeEventListener()` 移除，且必须传入**同一个函数引用**。因此使用匿名函数添加的处理程序**无法移除**。
> 

### 17.2.4 IE 事件处理程序

IE 实现了 `attachEvent()` 和 `detachEvent()`（已过时，了解即可）：

- 事件处理程序在**全局作用域**中运行，`this === window`
- 多个处理程序以**添加顺序的反序**触发

### 17.2.5 跨浏览器事件处理程序

可以编写一个工具对象来屏蔽浏览器差异：

```jsx
var EventUtil = {
  addHandler: function (element, type, handler) {
    if (element.addEventListener) {
      element.addEventListener(type, handler, false);
    } else if (element.attachEvent) {
      element.attachEvent("on" + type, handler);
    } else {
      element["on" + type] = handler;
    }
  },
  removeHandler: function (element, type, handler) {
    if (element.removeEventListener) {
      element.removeEventListener(type, handler, false);
    } else if (element.detachEvent) {
      element.detachEvent("on" + type, handler);
    } else {
      element["on" + type] = null;
    }
  }
};
```

---

## 17.3 事件对象

在 DOM 中发生事件时，所有相关信息都会被收集在一个名为 `event` 的对象中。

### 17.3.1 DOM 事件对象

`event` 对象是传给事件处理程序的**唯一参数**：

```jsx
btn.onclick = function (event) {
  console.log(event.type); // "click"
};

btn.addEventListener("click", (event) => {
  console.log(event.type); // "click"
}, false);
```

**常用属性和方法：**

| **属性 / 方法** | **类型** | **说明** |
| --- | --- | --- |
| `type` | String | 被触发的事件类型 |
| `target` | Element | 事件目标（触发事件的元素） |
| `currentTarget` | Element | 当前正在处理事件的元素（`this` 始终等于它） |
| `eventPhase` | Number | 1 = 捕获，2 = 目标，3 = 冒泡 |
| `bubbles` | Boolean | 事件是否冒泡 |
| `cancelable` | Boolean | 是否可以取消事件的默认行为 |
| `preventDefault()` | Function | 取消事件的默认行为（如阻止链接跳转） |
| `stopPropagation()` | Function | 阻止事件继续传播（捕获或冒泡） |
| `stopImmediatePropagation()` | Function | 阻止传播并阻止同一元素上的其他处理程序执行 |
| `defaultPrevented` | Boolean | `preventDefault()` 被调用后为 `true` |
| `trusted` | Boolean | `true` 表示浏览器生成，`false` 表示开发者模拟 |

**`target` vs `currentTarget`：**

```jsx
document.body.onclick = function (event) {
  console.log(event.currentTarget === document.body); // true (this)
  console.log(event.target === btn);                  // true (实际点击的按钮)
};
```

### 17.3.2 IE 事件对象（了解）

IE 中事件对象是 `window.event`，属性不同：`srcElement`（对应 `target`）、`returnValue`（对应 `preventDefault()`）、`cancelBubble`（对应 `stopPropagation()`）。

### 17.3.3 跨浏览器事件对象

可在 `EventUtil` 中增加方法统一获取事件对象、目标元素、阻止默认行为和停止传播等操作。

---

## 17.4 事件类型

DOM3 Events 定义了如下几类事件：

- **用户界面事件**（UI Events）—— 涉及与 BOM 交互的通用浏览器事件
- **焦点事件**（Focus Events）—— 元素获取和失去焦点时触发
- **鼠标事件**（Mouse Events）—— 使用鼠标在页面上操作时触发
- **滚轮事件**（Wheel Events）—— 使用鼠标滚轮时触发
- **输入事件**（Input Events）—— 向文档中输入文本时触发
- **键盘事件**（Keyboard Events）—— 使用键盘在页面上操作时触发
- **合成事件**（Composition Events）—— 使用 IME 输入法时触发

### 17.4.1 UI 事件

| **事件** | **说明** |
| --- | --- |
| `load` | 页面（`window`）、图片、脚本等加载完成后触发 |
| `unload` | 页面完全卸载后触发，常用于清理引用防止内存泄漏 |
| `resize` | 浏览器窗口被缩放时在 `window` 上触发 |
| `scroll` | 当用户滚动含滚动条的元素时触发 |
| `select` | 在文本框中选择了文本时触发 |

### 17.4.2 焦点事件

| **事件** | **说明** |
| --- | --- |
| `focus` | 元素获得焦点时触发，**不冒泡** |
| `blur` | 元素失去焦点时触发，**不冒泡** |
| `focusin` | 元素获得焦点时触发，**冒泡**（`focus` 的冒泡版） |
| `focusout` | 元素失去焦点时触发，**冒泡**（`blur` 的冒泡版） |

焦点从元素 A 移到元素 B 时的触发顺序：`focusout`(A) → `focusin`(B) → `blur`(A) → `focus`(B)

### 17.4.3 鼠标和滚轮事件

DOM3 Events 定义了 9 个鼠标事件：

- `click` —— 单击主按键或按回车触发
- `dblclick` —— 双击主按键触发
- `mousedown` —— 按下任意鼠标键触发
- `mouseup` —— 释放鼠标键触发
- `mousemove` —— 鼠标在元素上移动时反复触发
- `mouseenter` —— 鼠标从元素外部移入元素内部，**不冒泡**
- `mouseleave` —— 鼠标从元素内部移到元素外部，**不冒泡**
- `mouseover` —— 鼠标移入元素范围，**冒泡**
- `mouseout` —— 鼠标移出元素范围，**冒泡**

**点击事件顺序**：`mousedown` → `mouseup` → `click` → `mousedown` → `mouseup` → `click` → `dblclick`

**坐标相关属性：**

- **客户端坐标**：`event.clientX`、`event.clientY`（相对于视口）
- **页面坐标**：`event.pageX`、`event.pageY`（相对于页面，含滚动偏移）
- **屏幕坐标**：`event.screenX`、`event.screenY`（相对于整个屏幕）

**修饰键**：`event.shiftKey`、`event.ctrlKey`、`event.altKey`、`event.metaKey`（布尔值）

**相关元素**：`mouseover` 和 `mouseout` 事件的 `event.relatedTarget` 指向相关元素。

**鼠标按键**：`event.button`（0 = 主键，1 = 中键，2 = 副键）

**滚轮事件**：`mousewheel`（`event.wheelDelta`，向前滚 +120，向后滚 -120）

### 17.4.4 键盘与输入事件

- `keydown` —— 按下键盘上某个键时触发，持续按住会**重复触发**
- `keypress` —— 按下键盘上某个键并产生字符时触发（**已废弃**，使用 `textInput` 替代）
- `keyup` —— 释放键盘上某个键时触发

**触发顺序**：`keydown` → `keypress` → `keyup`

**键码（keyCode）与字符编码（charCode）**：

- `keydown` / `keyup` 中 `event.keyCode` 对应键的键码
- DOM3 新增 `event.key`（字符串，如 `"k"`、`"ArrowDown"`）和 `event.code`（物理键位，如 `"KeyK"`、`"ArrowDown"`）

**textInput 事件**：在字符被输入到可编辑区域时触发，只有实际字符输入才会触发（区别于 `keypress`），`event.data` 包含输入的字符。

### 17.4.5 合成事件（Composition Events）

用于处理 IME（输入法编辑器）输入：

- `compositionstart` —— IME 打开时触发
- `compositionupdate` —— 新字符插入输入字段时触发
- `compositionend` —— IME 关闭，返回正常输入时触发

### 17.4.6 变化事件（Mutation Events，已废弃）

DOM2 定义的变化事件用于 DOM 结构发生变化时通知，包括 `DOMNodeInserted`、`DOMNodeRemoved` 等，现已被 **MutationObserver** 取代。

### 17.4.7 HTML5 事件

| **事件** | **说明** |
| --- | --- |
| `contextmenu` | 右键菜单事件，允许自定义上下文菜单 |
| `beforeunload` | 页面卸载前触发，可提示用户"确定离开？" |
| `DOMContentLoaded` | DOM 树构建完成后触发（不等待图片、CSS 等资源），比 `load` 更早 |
| `readystatechange` | 文档或元素加载状态变化时触发（`loading` → `interactive` → `complete`） |
| `hashchange` | URL 散列值（`#` 后面的部分）发生变化时在 `window` 上触发 |

### 17.4.8 设备事件

- `orientationchange` —— 设备方向改变时触发（`window.orientation`：0°/90°/-90°/180°）
- `deviceorientation` —— 设备在三维空间中方向变化时触发（alpha / beta / gamma 角度）
- `devicemotion` —— 设备移动时触发，可获取加速度信息

### 17.4.9 触摸及手势事件

**触摸事件：**

- `touchstart` —— 手指放到屏幕上时触发
- `touchmove` —— 手指在屏幕上滑动时连续触发（调用 `preventDefault()` 可阻止滚动）
- `touchend` —— 手指从屏幕上移开时触发
- `touchcancel` —— 系统停止跟踪触摸时触发

触摸事件对象包含：

- `touches` —— 屏幕上所有触点的 `Touch` 对象数组
- `targetTouches` —— 当前事件目标上的触点
- `changedTouches` —— 本次事件中变化的触点

每个 `Touch` 对象包含 `clientX/Y`、`pageX/Y`、`screenX/Y`、`identifier`、`target` 等属性。

**手指点击屏幕的事件顺序**：`touchstart` → `touchend` → `mousemove` → `mousedown` → `mouseup` → `click`

**手势事件**（iOS Safari 特有）：

- `gesturestart` —— 一个手指已在屏幕上，另一个手指放上时触发
- `gesturechange` —— 任何一个手指的位置发生变化时触发
- `gestureend` —— 一个手指离开屏幕时触发

`event.rotation`（旋转角度）和 `event.scale`（缩放比例）

---

## 17.5 内存与性能

页面中事件处理程序的数量直接影响页面整体性能。原因包括：每个函数都是对象，占用内存；为指定事件处理程序所需的 DOM 访问会延迟页面交互就绪时间。

### 17.5.1 事件委托

**事件委托**利用事件冒泡，在 DOM 树尽量高的层次上添加一个事件处理程序来管理一类事件：

```jsx
// 不推荐：为每个 <li> 都添加处理程序
// 推荐：利用事件委托
let list = document.getElementById("myLinks");

list.addEventListener("click", (event) => {
  let target = event.target;
  switch (target.id) {
    case "doSomething":
      console.log("做某事");
      break;
    case "goSomewhere":
      location.href = "https://www.example.com";
      break;
    case "sayHi":
      console.log("hi");
      break;
  }
});
```

**优势：**

- 减少内存占用（更少的事件处理程序 = 更少的对象 = 更少的内存）
- 减少 DOM 引用，缩短页面交互就绪时间
- 动态添加的子元素自动拥有事件处理

> 💡 最适合使用事件委托的事件包括：`click`、`mousedown`、`mouseup`、`keydown`、`keyup`、`keypress`。
> 

### 17.5.2 删除事件处理程序

导致"空事件处理程序"的两种情况：

1. **删除带有事件处理程序的元素**（如使用 `innerHTML` 替换）—— 先手动移除处理程序再删除元素
2. **页面卸载**—— 在 `unload` 事件中清除所有事件处理程序

```jsx
// 在删除元素前先移除处理程序
btn.onclick = null;
document.getElementById("myDiv").innerHTML = "Processing...";
```

---

## 17.6 模拟事件

JavaScript 可以通过编程方式触发事件，模拟用户操作。

### 17.6.1 DOM 事件模拟

使用 `document.createEvent()` 创建事件对象，然后使用 `dispatchEvent()` 触发：

```jsx
// 模拟鼠标点击
let event = new MouseEvent("click", {
  bubbles: true,
  cancelable: true,
  view: document.defaultView
});
btn.dispatchEvent(event);
```

**可以模拟的事件类型：**

- **鼠标事件**：`new MouseEvent("click", { ... })`
- **键盘事件**：`new KeyboardEvent("keydown", { key: "a", code: "KeyA", ... })`
- **自定义事件**：`new CustomEvent("myevent", { detail: { ... }, bubbles: true })`

### 17.6.2 自定义 DOM 事件

使用 `CustomEvent` 构造函数创建自定义事件，`detail` 属性可传递任意数据：

```jsx
// 创建并触发自定义事件
let event = new CustomEvent("myevent", {
  detail: { message: "Hello!" },
  bubbles: true,
  cancelable: true
});

element.addEventListener("myevent", (e) => {
  console.log(e.detail.message); // "Hello!"
});

element.dispatchEvent(event);
```

### 17.6.3 IE 事件模拟（了解）

IE 使用 `document.createEventObject()` 创建事件对象，使用 `element.fireEvent()` 触发。已过时。

---

## 小结

<aside>
💡

**事件**是 JavaScript 与网页交互的主要方式。

- 事件流包括**捕获阶段**、**目标阶段**和**冒泡阶段**
- 推荐使用 `addEventListener()` 添加事件处理程序
- `event` 对象包含事件的所有相关信息，如 `target`、`type`、`preventDefault()`、`stopPropagation()` 等
- 事件类型涵盖 UI 事件、鼠标事件、键盘事件、触摸事件、HTML5 事件等
- 使用**事件委托**可以大幅提升性能，减少内存占用
- 及时**删除不用的事件处理程序**以避免内存泄漏
- 可以使用 `CustomEvent` 和 `dispatchEvent()` 模拟和自定义事件
</aside>