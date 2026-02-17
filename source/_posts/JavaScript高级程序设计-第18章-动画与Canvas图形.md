---
title: 第18章 动画与Canvas图形
date: 2026-02-17 15:02:10
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 18.1 使用 requestAnimationFrame

### 18.1.1 早期定时动画

早期 JavaScript 动画依赖 `setInterval()` 或 `setTimeout()` 来控制帧率：

```jsx
// 早期方式：使用 setInterval
setInterval(function() {
  // 更新动画状态
}, 1000 / 60); // 大约每秒60帧
```

**问题：**

- 浏览器无法精确保证定时器的执行时间间隔
- 定时器精度受限（通常 4ms 以上）
- 与浏览器的重绘周期不同步，可能导致 **丢帧** 或 **卡顿**
- 页面在后台时仍会执行，浪费资源

### 18.1.2 时间间隔的问题

浏览器的重绘频率通常为 **60Hz**（即每 16.67ms 一帧），但不同设备可能不同。`setInterval` 无法自动适配屏幕刷新率，可能导致：

- **过度绘制**：动画更新的频率高于屏幕刷新率，多余的计算被浪费
- **不同步**：定时器回调可能落在两次重绘之间，导致视觉不流畅

### 18.1.3 requestAnimationFrame

`requestAnimationFrame()` 是浏览器提供的专门用于动画的 API，它会在 **下一次重绘之前** 调用回调函数：

```jsx
function updateProgress() {
  let div = document.getElementById("status");
  div.style.width = (parseInt(div.style.width, 10) + 5) + "%";
  if (div.style.width !== "100%") {
    requestAnimationFrame(updateProgress);
  }
}
requestAnimationFrame(updateProgress);
```

**优势：**

- 自动与浏览器重绘周期同步
- 页面不可见时自动暂停，节省 CPU 和电量
- 回调函数接收一个 `DOMHighResTimeStamp` 参数，表示下一次重绘的时间

### 18.1.4 cancelAnimationFrame

与 `setTimeout` 类似，`requestAnimationFrame` 返回一个 ID，可用 `cancelAnimationFrame()` 取消：

```jsx
let requestID = requestAnimationFrame(function() {
  console.log("Repaint!");
});
cancelAnimationFrame(requestID);
```

### 18.1.5 通过 requestAnimationFrame 节流

`requestAnimationFrame` 还可以用于 **节流**（throttling）频繁触发的事件，如 `scroll`、`resize`：

```jsx
let enabled = true;

function expensiveOperation() {
  console.log("Invoked at", Date.now());
}

window.addEventListener("scroll", () => {
  if (enabled) {
    enabled = false;
    requestAnimationFrame(() => {
      expensiveOperation();
      // 在重绘后重新启用
      enabled = true;
    });
  }
});
```

也可以配合定时器进一步限制执行频率：

```jsx
let enabled = true;
function expensiveOperation() {
  console.log("Invoked at", Date.now());
}
window.addEventListener("scroll", () => {
  if (enabled) {
    enabled = false;
    requestAnimationFrame(() => {
      expensiveOperation();
    });
    // 50ms 后重新启用
    setTimeout(() => enabled = true, 50);
  }
});
```

---

## 18.2 基本的画布功能

### 创建 Canvas

使用 `<canvas>` 标签创建画布，必须设置 `width` 和 `height`：

```html
<canvas id="drawing" width="200" height="200">
  A drawing of something.
</canvas>
```

> 标签内的文本是 **后备内容**，当浏览器不支持 `<canvas>` 时显示。
> 

### 获取绘图上下文

```jsx
let drawing = document.getElementById("drawing");

// 确保浏览器支持 <canvas>
if (drawing.getContext) {
  let context = drawing.getContext("2d"); // 2D 上下文
  // ...绑定绑定
}
```

### 导出画布图像

使用 `toDataURL()` 方法可以导出画布内容为图片：

