---
title: 第24章 网络请求与远程资源
date: 2026-02-17 15:02:15
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 24.1 XMLHttpRequest 对象

Ajax（Asynchronous JavaScript and XML）技术的核心是 `XMLHttpRequest`（XHR）对象，最早由微软在 IE5 中引入，后被所有浏览器采纳。

### 24.1.1 使用 XHR

```jsx
let xhr = new XMLHttpRequest();

// open() 接受三个参数：请求方法、URL、是否异步
xhr.open("get", "example.php", true);

// send() 接受一个参数：请求体数据，若不需要则传 null
xhr.send(null);
```

**同步请求**下可直接访问响应属性；**异步请求**下需监听 `readyState` 变化：

| `readyState` 值 | 状态 | 说明 |
| --- | --- | --- |
| 0 | 未初始化（Unsent） | 尚未调用 `open()` |
| 1 | 已打开（Opened） | 已调用 `open()`，未调用 `send()` |
| 2 | 已发送（Headers Received） | 已调用 `send()`，已收到响应头 |
| 3 | 接收中（Loading） | 正在接收响应体 |
| 4 | 完成（Done） | 响应数据接收完毕 |

```jsx
xhr.onreadystatechange = function() {
  if (xhr.readyState === 4) {
    if (xhr.status >= 200 && xhr.status < 300 || xhr.status === 304) {
      console.log(xhr.responseText);
    } else {
      console.error("Request failed: " + xhr.status);
    }
  }
};
```

响应对象的关键属性：

- **`responseText`**：响应体文本
- **`responseXML`**：如果响应内容类型是 `"text/xml"` 或 `"application/xml"`，则包含响应的 XML DOM 文档
- **`status`**：HTTP 状态码
- **`statusText`**：HTTP 状态描述

> 在收到响应之前可以调用 `xhr.abort()` 取消异步请求。调用后 XHR 对象会停止触发事件，也不再允许访问任何与响应相关的属性。
> 

### 24.1.2 HTTP 头部

默认情况下，XHR 请求会发送以下头部：

- `Accept`：浏览器可以处理的内容类型
- `Accept-Charset`：浏览器可以显示的字符集
- `Accept-Encoding`：浏览器可以处理的压缩编码类型
- `Accept-Language`：浏览器当前设置的语言
- `Connection`：浏览器与服务器的连接类型
- `Cookie`：页面中设置的 Cookie
- `Host`：发送请求的页面所在的域
- `Referer`：发送请求的页面的 URI
- `User-Agent`：浏览器的用户代理字符串

使用 `setRequestHeader()` 可以设置**自定义**请求头，必须在 `open()` 之后、`send()` 之前调用：

```jsx
xhr.open("get", "example.php", true);
xhr.setRequestHeader("MyHeader", "MyValue");
xhr.send(null);
```

使用 `getResponseHeader()` 和 `getAllResponseHeaders()` 获取响应头。

### 24.1.3 GET 请求

查询字符串中的每个名和值都必须使用 `encodeURIComponent()` 编码：

```jsx
function addURLParam(url, name, value) {
  url += (url.indexOf("?") === -1 ? "?" : "&");
  url += encodeURIComponent(name) + "=" + encodeURIComponent(value);
  return url;
}
```

### 24.1.4 POST 请求

```jsx
xhr.open("post", "example.php", true);
xhr.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");

let form = document.getElementById("user-info");
xhr.send(serialize(form));  // 序列化表单数据
```

> POST 请求比 GET 请求占用更多资源。从性能角度看，发送相同数据量的请求，GET 最多可比 POST 快两倍。
> 

### 24.1.5 XMLHttpRequest Level 2

**`FormData` 类型**：方便序列化表单及创建与表单类似格式的数据：

```jsx
let data = new FormData();
data.append("name", "Nicholas");

// 或直接传入表单元素
let data2 = new FormData(document.forms[0]);

xhr.send(data);  // 不需要手动设置 Content-Type，XHR 会自动识别
```

**超时设定**：

```jsx
xhr.timeout = 1000;  // 1秒超时
xhr.ontimeout = function() {
  console.log("Request timed out.");
};
```

