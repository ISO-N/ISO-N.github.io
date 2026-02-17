---
title: 第13章 客户端检测
date: 2026-02-17 15:02:06
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
浏览器之间的差异是客户端开发中的常见问题。客户端检测的目标是**在不同浏览器环境下确保代码的正常运行**。本章介绍三种主要的客户端检测策略：能力检测、用户代理检测、软件与硬件检测。

> 💡 **核心原则**：优先使用**能力检测**，其次是**软件与硬件检测**，最后才考虑**用户代理字符串检测**。
> 

---

## 13.1 能力检测

能力检测（又称**特性检测**）是最常用、最推荐的客户端检测策略。它不关心浏览器是什么，只关心浏览器**能不能做某件事**。

### 基本模式

```jsx
if (object.propertyInQuestion) {
  // 使用 object.propertyInQuestion
}
```

例如，早期 IE5 不支持 `document.getElementById()`，可以这样检测：

```jsx
function getElement(id) {
  if (document.getElementById) {
    return document.getElementById(id);
  } else if (document.all) {
    return document.all[id];  // IE 早期方式
  } else {
    throw new Error("No way to retrieve element!");
  }
}
```

### 两个重要原则

1. **先检测达成目的最常用的特性**
    - 避免先测试不常用的特性，可以在大多数场景下减少不必要的检测
2. **必须检测实际要用到的特性**
    - 检测一个特性的存在不代表另一个相关特性也存在

### 更可靠的能力检测

仅仅检测属性是否存在还不够，有时候需要确认该属性确实是一个**函数**：

```jsx
// 不够可靠：有些对象的属性存在但不是函数
if (document.createElement) { ... }

// 更可靠：使用 typeof 检测
function isFunction(value) {
  return typeof value === "function";
}

if (typeof document.createElement === "function") {
  // 安全调用
}
```

<aside>
⚠️

在 IE 早期版本中，`typeof document.createElement` 返回 `"object"` 而非 `"function"`，因为 DOM 方法是以 COM 对象的形式实现的。现代浏览器已无此问题。

</aside>

### 基于能力检测进行浏览器分析

可以按照**能力集合**而非单个特性来检测，以确认浏览器是否符合某种基准。例如检测浏览器是否支持 Netscape 风格的插件：

```jsx
// 检测浏览器是否支持 Netscape 式的插件
let hasNSPlugins = !!(navigator.plugins && navigator.plugins.length);

// 检测浏览器是否具有 DOM Level 1 能力
let hasDOM1 = !!(document.getElementById && document.createElement
  && document.getElementsByTagName);
```

<aside>
💡

能力检测最好只用来判断**下一步该怎么做**，不要用来尝试判断用户使用的是哪种浏览器。

</aside>

---

## 13.2 用户代理检测

用户代理检测通过 `navigator.userAgent` 字符串来判断浏览器类型。由于历史原因，这个字符串非常复杂且不可靠，应作为**万不得已**的手段。

### 用户代理字符串的历史

用户代理字符串有一段著名的"伪装"历史：

1. **HTTP 初期**：Mosaic 的 UA 字符串格式为 `Mosaic/0.9`
2. **Netscape Navigator**：`Mozilla/Version`，"Mozilla" 意为 "Mosaic Killer"
3. **IE**：为了兼容 Netscape 的嗅探代码，伪装为 `Mozilla/2.0 (compatible; MSIE 3.0; Windows 95)`
4. **Gecko 引擎**（Firefox）：`Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:0.9.4) Gecko/20011128 Netscape6/6.2.1`
5. **WebKit 引擎**（Safari/Chrome）：在 UA 中同时包含 `like Gecko`
6. **Chrome**：加入 `Chrome/Version`，但保留 `Safari/Version`
7. **Opera**：多次修改 UA 字符串格式，甚至伪装为 IE 和 Firefox
8. **Edge**：早期包含 `Chrome/` 和 `Safari/` 前缀

<aside>
🔑

每个新浏览器都被迫在 UA 字符串中加入前代浏览器的标识，导致字符串越来越长、越来越不可信。这就是著名的**"UA 字符串伪装"**历史。

</aside>

### 用户代理字符串的解析

如果确实需要解析 UA 字符串，应按照特定顺序检测：