```jsx
let drawing = document.getElementById("drawing");
if (drawing.getContext) {
  // 导出为 PNG（默认）
  let imgURI = drawing.toDataURL("image/png");

  // 显示图片
  let image = document.createElement("img");
  image.src = imgURI;
  document.body.appendChild(image);
}
```

---

## 18.3 2D 绘图上下文

### 18.3.1 填充和描边

2D 上下文的两个基本操作是 **填充**（fill）和 **描边**（stroke）：

```jsx
let context = drawing.getContext("2d");

// 填充颜色
context.fillStyle = "red";        // CSS 颜色值
context.fillStyle = "#ff0000";    // 十六进制
context.fillStyle = "rgba(255, 0, 0, 0.5)"; // 半透明

// 描边颜色
context.strokeStyle = "blue";
```

### 18.3.2 绘制矩形

矩形是唯一可以直接在 2D 上下文中绘制的形状：

```jsx
// fillRect(x, y, width, height) — 填充矩形
context.fillStyle = "#ff0000";
context.fillRect(10, 10, 50, 50);

// strokeRect(x, y, width, height) — 描边矩形
context.strokeStyle = "rgba(0, 0, 255, 0.5)";
context.strokeRect(30, 30, 50, 50);

// clearRect(x, y, width, height) — 清除矩形区域
context.clearRect(40, 40, 10, 10);
```

### 18.3.3 绘制路径

路径是 2D 绘图的基础，通过以下方法创建复杂形状：

```jsx
let context = drawing.getContext("2d");
context.beginPath(); // 开始新路径

// 绘制弧线/圆
// arc(x, y, radius, startAngle, endAngle, counterclockwise)
context.arc(100, 100, 99, 0, 2 * Math.PI, false);

// 移动绘图游标
context.moveTo(194, 100);

// 绘制内圆
context.arc(100, 100, 94, 0, 2 * Math.PI, false);

// 绘制分针
context.moveTo(100, 100);
context.lineTo(100, 15);

// 绘制时针
context.moveTo(100, 100);
context.lineTo(35, 100);

// 描边路径
context.stroke();
```

**常用路径方法：**

| **方法** | **说明** |
| --- | --- |
| `beginPath()` | 开始新路径 |
| `arc(x, y, r, start, end, ccw)` | 绘制弧线 |
| `arcTo(x1, y1, x2, y2, r)` | 从上一点到 (x2, y2) 绘制经过 (x1, y1) 的弧线 |
| `bezierCurveTo(c1x, c1y, c2x, c2y, x, y)` | 三次贝塞尔曲线 |
| `quadraticCurveTo(cx, cy, x, y)` | 二次贝塞尔曲线 |
| `lineTo(x, y)` | 画直线到指定点 |
| `moveTo(x, y)` | 移动游标（不画线） |
| `rect(x, y, w, h)` | 绘制矩形路径 |
| `closePath()` | 闭合路径（连接到起点） |

使用 `isPointInPath(x, y)` 判断某点是否在当前路径上：

```jsx
if (context.isPointInPath(100, 100)) {
  alert("Point (100, 100) is in the path.");
}
```

### 18.3.4 绘制文本

两个方法用于绘制文本：

- `fillText(text, x, y, maxWidth)` — 填充文本
- `strokeText(text, x, y, maxWidth)` — 描边文本

文本样式属性：

```jsx
context.font = "bold 14px Arial";      // 字体样式
context.textAlign = "center";           // 对齐方式：start, end, left, right, center
context.textBaseline = "middle";        // 基线：top, hanging, middle, alphabetic, ideographic, bottom
```

使用 `measureText()` 测量文本宽度：

```jsx
let fontSize = 100;
context.font = fontSize + "px Arial";

while (context.measureText("Hello world!").width > 140) {
  fontSize--;
  context.font = fontSize + "px Arial";
}

context.fillText("Hello world!", 10, 10);
```

### 18.3.5 变换

变换可以操作绘制到上下文中的图像，相关方法：