**`overrideMimeType()`**：强制将响应当作指定 MIME 类型处理：

```jsx
xhr.overrideMimeType("text/xml");  // 强制按 XML 解析
```

---

## 24.2 进度事件

Progress Events 规范定义了客户端-服务器通信相关的事件：

| 事件 | 说明 |
| --- | --- |
| `loadstart` | 接收到响应的第一个字节时触发 |
| `progress` | 接收响应期间反复触发 |
| `error` | 请求出错时触发 |
| `abort` | 调用 `abort()` 终止时触发 |
| `load` | 成功接收完响应时触发 |
| `loadend` | 通信完成时触发（在 error / abort / load 之后） |

### 24.2.1 load 事件

`load` 事件可简化对 `readyState === 4` 的判断：

```jsx
xhr.onload = function() {
  if (xhr.status >= 200 && xhr.status < 300 || xhr.status === 304) {
    console.log(xhr.responseText);
  }
};
```

### 24.2.2 progress 事件

`progress` 事件的 `event` 对象包含三个额外属性：

- **`lengthComputable`**：布尔值，表示进度信息是否可用
- **`position`**：已接收的字节数
- **`totalSize`**：根据 `Content-Length` 头部预期的总字节数

```jsx
xhr.onprogress = function(event) {
  if (event.lengthComputable) {
    let percentComplete = event.position / event.totalSize * 100;
    console.log(percentComplete + "% 完成");
  }
};
```

---

## 24.3 跨源资源共享（CORS）

默认情况下，XHR 只能访问与发起请求的页面在**同一个域**内的资源，即**同源策略**。

**CORS**（Cross-Origin Resource Sharing）使用额外的 HTTP 头部让浏览器与服务器协商跨域访问：

1. 浏览器在请求中自动添加 `Origin` 头部：`Origin: http://www.example.com`
2. 服务器通过 `Access-Control-Allow-Origin` 响应头表示允许：`Access-Control-Allow-Origin: http://www.example.com`（或 `*` 允许所有来源）

### 简单请求 vs 预检请求

**简单请求**条件（满足以下全部）：

- 方法为 GET、POST 或 HEAD
- 只使用安全头部（`Accept`、`Accept-Language`、`Content-Language`、`Content-Type` 仅限 `application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`）

不满足简单请求条件的请求会先发送一个 **OPTIONS 预检请求**（Preflight）：

```
OPTIONS /resource HTTP/1.1
Origin: http://www.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: MyCustomHeader
```

服务器确认后响应：

```
Access-Control-Allow-Origin: http://www.example.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: MyCustomHeader
Access-Control-Max-Age: 1728000
```

> `Access-Control-Max-Age` 指定预检请求的缓存时间（秒），在此期间不需要重新发送预检请求。
> 

### 凭据请求

默认情况下，跨源请求**不会发送凭据**（Cookie、HTTP 认证、客户端 SSL 证书）。可通过设置实现：

```jsx
xhr.withCredentials = true;
```

服务器端需返回：`Access-Control-Allow-Credentials: true`

---

## 24.4 替代性跨源技术

### 24.4.1 图片探测（Image Pings）

利用 `<img>` 标签的 `src` 不受同源策略限制：

```jsx
let img = new Image();
img.onload = img.onerror = function() {
  console.log("Done!");
};
img.src = "http://www.example.com/test?name=Nicholas";
```

- 只能发送 **GET** 请求
- 无法获取服务器响应内容
- 常用于**跟踪用户点击**和**动态广告曝光次数**

### 24.4.2 JSONP

JSONP（JSON with Padding）通过动态创建 `<script>` 元素实现跨域数据获取：

```jsx
function handleResponse(response) {
  console.log(response.ip, response.city);
}

let script = document.createElement("script");
script.src = "http://freegeoip.net/json/?callback=handleResponse";
document.body.insertBefore(script, document.body.firstChild);
```

<aside>
⚠️

**JSONP 的缺点**：

- 从其他域加载代码执行，存在安全风险
- 难以确定请求是否失败（没有可靠的错误处理机制）
- 只支持 GET 请求
</aside>

---

## 24.5 Fetch API

