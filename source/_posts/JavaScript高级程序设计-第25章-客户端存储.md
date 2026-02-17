---
title: 第25章 客户端存储
date: 2026-02-17 15:02:16
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
本章介绍了浏览器端持久化数据的三种主要机制：**cookie**、**Web Storage**（`sessionStorage` / `localStorage`）和 **IndexedDB**。

---

## 25.1 cookie

### 25.1.1 概述

- cookie 最初由网景公司发明，用于在客户端存储**会话信息**。
- 服务器通过响应头 `Set-Cookie` 向浏览器写入 cookie；浏览器在后续请求中通过 `Cookie` 请求头将其回传。
- cookie 本质上是绑定到**特定域**的小段文本数据，随每次 HTTP 请求发送，因此会带来额外的网络开销。

### 25.1.2 cookie 的构成

每个 cookie 包含以下可选参数：

| 参数 | 说明 |
| --- | --- |
| **名称（name）** | 唯一标识 cookie 的名称，不区分大小写，需 URL 编码 |
| **值（value）** | cookie 存储的字符串值，需 URL 编码 |
| **域（domain）** | cookie 有效的域，默认为设置它的页面所在域 |
| **路径（path）** | 限制 cookie 只在该路径下发送 |
| **过期时间（expires / max-age）** | `expires` 为具体日期（GMT 格式），`max-age` 为秒数。默认在会话结束时删除 |
| **安全标志（secure）** | 设置后只在 HTTPS 连接中发送 |
| **HttpOnly** | 设置后 JavaScript 无法通过 `document.cookie` 访问，防止 XSS 攻击 |
| **SameSite** | 控制跨站请求时是否发送 cookie，值为 `Strict`、`Lax` 或 `None` |

### 25.1.3 JavaScript 中的 cookie

- 通过 `document.cookie` 读取和写入：

```jsx
// 读取所有 cookie（返回分号分隔的 name=value 对）
console.log(document.cookie);

// 写入一个 cookie
document.cookie = encodeURIComponent("name") + "=" +
                  encodeURIComponent("Nicholas") +
                  "; domain=.example.com; path=/; secure";
```

- 读取时返回**所有**当前可见的 cookie 字符串，需要自行解析。
- 写入时不会覆盖已有 cookie（除非同名同域同路径）。

### 25.1.4 封装 cookie 操作

<aside>
💡

书中推荐封装一个 `CookieUtil` 工具对象，便于读取、写入和删除 cookie。

</aside>

```jsx
class CookieUtil {
  static get(name) {
    let cookieName = encodeURIComponent(name) + "=",
        cookieStart = document.cookie.indexOf(cookieName),
        cookieValue = null;
    if (cookieStart > -1) {
      let cookieEnd = document.cookie.indexOf(";", cookieStart);
      if (cookieEnd === -1) {
        cookieEnd = document.cookie.length;
      }
      cookieValue = decodeURIComponent(
        document.cookie.substring(cookieStart + cookieName.length, cookieEnd)
      );
    }
    return cookieValue;
  }

  static set(name, value, expires, path, domain, secure) {
    let cookieText = encodeURIComponent(name) + "=" +
                     encodeURIComponent(value);
    if (expires instanceof Date) {
      cookieText += "; expires=" + expires.toGMTString();
    }
    if (path) cookieText += "; path=" + path;
    if (domain) cookieText += "; domain=" + domain;
    if (secure) cookieText += "; secure";
    document.cookie = cookieText;
  }

  static unset(name, path, domain, secure) {
    CookieUtil.set(name, "", new Date(0), path, domain, secure);
  }
}
```

### 25.1.5 cookie 的限制

- **数量限制**：大多数浏览器每个域限制 **≤ 50** 个 cookie。
- **大小限制**：单个 cookie 大小不超过 **4096 字节**（约 4 KB），包含名称、值和属性。
- cookie 是同步读写的，不适合存储大量数据。

### 25.1.6 子 cookie

- 为了绕过数量限制，可以在一个 cookie 值中使用 `&` 分隔多个子键值对：

```
data=name=Nicholas&book=Professional+JavaScript
```

- 需要自行编写解析和序列化子 cookie 的逻辑。

---

## 25.2 Web Storage

Web Storage 定义了两种在客户端存储数据的对象：`sessionStorage` 和 `localStorage`。它们的 API 完全相同，区别在于**生命周期和作用域**。

