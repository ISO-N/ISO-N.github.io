---
title: 第12章 BOM
date: 2026-02-17 15:02:05
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
BOM（Browser Object Model，浏览器对象模型）提供了与浏览器窗口交互的对象。BOM 的核心是 `window` 对象，它既是 ECMAScript 中的 `Global` 对象，也是浏览器窗口的 JavaScript 接口。

---

## window 对象

`window` 对象在浏览器中有双重角色：

- 作为 ECMAScript 的 **Global 对象**，所有全局变量和函数都是它的属性和方法
- 作为浏览器窗口的 **JavaScript 接口**

### Global 作用域

通过 `var` 声明的全局变量和函数会成为 `window` 的属性和方法：

```jsx
var age = 29;
var sayAge = () => alert(this.age);

alert(window.age);   // 29
sayAge();             // 29
window.sayAge();      // 29
```

<aside>
⚠️

使用 `let` 或 `const` 声明的全局变量**不会**挂载到 `window` 对象上。

</aside>

`var` 声明的全局变量与直接在 `window` 上定义属性的区别：全局变量不能通过 `delete` 删除，而直接定义的属性可以。

```jsx
var age = 29;
window.color = 'red';

delete window.age;    // false
delete window.color;  // true
```

### 窗口关系

- `window.top`：始终指向最外层窗口（浏览器窗口本身）
- `window.parent`：指向当前窗口的父窗口
- `window.self`：始终指向 `window` 本身

### 窗口位置与像素比

窗口相对于屏幕左上角的位置：

- `window.screenLeft` / `window.screenTop`：窗口在屏幕上的坐标
- `window.moveTo(x, y)`：移动到绝对坐标 (x, y)
- `window.moveBy(dx, dy)`：相对当前位置移动 (dx, dy)

**像素比**：`window.devicePixelRatio` 表示物理像素与 CSS 像素的比率。例如手机屏幕上该值通常为 2 或 3。

### 窗口大小

| 属性 | 含义 |
| --- | --- |
| `outerWidth` / `outerHeight` | 浏览器窗口自身的大小 |
| `innerWidth` / `innerHeight` | 浏览器视口（viewport）大小，不包含浏览器边框和工具栏 |
| `document.documentElement.clientWidth/clientHeight` | 页面视口的宽高 |

调整窗口大小：

- `window.resizeTo(width, height)`：调整到指定大小
- `window.resizeBy(dw, dh)`：相对当前大小调整

### 视口位置

文档相对于视口的滚动距离：

- `window.scrollX` / `window.pageXOffset`
- `window.scrollY` / `window.pageYOffset`

滚动方法：

```jsx
// 滚动到固定坐标
window.scrollTo(0, 100);

// 相对当前位置滚动
window.scrollBy(0, 100);

// 使用 options 对象，支持平滑滚动
window.scrollTo({
  left: 0,
  top: 100,
  behavior: 'smooth'   // 平滑滚动
});
```

### 导航与打开新窗口

`window.open()` 可以导航到指定 URL 或打开新窗口：

```jsx
// 参数：URL, 目标窗口, 特性字符串, 是否替换历史记录
window.open("https://www.example.com", "newWindow",
  "height=400,width=400,top=10,left=10,resizable=yes");
```

- 返回新窗口的 `window` 对象引用
- 新窗口可通过 `window.opener` 访问打开它的窗口
- 将 `opener` 设为 `null` 可切断两个窗口之间的联系

**弹窗检测**：浏览器的弹窗屏蔽程序会阻止弹窗，可用以下方式检测：

```jsx
let blocked = false;
try {
  let popup = window.open("https://www.example.com", "_blank");
  if (popup == null) {
    blocked = true;
  }
} catch (ex) {
  blocked = true;
}
if (blocked) {
  alert("弹窗被屏蔽了！");
}
```

### 定时器

- **`setTimeout(fn, delay)`**：在指定毫秒后执行一次回调

```jsx
let timeoutId = setTimeout(() => alert("Hello!"), 1000);
clearTimeout(timeoutId);  // 取消
```

- **`setInterval(fn, interval)`**：每隔指定毫秒重复执行

```jsx
let intervalId = setInterval(() => alert("Hello!"), 1000);
clearInterval(intervalId);  // 取消
```

<aside>
💡

一般推荐用 `setTimeout` 模拟 `setInterval`，可以更精确地控制时间间隔，避免后一个定时任务在前一个尚未完成时就被加入队列。

</aside>

### 系统对话框

- `alert(message)`：警告框，仅含确认按钮
- `confirm(message)`：确认框，返回 `true`（确定）或 `false`（取消）
- `prompt(message, default)`：提示框，返回用户输入的字符串或 `null`

这些对话框是**同步模态**的，调用后代码会暂停执行。

---

## location 对象

`location` 既是 `window` 的属性，也是 `document` 的属性（`window.location === document.location`）。它提供了当前窗口加载文档的 URL 信息，并提供导航功能。

### URL 组成属性

假设 URL 为 `https://user:pass@www.example.com:8080/path/?q=js#contents`：