```jsx
// 旋转（弧度）
context.rotate(angle);

// 缩放
context.scale(scaleX, scaleY);

// 平移原点
context.translate(x, y);

// 变换矩阵 — 直接修改
context.transform(m1_1, m1_2, m2_1, m2_2, dx, dy);

// 重置并设置变换矩阵
context.setTransform(m1_1, m1_2, m2_1, m2_2, dx, dy);
```

**示例：用变换简化时钟绘制：**

```jsx
context.beginPath();

// 将原点移动到圆心
context.translate(100, 100);
context.rotate(1); // 旋转 1 弧度

// 绘制外圆（原点已在圆心，故 x=0, y=0）
context.arc(0, 0, 99, 0, 2 * Math.PI, false);

context.moveTo(94, 0);
context.arc(0, 0, 94, 0, 2 * Math.PI, false);

context.moveTo(0, 0);
context.lineTo(0, -85); // 分针

context.moveTo(0, 0);
context.lineTo(-65, 0); // 时针

context.stroke();
```

### 18.3.6 绘制图像

使用 `drawImage()` 将图像绘制到画布：

```jsx
let image = document.images[0];

// 基本绘制
context.drawImage(image, 10, 10);

// 指定大小
context.drawImage(image, 50, 10, 20, 30);

// 裁剪源图像的一部分绘制到画布
// drawImage(image, 源x, 源y, 源宽, 源高, 目标x, 目标y, 目标宽, 目标高)
context.drawImage(image, 0, 10, 50, 50, 0, 100, 40, 60);
```

> `drawImage()` 的第一个参数也可以是另一个 `<canvas>` 元素。
> 

### 18.3.7 阴影

2D 上下文支持为已有形状或路径生成阴影：

```jsx
context.shadowOffsetX = 5;     // 阴影 X 偏移
context.shadowOffsetY = 5;     // 阴影 Y 偏移
context.shadowBlur = 4;        // 模糊量（像素）
context.shadowColor = "rgba(0, 0, 0, 0.5)"; // 阴影颜色

// 之后绘制的形状自动带阴影
context.fillStyle = "red";
context.fillRect(10, 10, 50, 50);
```

### 18.3.8 渐变

**线性渐变：**

```jsx
// createLinearGradient(起始x, 起始y, 结束x, 结束y)
let gradient = context.createLinearGradient(30, 30, 70, 70);
gradient.addColorStop(0, "white");   // 起点颜色
gradient.addColorStop(1, "black");   // 终点颜色

context.fillStyle = gradient;
context.fillRect(30, 30, 50, 50);
```

**径向渐变：**

```jsx
// createRadialGradient(x1, y1, r1, x2, y2, r2)
let gradient = context.createRadialGradient(55, 55, 10, 55, 55, 30);
gradient.addColorStop(0, "white");
gradient.addColorStop(1, "black");

context.fillStyle = gradient;
context.fillRect(30, 30, 50, 50);
```

### 18.3.9 图案

使用 `createPattern()` 以图像创建重复图案：

```jsx
let image = document.images[0];

// 第二个参数: "repeat", "repeat-x", "repeat-y", "no-repeat"
let pattern = context.createPattern(image, "repeat");

context.fillStyle = pattern;
context.fillRect(10, 10, 150, 150);
```

> `createPattern()` 的第一个参数也可以是 `<video>` 元素或另一个 `<canvas>` 元素。
> 

### 18.3.10 图像数据

使用 `getImageData()` 获取原始图像数据：

```jsx
// getImageData(x, y, width, height)
let imageData = context.getImageData(10, 5, 50, 50);
```

返回的 `ImageData` 对象包含：

- `width` — 宽度
- `height` — 高度
- `data` — 一维数组，每 4 个值代表一个像素的 **R, G, B, A**

**灰度滤镜示例：**

