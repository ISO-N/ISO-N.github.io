---
title: 第20章 JavaScript API
date: 2026-02-17 15:02:12
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
本章涵盖了浏览器环境中常用的各类 JavaScript API，包括 **Atomics 与 SharedArrayBuffer**、**跨上下文消息**、**Encoding API**、**File API**、**媒体元素**、**原生拖放**、**Notifications API**、**Page Visibility API**、**Streams API**、**计时 API**、**Web 组件** 以及 **Web Cryptography API**。

---

## 20.1 Atomics 与 SharedArrayBuffer

### SharedArrayBuffer

- `SharedArrayBuffer` 与 `ArrayBuffer` 类似，但其引用的内存可以**在多个执行上下文之间共享**（如主线程与 Worker 线程）
- 创建方式：`const sharedBuf = new SharedArrayBuffer(4);`
- 在不同线程中，可以使用 `TypedArray`（如 `Int32Array`）在同一块共享内存上进行读写

### 竞态条件问题

- 多个线程同时读写共享内存时，会产生**竞态条件（Race Condition）**
- 例如两个线程同时对同一个位置执行自增操作，结果可能不是期望值

### Atomics 对象

- `Atomics` 提供了一组**原子操作**方法，保证操作在共享内存上是不可中断的
- 常用方法：

| **方法** | **说明** |
| --- | --- |
| `Atomics.add()` | 原子加法，返回旧值 |
| `Atomics.sub()` | 原子减法，返回旧值 |
| `Atomics.load()` | 原子读取 |
| `Atomics.store()` | 原子写入，返回写入的值 |
| `Atomics.exchange()` | 原子替换，返回旧值 |
| `Atomics.compareExchange()` | CAS 操作：仅当当前值等于预期值时才替换 |
| `Atomics.wait()` | 阻塞线程直到被唤醒或超时（仅 Worker 中可用） |
| `Atomics.notify()` | 唤醒在指定位置等待的线程 |
| `Atomics.isLockFree()` | 检查指定字节大小的操作是否是无锁的 |

```jsx
const sab = new SharedArrayBuffer(4);
const view = new Int32Array(sab);

// 在 Worker 中
Atomics.add(view, 0, 1);       // 原子自增
Atomics.load(view, 0);         // 原子读取
Atomics.store(view, 0, 10);    // 原子写入
Atomics.compareExchange(view, 0, 10, 20); // CAS
```

---

## 20.2 跨上下文消息（XDM）

- **跨文档消息（Cross-Document Messaging，XDM）** 允许不同源的文档之间安全通信
- 核心方法：`postMessage(message, targetOrigin)`

### 发送消息

```jsx
// 向内嵌的 iframe 发送消息
const iframeWindow = document.getElementById("myIframe").contentWindow;
iframeWindow.postMessage("Hello!", "http://www.example.com");
```

- 第二个参数 `targetOrigin` 指定目标窗口的源，`"*"` 表示不限制（不推荐）

### 接收消息

```jsx
window.addEventListener("message", (event) => {
  // 验证来源
  if (event.origin === "http://www.example.com") {
    console.log(event.data);    // 接收到的数据
    console.log(event.source);  // 发送方窗口的代理
    event.source.postMessage("Received!", event.origin); // 回复
  }
});
```

- `event.data`：传递的消息数据
- `event.origin`：发送消息的文档源
- `event.source`：发送消息的窗口对象代理

---

## 20.3 Encoding API

用于在**字符串**和**定型数组**之间进行转换。

### TextEncoder

- 将字符串编码为 `Uint8Array`（UTF-8）

```jsx
const encoder = new TextEncoder();
const encoded = encoder.encode("Hello");
// Uint8Array [72, 101, 108, 108, 111]

// 编码到已有的缓冲区
const buf = new Uint8Array(5);
const result = encoder.encodeInto("Hello", buf);
// result: { read: 5, written: 5 }
```

### TextDecoder

- 将定型数组解码为字符串，支持多种字符编码

```jsx
const decoder = new TextDecoder("utf-8");
const decoded = decoder.decode(new Uint8Array([72, 101, 108, 108, 111]));
// "Hello"
```

- 支持 `stream` 选项，用于处理分块的数据流：

```jsx
const decoder = new TextDecoder();
decoder.decode(chunk1, { stream: true });
decoder.decode(chunk2, { stream: true });
decoder.decode(chunk3); // 最后一块不传 stream
```

---

## 20.4 File API 与 FileReader

### File 类型

- `File` 对象继承自 `Blob`，增加了 `name`、`lastModifiedDate` 等属性
- 通常通过 `<input type="file">` 的 `files` 属性获取