| 属性 | 值 | 说明 |
| --- | --- | --- |
| `location.hash` | `"#contents"` | URL 散列值 |
| `location.host` | `"www.example.com:8080"` | 域名 + 端口 |
| `location.hostname` | `"www.example.com"` | 仅域名 |
| `location.href` | 完整 URL | 完整 URL 字符串 |
| `location.pathname` | `"/path/"` | 路径 |
| `location.port` | `"8080"` | 端口号 |
| `location.protocol` | `"https:"` | 协议 |
| `location.search` | `"?q=js"` | 查询字符串 |
| `location.origin` | `"https://www.example.com:8080"` | 源（只读） |
| `location.username` | `"user"` | 用户名 |
| `location.password` | `"pass"` | 密码 |

### URLSearchParams

`URLSearchParams` 提供了标准化的查询字符串操作方法：

```jsx
let params = new URLSearchParams("?q=javascript&num=10");

params.has("q");            // true
params.get("q");            // "javascript"
params.set("page", "1");
params.delete("num");
params.toString();          // "q=javascript&page=1"

// 可迭代
for (let [key, value] of params) {
  console.log(`${key} = ${value}`);
}
```

### 操作地址

```jsx
// 以下三种方式等价，都会导航到新 URL 并在历史记录中增加一条记录
location.assign("https://www.example.com");
window.location = "https://www.example.com";
location.href = "https://www.example.com";

// 修改 location 的其他属性也会触发页面导航并添加历史记录
location.hash = "#section1";
location.search = "?q=js";
location.hostname = "www.other.com";
location.pathname = "/new-path";
location.port = 8080;
```

<aside>
⚠️

`location.replace(url)` 会导航到新 URL 但**不会**在历史记录中增加新条目，用户无法回退到前一个页面。

</aside>

`location.reload()`：重新加载当前页面。传入 `true` 强制从服务器重新加载（而非使用缓存）。

```jsx
location.reload();      // 可能从缓存加载
location.reload(true);  // 强制从服务器加载
```

---

## navigator 对象

`navigator` 对象包含浏览器和操作系统的相关信息，是客户端标识浏览器的标准。

### 常用属性与方法

| 属性/方法 | 说明 |
| --- | --- |
| `navigator.userAgent` | 浏览器的用户代理字符串 |
| `navigator.language` | 浏览器的主语言（如 `"zh-CN"`） |
| `navigator.languages` | 浏览器偏好语言数组 |
| `navigator.onLine` | 是否联网 |
| `navigator.geolocation` | Geolocation API，获取用户地理位置 |
| `navigator.clipboard` | 剪贴板 API |
| `navigator.sendBeacon(url, data)` | 异步发送少量数据到服务器（常用于统计） |
| `navigator.vendor` | 浏览器厂商名称 |
| `navigator.platform` | 操作系统平台 |
| `navigator.cookieEnabled` | 是否启用 Cookie |

### 检测插件

通过 `navigator.plugins` 数组可以检测浏览器安装的插件（主要适用于非 IE 浏览器）：

```jsx
for (let plugin of navigator.plugins) {
  console.log(plugin.name);         // 插件名
  console.log(plugin.description);  // 插件描述
  console.log(plugin.filename);     // 插件文件名
}
```

### 注册处理程序

`navigator.registerProtocolHandler()` 可以将一个网站注册为处理特定协议的应用程序：

```jsx
navigator.registerProtocolHandler(
  "mailto",
  "https://www.example.com?cmd=%s",
  "Example Mail"
);
```

---

## screen 对象

`screen` 对象保存了客户端显示器的信息，通常用于收集浏览器运行环境的能力信息。

| 属性 | 说明 |
| --- | --- |
| `screen.width` / `screen.height` | 屏幕的像素宽度/高度 |
| `screen.availWidth` / `screen.availHeight` | 屏幕可用区域的宽度/高度（减去系统任务栏等） |
| `screen.colorDepth` | 屏幕颜色的位数（通常为 24 或 32） |
| `screen.pixelDepth` | 屏幕的位深度 |
| `screen.orientation` | 屏幕朝向信息（`type` 和 `angle`） |

---

## history 对象

`history` 对象表示当前窗口的浏览历史记录。出于安全考虑，无法获取用户访问过的具体 URL。

### 导航

```jsx
// 后退一页
history.go(-1);
history.back();

// 前进一页
history.go(1);
history.forward();

// 前进/后退 n 页
history.go(n);

// 历史记录条数
history.length;  // 如果是第一个打开的页面，值为 1
```

### 历史状态管理

HTML5 引入了 `pushState()` 和 `replaceState()` 方法，允许在不刷新页面的情况下操作浏览器历史记录，这是**单页应用（SPA）**路由的核心基础。

```jsx
// 添加一条历史记录
// 参数：状态对象, 标题(通常为空字符串), 新 URL(可选)
history.pushState({ page: 1 }, "", "/page1");

// 替换当前历史记录（不会新增记录）
history.replaceState({ page: 2 }, "", "/page2");
```

当用户点击后退/前进按钮时，会触发 `popstate` 事件：

```jsx
window.addEventListener("popstate", (event) => {
  let state = event.state;  // 取回 pushState 时传入的状态对象
  if (state) {
    processState(state);
  }
});
```

<aside>
💡

`pushState()` 创建的状态对象大小限制通常在 **500KB ~ 2MB** 之间（取决于浏览器），应当仅存储必要数据。

</aside>

<aside>
📌

**小结**：BOM 以 `window` 对象为核心，通过 `location` 管理 URL 和导航，通过 `navigator` 获取浏览器信息，通过 `screen` 获取显示器信息，通过 `history` 管理历史记录。现代 Web 开发中，`location` 和 `history`（尤其是 `pushState`）是最常用的 BOM API。

</aside>