Fetch API 是 XHR 的现代替代方案，提供了基于 **Promise** 的更简洁接口。

### 24.5.1 基本用法

```jsx
// 最简单的用法，默认 GET 请求
fetch("bar.txt")
  .then(response => {
    console.log(response.status);    // 200
    console.log(response.statusText); // OK
  });

// 使用 await
let response = await fetch("bar.txt");
```

`fetch()` 返回一个 `Response` 对象，其关键属性：

| 属性 | 说明 |
| --- | --- |
| `status` | HTTP 状态码 |
| `statusText` | 状态文本 |
| `ok` | 布尔值，状态码 200~299 时为 true |
| `headers` | 响应头的 Headers 对象 |
| `url` | 最终的请求 URL（跟随重定向后的） |
| `redirected` | 是否经过重定向 |
| `type` | 响应类型（basic / cors / error 等） |

### 24.5.2 常见的 Fetch 请求模式

**发送 JSON 数据**：

```jsx
let response = await fetch("/api/user", {
  method: "POST",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({ name: "Nicholas" })
});
```

**发送 FormData**：

```jsx
let formData = new FormData();
formData.append("name", "Nicholas");

fetch("/api/user", {
  method: "POST",
  body: formData
});
```

**发送文件**：

```jsx
let fileInput = document.querySelector('input[type="file"]');
let formData = new FormData();
formData.append("file", fileInput.files[0]);

fetch("/upload", { method: "POST", body: formData });
```

**加载 Blob 文件**：

```jsx
let response = await fetch("my-image.png");
let blob = await response.blob();

let objectURL = URL.createObjectURL(blob);
let img = document.createElement("img");
img.src = objectURL;
document.body.appendChild(img);
```

### 24.5.3 Headers 对象

`Headers` 对象是所有请求头/响应头的集合，类似于 `Map`：

```jsx
let headers = new Headers();
headers.set("Content-Type", "application/json");
headers.has("Content-Type");  // true
headers.get("Content-Type");  // "application/json"
headers.delete("Content-Type");

// 支持迭代
for (let [key, value] of headers.entries()) {
  console.log(`${key}: ${value}`);
}
```

### 24.5.4 Request 对象

```jsx
let request = new Request("https://foo.com", {
  method: "POST",
  body: "foobar"
});

// 将 Request 对象传给 fetch
fetch(request);

// 克隆请求（用于需要多次发送同一请求的情况）
let request2 = request.clone();
```

> **注意**：使用过 `body` 的 `Request` 对象不能再次使用，必须先 `clone()`。
> 

### 24.5.5 Response 对象

**读取响应体**的方法（均返回 Promise）：

| 方法 | 返回值 | 典型用途 |
| --- | --- | --- |
| `response.text()` | `string` | 纯文本 |
| `response.json()` | 解析后的 JSON 对象 | JSON 数据 |
| `response.blob()` | `Blob` | 二进制文件 |
| `response.arrayBuffer()` | `ArrayBuffer` | 底层二进制 |
| `response.formData()` | `FormData` | 表单数据 |

> **Body 只能读取一次**。读取后 `bodyUsed` 属性变为 `true`，再次读取会抛出错误。如需多次读取，先调用 `response.clone()`。
> 

```jsx
let response = await fetch("data.json");
let data = await response.json();
console.log(data);
```

### 24.5.6 Request 的 `init` 参数

`fetch()` 第二个参数是一个配置对象，常用选项：

| 选项 | 说明 | 默认值 |
| --- | --- | --- |
| `method` | 请求方法 | `"GET"` |
| `headers` | 请求头 | `{}` |
| `body` | 请求体 | `null` |
| `mode` | 跨源模式 | `"cors"` |
| `credentials` | 凭据策略 | `"same-origin"` |
| `cache` | 缓存策略 | `"default"` |
| `redirect` | 重定向处理 | `"follow"` |
| `signal` | `AbortSignal` 实例 | — |
| `keepalive` | 允许页面卸载后继续请求 | `false` |

**`credentials` 选项**：