### 25.2.1 Storage 类型

`Storage` 对象提供以下方法和属性：

```jsx
// 存储数据
storage.setItem("key", "value");

// 读取数据
let value = storage.getItem("key");

// 删除数据
storage.removeItem("key");

// 清除所有数据
storage.clear();

// 获取存储的键值对数量
console.log(storage.length);

// 通过索引获取键名
let key = storage.key(0);
```

<aside>
⚠️

Storage 只能存储**字符串**。存储对象或数组时，需要先用 `JSON.stringify()` 序列化，读取时再用 `JSON.parse()` 反序列化。

</aside>

### 25.2.2 sessionStorage

- 数据只在**当前会话（浏览器标签页）** 中有效，关闭标签页后数据清除。
- 同源的不同标签页之间**不共享** `sessionStorage`。
- 适用于存储只需在当前会话中使用的临时数据。

```jsx
// 存储
sessionStorage.setItem("name", "Nicholas");

// 读取
let name = sessionStorage.getItem("name");

// 遍历
for (let i = 0; i < sessionStorage.length; i++) {
  let key = sessionStorage.key(i);
  let value = sessionStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### 25.2.3 localStorage

- 数据**持久保存**，除非手动删除或用户清除浏览器数据，否则一直存在。
- 同源的所有标签页和窗口**共享** `localStorage`。
- 适用于需要跨会话保留的数据（如用户偏好设置）。

```jsx
// 存储
localStorage.setItem("theme", "dark");

// 读取
let theme = localStorage.getItem("theme");

// 删除
localStorage.removeItem("theme");
```

### 25.2.4 storage 事件

- 当 `localStorage` 数据发生变化时，同源的**其他**窗口/标签页会触发 `storage` 事件：

```jsx
window.addEventListener("storage", (event) => {
  console.log(event.domain);  // 变化的存储域
  console.log(event.key);     // 变化的键
  console.log(event.newValue);// 新值
  console.log(event.oldValue);// 旧值
});
```

<aside>
📌

`storage` 事件不会在引起变化的当前页面触发，只在同源的其他页面触发。

</aside>

### 25.2.5 存储限制

- 大多数浏览器对每个源（origin）的 Web Storage 限制为 **5 MB**。
- 超出限制时会抛出错误（通常是 `QuotaExceededError`）。
- 与 cookie 不同，Web Storage 数据**不会**随 HTTP 请求自动发送给服务器。

---

## 25.3 IndexedDB

### 25.3.1 概述

- IndexedDB 是浏览器中的**结构化、事务型、异步**的客户端数据库。
- 几乎所有操作都是**异步**的，通过请求（`IDBRequest`）对象和事件回调完成。
- 适合存储大量结构化数据，功能远超 cookie 和 Web Storage。

### 25.3.2 打开/创建数据库

```jsx
let db;
const request = indexedDB.open("myDatabase", 1); // 数据库名, 版本号

request.onerror = (event) => {
  console.error("数据库打开失败");
};

request.onsuccess = (event) => {
  db = event.target.result;
  console.log("数据库打开成功");
};

// 版本号变化时触发（首次创建或升级）
request.onupgradeneeded = (event) => {
  db = event.target.result;
  // 在这里创建对象存储和索引
};
```

### 25.3.3 对象存储（Object Store）

- 对象存储是 IndexedDB 中存放数据的"表"。
- 每条记录都需要一个**键**（主键），可以是属性路径（`keyPath`）或自动递增（`autoIncrement`）。

```jsx
request.onupgradeneeded = (event) => {
  const db = event.target.result;

  // 创建对象存储，以 "id" 为主键
  if (!db.objectStoreNames.contains("users")) {
    const store = db.createObjectStore("users", { keyPath: "id" });
    // 创建索引，便于通过 "name" 字段查找
    store.createIndex("name", "name", { unique: false });
    store.createIndex("email", "email", { unique: true });
  }
};
```

### 25.3.4 事务（Transaction）

- 所有数据读写操作都必须在**事务**中执行。
- 事务有三种模式：`"readonly"`、`"readwrite"` 和 `"versionchange"`（仅在 `onupgradeneeded` 中自动使用）。

```jsx
// 创建读写事务
const transaction = db.transaction(["users"], "readwrite");
const store = transaction.objectStore("users");