```jsx
let imageData = context.getImageData(0, 0, image.width, image.height);
let data = imageData.data;

for (let i = 0, len = data.length; i < len; i += 4) {
  let red = data[i];
  let green = data[i + 1];
  let blue = data[i + 2];
  // alpha = data[i + 3]

  let average = Math.floor((red + green + blue) / 3);
  data[i] = average;       // red
  data[i + 1] = average;   // green
  data[i + 2] = average;   // blue
}

// 将修改后的数据写回画布
context.putImageData(imageData, 0, 0);
```

### 18.3.11 合成

**globalAlpha** — 全局透明度（0~1）：

```jsx
context.globalAlpha = 0.5; // 之后绘制的内容都半透明
```

**globalCompositeOperation** — 控制新绘制内容与已有内容的合成方式：

| **值** | **说明** |
| --- | --- |
| `source-over` | 默认值，新图形绘制在已有图形之上 |
| `source-in` | 仅显示新图形与已有图形重叠部分（新图形的颜色） |
| `source-out` | 仅显示新图形不与已有图形重叠的部分 |
| `source-atop` | 新图形仅在与已有图形重叠处绘制 |
| `destination-over` | 已有图形绘制在新图形之上 |
| `destination-in` | 仅保留与新图形重叠的已有图形部分 |
| `destination-out` | 仅保留不与新图形重叠的已有图形部分 |
| `destination-atop` | 已有图形仅在与新图形重叠处保留 |
| `lighter` | 重叠部分颜色值相加 |
| `copy` | 仅显示新图形 |
| `xor` | 重叠部分执行异或操作 |

---

## 18.4 WebGL

WebGL 是画布的 **3D 上下文**，基于 **OpenGL ES 2.0** 定义。

### 18.4.1 WebGL 上下文

```jsx
let drawing = document.getElementById("drawing");

let gl = drawing.getContext("webgl");
if (gl) {
  // 使用 WebGL
}
```

可以传入配置选项：

```jsx
let gl = drawing.getContext("webgl", {
  alpha: true,            // 是否有 alpha 通道（默认 true）
  depth: true,            // 是否有深度缓冲区（默认 true）
  stencil: false,         // 是否有模板缓冲区（默认 false）
  antialias: true,        // 是否抗锯齿（默认 true）
  premultipliedAlpha: true, // 是否预乘 alpha（默认 true）
  preserveDrawingBuffer: false // 是否保留绘制缓冲区（默认 false）
});
```

### 18.4.2 WebGL 基础

**常量与方法命名**基于 OpenGL ES 2.0，使用 `gl.` 前缀：

```jsx
// 清除颜色缓冲区
gl.clearColor(0, 0, 0, 1); // 黑色背景
gl.clear(gl.COLOR_BUFFER_BIT);
```

**视口与坐标：**

```jsx
// 设置视口 — viewport(x, y, width, height)
gl.viewport(0, 0, drawing.width, drawing.height);
```

WebGL 坐标系：

- 原点 (0, 0) 在画布 **中心**
- x 轴向右（-1 到 +1）
- y 轴向上（-1 到 +1）

**缓冲区：**

```jsx
// 创建缓冲区
let buffer = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([0, 0.5, 1]), gl.STATIC_DRAW);
```

`gl.bufferData()` 的第三个参数（用途提示）：

- `gl.STATIC_DRAW` — 数据加载一次，多次绘制
- `gl.STREAM_DRAW` — 数据加载一次，绘制少数几次
- `gl.DYNAMIC_DRAW` — 数据频繁修改和绘制

**错误处理：**

```jsx
// WebGL 不会主动抛出错误，需要手动检查
let errorCode = gl.getError();
// 常见错误码：
// gl.NO_ERROR (0)          — 无错误
// gl.INVALID_ENUM          — 无效枚举
// gl.INVALID_VALUE         — 无效值
// gl.INVALID_OPERATION     — 无效操作
// gl.OUT_OF_MEMORY         — 内存不足
// gl.CONTEXT_LOST_WEBGL    — 上下文丢失
```

### 18.4.3 着色器（Shader）

WebGL 中有两种着色器：