```jsx
const fileInput = document.getElementById("fileInput");
fileInput.addEventListener("change", (event) => {
  const files = event.target.files;
  for (let file of files) {
    console.log(file.name);           // 文件名
    console.log(file.size);           // 大小（字节）
    console.log(file.type);           // MIME 类型
    console.log(file.lastModifiedDate); // 最后修改时间
  }
});
```

### FileReader

- 异步读取文件内容，常用方法：

| **方法** | **说明** |
| --- | --- |
| `readAsText(file, encoding?)` | 以文本形式读取 |
| `readAsDataURL(file)` | 读取为 Data URL（Base64） |
| `readAsBinaryString(file)` | 读取为二进制字符串 |
| `readAsArrayBuffer(file)` | 读取为 ArrayBuffer |
- 事件：`load`、`error`、`progress`

```jsx
const reader = new FileReader();
reader.onload = (e) => {
  console.log(e.target.result); // 读取结果
};
reader.onprogress = (e) => {
  if (e.lengthComputable) {
    console.log(`${(e.loaded / e.total * 100).toFixed(0)}%`);
  }
};
reader.readAsText(file);
```

### FileReaderSync

- **同步版**的 FileReader，仅在 **Worker** 中可用
- 方法与 FileReader 相同，但直接返回结果而不是触发事件

### Blob 与对象 URL

- `Blob`（Binary Large Object）表示不可变的二进制数据
- 可以使用 `URL.createObjectURL(blob)` 创建临时 URL
- 使用完毕后应调用 `URL.revokeObjectURL(url)` 释放资源

```jsx
const url = URL.createObjectURL(file);
img.src = url;  // 直接用于显示图片
// 不再需要时
URL.revokeObjectURL(url);
```

---

## 20.5 媒体元素

HTML5 引入了 `<audio>` 和 `<video>` 元素，提供了统一的 JavaScript API。

### 常用属性

| **属性** | **说明** |
| --- | --- |
| `src` | 媒体资源 URL |
| `currentTime` | 当前播放位置（秒） |
| `duration` | 总时长（秒） |
| `paused` | 是否暂停 |
| `volume` | 音量（0.0 ~ 1.0） |
| `muted` | 是否静音 |
| `playbackRate` | 播放速率 |
| `buffered` | 已缓冲的时间范围 |

### 常用方法和事件

```jsx
const video = document.getElementById("myVideo");

video.play();     // 播放
video.pause();    // 暂停
video.load();     // 重新加载

// 常用事件
video.addEventListener("canplay", () => { /* 可以播放 */ });
video.addEventListener("timeupdate", () => { /* 播放位置更新 */ });
video.addEventListener("ended", () => { /* 播放结束 */ });
video.addEventListener("error", () => { /* 加载出错 */ });
```

### 检测编解码器支持

```jsx
const audio = document.createElement("audio");
if (audio.canPlayType("audio/mpeg")) {
  // 支持 MP3
}
// 返回值："probably"、"maybe" 或 ""（空字符串表示不支持）
```

---

## 20.6 原生拖放

### 拖放事件

**拖动元素**上触发的事件（按顺序）：

1. `dragstart` — 开始拖动
2. `drag` — 拖动过程中持续触发
3. `dragend` — 拖动结束

**放置目标**上触发的事件：

1. `dragenter` — 拖动元素进入目标区域
2. `dragover` — 在目标区域内移动（持续触发）
3. `dragleave` / `drop` — 离开目标区域 / 在目标区域释放

### 自定义放置目标

- 默认情况下，大多数元素不允许放置
- 需要阻止 `dragenter` 和 `dragover` 的默认行为：

```jsx
const dropTarget = document.getElementById("dropTarget");
dropTarget.addEventListener("dragover", (e) => e.preventDefault());
dropTarget.addEventListener("dragenter", (e) => e.preventDefault());
dropTarget.addEventListener("drop", (e) => {
  e.preventDefault();
  const data = e.dataTransfer.getData("text");
  console.log(data);
});
```

### dataTransfer 对象

- 用于在拖放操作中**传递数据**
- `setData(format, data)` — 设置数据
- `getData(format)` — 获取数据
- 支持的格式：`"text"`（等同于 `"text/plain"`）和 `"url"`（等同于 `"text/uri-list"`）

### dropEffect 与 effectAllowed

- `dropEffect`：放置目标上设置，表示允许的放置效果（`none`、`move`、`copy`、`link`）
- `effectAllowed`：拖动元素上设置，表示允许的拖动效果

### 可拖动性

```html
<!-- 使元素可拖动 -->
<div draggable="true">可拖动的内容</div>
```

- 图片和链接默认可拖动，其他元素需要设置 `draggable="true"`

---

## 20.7 Notifications API

- 用于向用户显示**系统通知**（桌面通知）
- 需要用户**授权**

### 请求权限