- `"omit"`：不发送也不接收 Cookie
- `"same-origin"`：同源时发送 Cookie（默认）
- `"include"`：始终发送 Cookie（类似 `xhr.withCredentials = true`）

**`cache` 选项**：

- `"default"`：正常缓存行为
- `"no-store"`：完全忽略缓存
- `"reload"`：忽略缓存直接请求，但用响应更新缓存
- `"no-cache"`：无条件发起请求，协商缓存
- `"force-cache"`：优先使用缓存，即使已过期
- `"only-if-cached"`：只在有缓存时返回，否则返回 504

### 24.5.7 中断请求（AbortController）

```jsx
let controller = new AbortController();

fetch("long-request.txt", { signal: controller.signal })
  .catch(() => console.log("Request aborted!"));

// 在需要的时候中断请求
setTimeout(() => controller.abort(), 3000);
```

---

## 24.6 Beacon API

`navigator.sendBeacon()` 用于在页面卸载时异步发送少量数据到服务器，保证数据不会因页面关闭而丢失：

```jsx
navigator.sendBeacon("https://example.com/analytics", JSON.stringify({
  event: "page_exit",
  duration: 5000
}));
```

<aside>
💡

**Beacon 的特点**：

- 发送 **POST** 请求
- 请求会在**页面卸载后继续**（不会因页面关闭而取消）
- 对数据大小有限制（通常 **64KB**）
- 不能接收服务器响应
- 常用于**分析数据**和**诊断日志**
</aside>

---

## 24.7 Web Socket

Web Socket 提供了浏览器与服务器之间的**全双工、双向通信**通道。

### 24.7.1 创建 Web Socket

```jsx
// 必须使用 ws:// 或 wss://（加密）协议
let socket = new WebSocket("wss://www.example.com/server");
```

`readyState` 属性表示当前状态：

| 值 | 常量 | 说明 |
| --- | --- | --- |
| 0 | `WebSocket.OPENING` | 正在建立连接 |
| 1 | `WebSocket.OPEN` | 连接已建立 |
| 2 | `WebSocket.CLOSING` | 正在关闭 |
| 3 | `WebSocket.CLOSE` | 连接已关闭 |

### 24.7.2 发送和接收数据

```jsx
// 发送文本
socket.send("Hello Server!");

// 发送 JSON
socket.send(JSON.stringify({ type: "message", content: "Hello" }));

// 接收消息
socket.onmessage = function(event) {
  let data = event.data;  // 可能是字符串或 Blob/ArrayBuffer
  console.log(data);
};
```

### 24.7.3 其他事件

```jsx
socket.onopen = function() {
  console.log("Connection established.");
};

socket.onerror = function() {
  console.log("Connection error.");
};

socket.onclose = function(event) {
  console.log(`Closed: code=${event.code}, reason=${event.reason}, clean=${event.wasClean}`);
};
```

> 关闭连接使用 `socket.close()`。WebSocket 不支持自动重连，需要手动实现。
> 

---

## 24.8 安全

<aside>
🔒

在未授权系统中利用 Ajax 发起攻击的常见手段：

- **CSRF**（跨站请求伪造）：伪造合法用户的请求，在未经授权的情况下执行操作
- **注入攻击**：将恶意数据注入请求中

常见防御策略：

- 要求使用 **SSL/TLS** 访问受保护资源
- 要求每个请求附带**验证 Token**（非 Cookie 中的信息）
- 检查请求来源的 **Referer** 头部
- 使用基于 Cookie 信息的**认证令牌**验证机制
</aside>

---

## 本章小结

<aside>
📝

- **Ajax** 的核心是 `XMLHttpRequest`（XHR），支持异步发送请求和接收响应
- **CORS** 通过额外的 HTTP 头部实现受控的跨域访问
- 旧式跨域方案包括**图片探测**和 **JSONP**，但功能和安全性有限
- **Fetch API** 是 XHR 的现代替代，基于 Promise，接口更简洁强大
- **Beacon API** 适合在页面卸载时发送分析数据
- **Web Socket** 实现了全双工双向通信，适合实时应用（聊天、实时推送等）
- 安全方面需防范 CSRF、注入攻击，应使用 SSL 和 Token 等防御措施
</aside>