- **顶点着色器**（Vertex Shader）— 将 3D 顶点转换为 2D 坐标
- **片段着色器**（Fragment Shader）— 计算每个像素的颜色

着色器使用 **GLSL**（OpenGL Shading Language）编写：

```jsx
// 顶点着色器
let vertexShaderSource = `
  attribute vec2 aVertexPosition;
  void main() {
    gl_Position = vec4(aVertexPosition, 0.0, 1.0);
  }
`;

// 片段着色器
let fragmentShaderSource = `
  uniform vec4 uColor;
  void main() {
    gl_FragColor = uColor;
  }
`;
```

**编译和链接着色器：**

```jsx
// 创建着色器
let vertexShader = gl.createShader(gl.VERTEX_SHADER);
gl.shaderSource(vertexShader, vertexShaderSource);
gl.compileShader(vertexShader);

let fragmentShader = gl.createShader(gl.FRAGMENT_SHADER);
gl.shaderSource(fragmentShader, fragmentShaderSource);
gl.compileShader(fragmentShader);

// 创建程序并链接着色器
let program = gl.createProgram();
gl.attachShader(program, vertexShader);
gl.attachShader(program, fragmentShader);
gl.linkProgram(program);
gl.useProgram(program);
```

**向着色器传值：**

```jsx
// uniform 变量 — 对所有顶点相同
let uColor = gl.getUniformLocation(program, "uColor");
gl.uniform4fv(uColor, [0, 0, 0, 1]); // 黑色

// attribute 变量 — 每个顶点不同
let aVertexPosition = gl.getAttribLocation(program, "aVertexPosition");
gl.enableVertexAttribArray(aVertexPosition);
gl.vertexAttribPointer(aVertexPosition, 2, gl.FLOAT, false, 0, 0);
```

### 18.4.4 绘制

WebGL 只能绘制三种基本形状：**点**、**线**、**三角形**：

```jsx
// 绘制三角形
gl.drawArrays(gl.TRIANGLES, 0, 3);

// 绘制线段
gl.drawArrays(gl.LINES, 0, 2);

// 绘制点
gl.drawArrays(gl.POINTS, 0, 1);
```

`gl.drawArrays()` 的第一个参数常量：

- `gl.POINTS` — 点
- `gl.LINES` — 线段（每 2 个顶点一段）
- `gl.LINE_STRIP` — 线条（连续连接）
- `gl.LINE_LOOP` — 线圈（首尾相连）
- `gl.TRIANGLES` — 三角形（每 3 个顶点一个）
- `gl.TRIANGLE_STRIP` — 三角带
- `gl.TRIANGLE_FAN` — 三角扇

### 18.4.5 纹理

```jsx
let image = new Image();
image.src = "smile.gif";
image.onload = function() {
  let texture = gl.createTexture();
  gl.bindTexture(gl.TEXTURE_2D, texture);
  gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);

  // 上传图像数据
  gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, image);

  // 设置纹理参数
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
  gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
};
```

### 18.4.6 读取像素

类似 2D 上下文的 `getImageData()`：

```jsx
let pixels = new Uint8Array(25 * 25 * 4); // 25x25 区域
gl.readPixels(0, 0, 25, 25, gl.RGBA, gl.UNSIGNED_BYTE, pixels);
```

---

## 18.5 小结

<aside>
📝

- **`requestAnimationFrame`** 是创建平滑动画的最佳方式，它与浏览器重绑周期同步，并在页面不可见时自动暂停
- `<canvas>` 提供 **2D 绘图上下文**，支持矩形、路径、文本、图像、变换、渐变、阴影、图案和像素操作
- **WebGL** 是 `<canvas>` 的 3D 上下文，基于 OpenGL ES 2.0，使用 GLSL 着色器，可以进行复杂的 3D 图形渲染
- WebGL 绘制的基本单元是 **点、线、三角形**，通过着色器程序控制顶点位置和像素颜色
- `getImageData()` / `putImageData()`（2D）和 `readPixels()`（WebGL）允许直接操作像素数据
</aside>