```jsx
Notification.requestPermission().then((permission) => {
  // permission: "granted" | "denied" | "default"
  if (permission === "granted") {
    new Notification("Hello!");
  }
});
```

### 创建通知

```jsx
const notification = new Notification("标题", {
  body: "通知正文内容",
  icon: "icon.png",
  tag: "unique-tag",   // 相同 tag 的通知会替换
  vibrate: [200, 100, 200] // 振动模式（移动设备）
});

// 事件
notification.onclick = () => { /* 点击通知 */ };
notification.onclose = () => { /* 通知关闭 */ };
notification.onerror = () => { /* 出错 */ };
notification.onshow = () => { /* 通知显示 */ };

// 关闭通知
notification.close();
```

### 权限状态

```jsx
Notification.permission; // "granted" | "denied" | "default"
```

---

## 20.8 Page Visibility API

- 用于检测页面是否对用户**可见**（如标签页是否在前台）

### 核心属性和事件

```jsx
// 页面是否隐藏
document.hidden;           // true / false（已废弃，建议用 visibilityState）
document.visibilityState;  // "visible" | "hidden"

// 监听可见性变化
document.addEventListener("visibilitychange", () => {
  if (document.visibilityState === "hidden") {
    // 页面不可见，可以暂停动画、停止轮询等
  } else {
    // 页面可见，恢复操作
  }
});
```

### 典型应用场景

- 页面不可见时暂停视频播放或动画
- 减少后台页面的网络请求和计时器频率
- 记录用户实际浏览页面的时间

---

## 20.9 Streams API

Streams API 用于处理**流式数据**，允许以小块的形式消费或生成数据，而不需要一次性加载到内存。

### 三种流类型

1. **可读流（ReadableStream）** — 数据的来源
2. **可写流（WritableStream）** — 数据的目的地
3. **转换流（TransformStream）** — 对数据进行中间转换

### 可读流

```jsx
// 创建自定义可读流
const readableStream = new ReadableStream({
  start(controller) {
    controller.enqueue("chunk1");
    controller.enqueue("chunk2");
    controller.close();
  }
});

// 消费可读流
const reader = readableStream.getReader();
async function read() {
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    console.log(value);
  }
}
```

### 可写流

```jsx
const writableStream = new WritableStream({
  write(chunk) {
    console.log("写入:", chunk);
  },
  close() {
    console.log("流关闭");
  }
});

const writer = writableStream.getWriter();
writer.write("数据块");
writer.close();
```

### 转换流

```jsx
const transformStream = new TransformStream({
  transform(chunk, controller) {
    controller.enqueue(chunk.toUpperCase());
  }
});
```

### 管道（Piping）

```jsx
// 将可读流通过转换流传输到可写流
readableStream
  .pipeThrough(transformStream)
  .pipeTo(writableStream);
```

---

## 20.10 计时 API

### Performance 接口

- `performance.now()` — 返回自页面导航开始的**高精度时间戳**（微秒级精度）
- 比 `Date.now()` 更适合性能测量

```jsx
const t0 = performance.now();
// 执行某些操作
const t1 = performance.now();
console.log(`耗时: ${t1 - t0} 毫秒`);
```

### Performance Timeline API

```jsx
// 创建自定义标记
performance.mark("startTask");
// ... 执行操作
performance.mark("endTask");

// 测量两个标记之间的时间
performance.measure("taskDuration", "startTask", "endTask");

// 获取测量结果
const measures = performance.getEntriesByName("taskDuration");
console.log(measures[0].duration); // 毫秒
```

### Navigation Timing API

- 提供页面导航过程中各阶段的时间点

```jsx
const timing = performance.getEntriesByType("navigation")[0];
console.log(timing.domContentLoadedEventEnd - timing.startTime); // DOMContentLoaded 耗时
console.log(timing.loadEventEnd - timing.startTime);             // 完全加载耗时
```

### Resource Timing API

```jsx
// 获取所有资源加载的计时信息
const resources = performance.getEntriesByType("resource");
resources.forEach((r) => {
  console.log(`${r.name}: ${r.duration}ms`);
});
```

---

## 20.11 Web 组件

Web 组件是一组用于创建**可复用、封装良好**的自定义 HTML 元素的技术。

### 三大核心技术

1. **Shadow DOM** — 封装组件内部的 DOM 和样式
2. **自定义元素（Custom Elements）** — 定义新的 HTML 标签
3. **HTML 模板（`<template>`）** — 定义可复用的 HTML 片段

### HTML 模板

```html
<template id="myTemplate">
  <style>
    p { color: red; }
  </style>
  <p>模板内容</p>
</template>
```

```jsx
const template = document.getElementById("myTemplate");
const clone = template.content.cloneNode(true);
document.body.appendChild(clone);
```