```jsx
class BrowserDetector {
  constructor() {
    // 测试条件从最特殊到最一般
    const ua = navigator.userAgent;

    this.isBrowser_Edge = /Edg/.test(ua);
    this.isBrowser_Chrome = /Chrome/.test(ua) && !this.isBrowser_Edge;
    this.isBrowser_Firefox = /Firefox/.test(ua);
    this.isBrowser_Safari = /Safari/.test(ua) && !this.isBrowser_Chrome
      && !this.isBrowser_Edge;
    this.isBrowser_Opera = /OPR/.test(ua);
    this.isBrowser_IE = /Trident/.test(ua);
  }
}
```

<aside>
⚠️

检测顺序很重要！由于浏览器 UA 字符串相互包含，必须**先检测更特殊的**（如 Edge），再检测更通用的（如 Chrome）。

</aside>

---

## 13.3 软件与硬件检测

现代浏览器提供了一系列 API，可以直接获取有关浏览器、操作系统、硬件等信息，不必依赖 UA 字符串。

### 13.3.1 识别浏览器与操作系统

#### `navigator.oscpu`

返回操作系统/CPU 架构信息：

```jsx
console.log(navigator.oscpu);
// 例如: "Windows NT 10.0; Win64; x64"
```

#### `navigator.vendor`

返回浏览器开发商信息：

```jsx
// Chrome: "Google Inc."
// Firefox: ""
// IE: undefined
```

#### `navigator.platform`

返回浏览器所在操作系统：

```jsx
console.log(navigator.platform);
// 例如: "Win32", "MacIntel", "Linux x86_64"
```

#### `screen.colorDepth` / `screen.pixelDepth`

返回显示器色深：

```jsx
console.log(screen.colorDepth); // 通常为 24
console.log(screen.pixelDepth); // 通常为 24
```

#### `screen.orientation`

返回屏幕方向信息：

```jsx
console.log(screen.orientation.type);
// "portrait-primary"    竖屏
// "portrait-secondary"  倒转竖屏
// "landscape-primary"   横屏
// "landscape-secondary" 倒转横屏
```

### 13.3.2 浏览器元数据

#### Geolocation API

获取用户地理位置：

```jsx
navigator.geolocation.getCurrentPosition(
  (position) => {
    console.log(position.coords.latitude);
    console.log(position.coords.longitude);
  },
  (error) => {
    console.log(error.message);
  }
);
```

- `getCurrentPosition()`：获取一次位置
- `watchPosition()`：持续监视位置变化
- 需要用户**授权**（HTTPS 环境下）

#### Connection Properties

`navigator.connection` 提供网络连接信息（Network Information API）：

```jsx
const conn = navigator.connection;

console.log(conn.downlink);         // 带宽估计（Mbps）
console.log(conn.effectiveType);    // "4g", "3g", "2g", "slow-2g"
console.log(conn.rtt);              // 估计往返时间（ms）
console.log(conn.saveData);         // 用户是否开启省流量模式
```

#### `navigator.onLine`

判断浏览器是否在线：

```jsx
if (navigator.onLine) {
  console.log("在线");
} else {
  console.log("离线");
}

// 监听在线/离线状态变化
window.addEventListener("online", () => console.log("恢复在线"));
window.addEventListener("offline", () => console.log("已离线"));
```

#### Battery Status API

获取设备电池信息：

```jsx
navigator.getBattery().then((battery) => {
  console.log(battery.charging);       // 是否在充电
  console.log(battery.chargingTime);   // 充满所需时间（秒）
  console.log(battery.dischargingTime);// 放电剩余时间（秒）
  console.log(battery.level);          // 电量百分比 0~1
});
```

### 13.3.3 硬件

#### 处理器核心数

```jsx
console.log(navigator.hardwareConcurrency);
// 例如: 8（表示8个逻辑核心）
```

#### 设备内存

```jsx
console.log(navigator.deviceMemory);
// 例如: 8（表示8GB，取最接近的2的幂次）
```

#### 最大触控点数

```jsx
console.log(navigator.maxTouchPoints);
// 0 表示不支持触控
// 触控设备通常返回正整数
```

---

## 本章小结

| **检测方式** | **优先级** | **说明** |
| --- | --- | --- |
| 能力检测 | ⭐ 最优先 | 检测浏览器是否支持特定功能，不关心浏览器类型 |
| 软件/硬件检测 | ⭐ 次优先 | 通过 `navigator` 和 `screen` API 获取环境信息 |
| 用户代理检测 | 最后手段 | 解析 UA 字符串，但不可靠，仅在万不得已时使用 |

<aside>
📌

**最佳实践**：优先使用**能力检测**来确保代码正常运行，只有在需要获取系统环境信息时才使用浏览器与硬件 API，尽量**避免依赖用户代理字符串**。

</aside>