// 添加数据
store.add({ id: 1, name: "Nicholas", email: "nicholas@example.com" });

// 读取数据
const getRequest = store.get(1);
getRequest.onsuccess = () => {
  console.log(getRequest.result);
};

// 更新数据（put 会覆盖同 key 的记录）
store.put({ id: 1, name: "Greg", email: "greg@example.com" });

// 删除数据
store.delete(1);

// 事务完成回调
transaction.oncomplete = () => {
  console.log("事务完成");
};
```

### 25.3.5 游标（Cursor）

- 游标用于**遍历**对象存储中的所有记录。

```jsx
const transaction = db.transaction(["users"], "readonly");
const store = transaction.objectStore("users");
const cursorRequest = store.openCursor();

cursorRequest.onsuccess = (event) => {
  const cursor = event.target.result;
  if (cursor) {
    console.log(`Key: ${cursor.key}, Value:`, cursor.value);
    cursor.continue(); // 移动到下一条记录
  } else {
    console.log("遍历完成");
  }
};
```

**游标方向**：`openCursor()` 可传入方向参数：

| 方向 | 说明 |
| --- | --- |
| `"next"` | 从前往后（默认），允许重复键 |
| `"nextunique"` | 从前往后，跳过重复键 |
| `"prev"` | 从后往前 |
| `"prevunique"` | 从后往前，跳过重复键 |

### 25.3.6 键范围（Key Range）

- 使用 `IDBKeyRange` 限定游标查询的范围：

```jsx
// 只取 key = 1 的记录
const onlyRange = IDBKeyRange.only(1);

// key >= 2
const lowerRange = IDBKeyRange.lowerBound(2);

// key > 2（不包含 2）
const lowerRangeOpen = IDBKeyRange.lowerBound(2, true);

// key <= 10
const upperRange = IDBKeyRange.upperBound(10);

// 2 <= key <= 10
const boundRange = IDBKeyRange.bound(2, 10);

// 2 < key < 10（不包含边界）
const boundRangeOpen = IDBKeyRange.bound(2, 10, true, true);

// 使用键范围打开游标
store.openCursor(boundRange).onsuccess = (event) => {
  const cursor = event.target.result;
  if (cursor) {
    console.log(cursor.value);
    cursor.continue();
  }
};
```

### 25.3.7 索引（Index）

- 索引允许通过**非主键**属性高效查找记录。

```jsx
// 在 onupgradeneeded 中创建索引
store.createIndex("nameIndex", "name", { unique: false });

// 通过索引查询
const transaction = db.transaction(["users"], "readonly");
const store = transaction.objectStore("users");
const index = store.index("nameIndex");

const request = index.get("Nicholas");
request.onsuccess = () => {
  console.log(request.result);
};

// 通过索引打开游标
index.openCursor().onsuccess = (event) => {
  const cursor = event.target.result;
  if (cursor) {
    console.log(cursor.value);
    cursor.continue();
  }
};
```

### 25.3.8 并发与版本管理

- 如果一个页面尝试打开更高版本的数据库，而另一个页面还持有旧版本连接，则会触发 `onblocked` 事件。
- 应监听 `versionchange` 事件以便及时关闭连接：

```jsx
db.onversionchange = () => {
  db.close();
  console.log("数据库版本已更新，请刷新页面");
};
```

### 25.3.9 IndexedDB 的限制

- 存储空间因浏览器而异，通常为**数百 MB 甚至更多**，远大于 Web Storage。
- 受**同源策略**限制，每个源拥有独立的数据库。
- 不支持 SQL 查询语言，查询能力相对有限，需要通过索引和游标手动实现。

---

## 25.4 小结

| 特性 | **cookie** | **Web Storage** | **IndexedDB** |
| --- | --- | --- | --- |
| 存储大小 | ~4 KB | ~5 MB | 数百 MB+ |
| 数据类型 | 字符串 | 字符串 | 结构化数据（对象） |
| 是否随请求发送 | ✅ 自动发送 | ❌ | ❌ |
| API 风格 | 字符串操作 | 同步 key-value | 异步、事务型 |
| 生命周期 | 可设过期时间 | session 或永久 | 永久（手动删除） |
| 适用场景 | 会话标识、少量配置 | 用户偏好、轻量缓存 | 大量结构化离线数据 |