- `<template>` 中的内容不会被渲染，直到通过 JS 克隆并插入到 DOM 中
- 通过 `<slot>` 元素支持内容投射

### Shadow DOM

- 为元素附加一个**隔离的 DOM 子树**，其样式和结构与外部互不影响

```jsx
const host = document.getElementById("host");
const shadowRoot = host.attachShadow({ mode: "open" }); // 或 "closed"
shadowRoot.innerHTML = `
  <style>p { color: blue; }</style>
  <p>这是 Shadow DOM 中的内容</p>
`;
```

- `mode: "open"`：外部可以通过 `element.shadowRoot` 访问
- `mode: "closed"`：外部无法访问
- Shadow DOM 中的样式**不会泄露**到外部，外部样式也**不会影响**内部

### 自定义元素

```jsx
class MyComponent extends HTMLElement {
  constructor() {
    super();
    const shadow = this.attachShadow({ mode: "open" });
    shadow.innerHTML = `
      <style>
        :host { display: block; padding: 10px; }
        .title { font-weight: bold; }
      </style>
      <div class="title"><slot name="title">默认标题</slot></div>
      <div><slot>默认内容</slot></div>
    `;
  }

  // 生命周期回调
  connectedCallback() {
    // 元素被插入到 DOM 时调用
  }
  disconnectedCallback() {
    // 元素从 DOM 中移除时调用
  }
  attributeChangedCallback(name, oldVal, newVal) {
    // 被观察的属性变化时调用
  }
  static get observedAttributes() {
    return ["my-attr"]; // 需要观察的属性列表
  }
}

// 注册自定义元素
customElements.define("my-component", MyComponent);
```

```html
<!-- 使用 -->
<my-component>
  <span slot="title">自定义标题</span>
  <p>自定义内容</p>
</my-component>
```

---

## 20.12 Web Cryptography API

提供了一套**原生的密码学操作** API，性能优于纯 JS 实现。

### 生成随机值

```jsx
const array = new Uint8Array(16);
crypto.getRandomValues(array);
// 生成密码学安全的随机值
```

### SubtleCrypto

- 通过 `crypto.subtle` 访问，提供加密、解密、签名、验证等操作
- **所有方法都返回 Promise**

### 生成密钥

```jsx
const key = await crypto.subtle.generateKey(
  { name: "AES-GCM", length: 256 },
  true,         // 是否可导出
  ["encrypt", "decrypt"]
);
```

### 加密与解密

```jsx
const encoder = new TextEncoder();
const data = encoder.encode("Hello, World!");

// 加密
const iv = crypto.getRandomValues(new Uint8Array(12)); // 初始化向量
const encrypted = await crypto.subtle.encrypt(
  { name: "AES-GCM", iv },
  key,
  data
);

// 解密
const decrypted = await crypto.subtle.decrypt(
  { name: "AES-GCM", iv },
  key,
  encrypted
);
const decoded = new TextDecoder().decode(decrypted); // "Hello, World!"
```

### 摘要（哈希）

```jsx
const msgBuffer = new TextEncoder().encode("Hello");
const hashBuffer = await crypto.subtle.digest("SHA-256", msgBuffer);
const hashArray = Array.from(new Uint8Array(hashBuffer));
const hashHex = hashArray.map(b => b.toString(16).padStart(2, "0")).join("");
```

### 支持的算法

| **类别** | **算法** |
| --- | --- |
| 对称加密 | AES-CBC、AES-CTR、AES-GCM |
| 非对称加密 | RSA-OAEP |
| 签名 | RSASSA-PKCS1-v1_5、RSA-PSS、ECDSA、HMAC |
| 摘要 | SHA-1、SHA-256、SHA-384、SHA-512 |
| 密钥派生 | HKDF、PBKDF2 |
| 密钥协商 | ECDH |

---

## 本章小结

<aside>
📌

本章介绍了众多实用的 JavaScript API：

- **Atomics 与 SharedArrayBuffer**：在多线程间安全地共享和操作内存
- **跨上下文消息**：通过 `postMessage()` 实现不同源之间的安全通信
- **Encoding API**：字符串与二进制数据之间的编解码
- **File API**：访问用户文件系统中的文件并读取内容
- **媒体元素**：使用统一的 API 控制音视频播放
- **原生拖放**：通过 `dataTransfer` 在拖放操作间传递数据
- **Notifications API**：显示系统级桌面通知
- **Page Visibility API**：检测页面可见性以优化资源使用
- **Streams API**：以流的方式高效处理大量数据
- **计时 API**：精确测量页面和资源加载性能
- **Web 组件**：通过 Shadow DOM、自定义元素和模板创建可复用组件
- **Web Cryptography API**：原生高效的密码学操作
</aside>