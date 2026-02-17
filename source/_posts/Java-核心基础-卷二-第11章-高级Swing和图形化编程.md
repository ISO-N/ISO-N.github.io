---
title: 第11章 高级Swing和图形化编程
date: 2026-02-17 15:12:07
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 11.1 列表与表格

### JList 组件

- `JList` 用于显示一组可选择的项
- 支持三种选择模式：
    - `SINGLE_SELECTION`：只能选一个
    - `SINGLE_INTERVAL_SELECTION`：可选连续区间
    - `MULTIPLE_INTERVAL_SELECTION`：可选多个不连续项
- 使用 `ListModel` 接口管理数据，`DefaultListModel` 是常用实现
- 通过 `ListCellRenderer` 自定义单元格渲染

### JTable 组件

- `JTable` 用于以行列形式展示二维数据
- 核心接口 `TableModel`，常用 `DefaultTableModel` 和 `AbstractTableModel`
- **列模型**：`TableColumnModel` 控制列的显示顺序、宽度、渲染器和编辑器
- **单元格渲染与编辑**：
    - `TableCellRenderer`：控制单元格如何显示
    - `TableCellEditor`：控制单元格如何编辑
- **排序与过滤**：
    - `TableRowSorter` 支持点击列标题排序
    - `RowFilter` 可实现行过滤

---

## 11.2 树

### JTree 组件

- 用于展示层次结构数据（树形结构）
- 节点使用 `DefaultMutableTreeNode`，实现 `TreeNode` 接口
- 数据模型为 `TreeModel`，默认实现 `DefaultTreeModel`

### 树的操作

- **展开/折叠**：`expandPath()` / `collapsePath()`
- **选择**：`TreeSelectionModel` 控制选择模式
- **监听事件**：
    - `TreeSelectionListener`：监听节点选择变化
    - `TreeExpansionListener`：监听展开/折叠事件
    - `TreeModelListener`：监听数据模型变化

### 自定义渲染

- `DefaultTreeCellRenderer`：自定义节点图标和文字
- `TreeCellEditor`：自定义节点编辑

---

## 11.3 文本组件

### 文本组件层次

- `JTextComponent`（抽象基类）
    - `JTextField`：单行文本
    - `JTextArea`：多行纯文本
    - `JEditorPane`：支持 HTML 和 RTF 格式
    - `JTextPane`：支持样式化文本（带属性的文档）

### StyledDocument 与 AttributeSet

- `StyledDocument` 是 `JTextPane` 的文档模型
- `AttributeSet` 用于设置字体、颜色、大小等样式属性
- `SimpleAttributeSet` 和 `StyleConstants` 用于创建和修改属性

### 文档过滤器

- `DocumentFilter` 可在文本插入/删除前拦截修改
- 典型应用：限制输入长度、限制只能输入数字等

---

## 11.4 进度指示器

### JProgressBar

- 显示任务的完成进度
- 支持确定模式和不确定模式（`setIndeterminate(true)`）
- 可设置最小值、最大值和当前值
- 可选显示百分比字符串：`setStringPainted(true)`

### ProgressMonitor

- 弹出对话框形式的进度监控器
- 自动决定是否显示（短任务不弹窗）
- 提供取消按钮

### ProgressMonitorInputStream

- 监控输入流读取进度
- 自动弹出进度对话框

---

## 11.5 组件组织器

### 分隔面板 JSplitPane

- 将两个组件用可拖动的分隔条隔开
- 支持水平分隔（`HORIZONTAL_SPLIT`）和垂直分隔（`VERTICAL_SPLIT`）
- `setDividerLocation()` 设置分隔条位置
- `setOneTouchExpandable(true)` 显示快速展开/折叠按钮

### 选项卡面板 JTabbedPane

- 多个面板共享同一区域，通过标签页切换
- 支持设置标签位置（上/下/左/右）
- 支持添加图标、工具提示
- `addChangeListener()` 监听切换事件

### 桌面面板与内部框架

- `JDesktopPane`：容纳多个内部框架的容器
- `JInternalFrame`：类似窗口的轻量级组件，可在桌面面板内拖动、缩放、最小化
- 实现 MDI（Multiple Document Interface）风格

---

## 11.6 高级布局管理

### GridBagLayout

- 最灵活的标准布局管理器
- 使用 `GridBagConstraints` 控制每个组件：
    - `gridx`, `gridy`：组件所在网格位置
    - `gridwidth`, `gridheight`：组件跨越的网格数
    - `weightx`, `weighty`：额外空间的分配权重
    - `fill`：组件在网格中的填充方式（`NONE`, `HORIZONTAL`, `VERTICAL`, `BOTH`）
    - `anchor`：组件在网格中的对齐位置
    - `insets`：组件与网格边界的间距

### GroupLayout

- 专为 GUI 构建器设计（如 NetBeans）
- 分别在水平和垂直方向上定义组件组
- 支持顺序组（`SequentialGroup`）和并行组（`ParallelGroup`）

### 自定义布局管理器

- 实现 `LayoutManager` 或 `LayoutManager2` 接口
- 需实现 `layoutContainer()`、`preferredLayoutSize()` 等方法

---

## 11.7 高级 AWT

### 绘制图形 Graphics2D

- `Graphics2D` 是 `Graphics` 的子类，提供更强大的 2D 绘图能力
- **形状类（Shape）**：
    - `Line2D`, `Rectangle2D`, `Ellipse2D`, `Arc2D`
    - `QuadCurve2D`（二次曲线）, `CubicCurve2D`（三次曲线）
    - `GeneralPath` / `Path2D`：自定义路径
    - `Area`：支持形状的合并（`add`）、交集（`intersect`）、差集（`subtract`）、异或（`exclusiveOr`）

### 变换 AffineTransform

- 支持平移（`translate`）、旋转（`rotate`）、缩放（`scale`）、剪切（`shear`）
- 可组合多个变换
- `g2.setTransform()` / `g2.transform()` 应用变换

### 裁剪 Clipping

- `g2.setClip(Shape)` 限制绘图区域
- 只有裁剪区域内的内容会被渲染

### 透明与合成

- `AlphaComposite` 控制透明度和合成规则
- `g2.setComposite(AlphaComposite.getInstance(rule, alpha))`
- 常用规则：`SRC_OVER`（源覆盖目标）

### 渲染提示 RenderingHints

- 控制绘图质量与性能的权衡
- 常用提示：
    - `KEY_ANTIALIASING`：抗锯齿
    - `KEY_RENDERING`：渲染质量优先或速度优先
    - `KEY_TEXT_ANTIALIASING`：文字抗锯齿

---

## 11.8 图像处理

### 读取与写入图像

- [`ImageIO.read](http://ImageIO.read)(File/URL/InputStream)` 读取图像
- `ImageIO.write(image, format, File/OutputStream)` 写入图像
- 支持格式：PNG、JPEG、GIF、BMP 等

### BufferedImage

- 内存中的图像对象，可直接操作像素
- 创建：`new BufferedImage(width, height, type)`
- 常用类型：`TYPE_INT_ARGB`、`TYPE_INT_RGB`

### 图像过滤

- `BufferedImageOp` 接口，常用实现：
    - `AffineTransformOp`：几何变换（旋转、缩放）
    - `RescaleOp`：亮度/对比度调整
    - `LookupOp`：查找表变换（色调调整）
    - `ColorConvertOp`：颜色空间转换
    - `ConvolveOp`：卷积操作（模糊、锐化、边缘检测）

---

## 11.9 打印

### 打印 API

- `Printable` 接口：实现 `print(Graphics, PageFormat, pageIndex)` 方法
- `PrinterJob`：管理打印任务
    - `getPrinterJob()` 获取打印任务
    - `setPrintable()` 设置打印内容
    - `printDialog()` 显示打印对话框
    - `print()` 执行打印

### 多页打印

- 实现 `Pageable` 接口或使用 `Book` 类
- `Book` 可以包含不同格式的多个页面

### 打印服务 API

- `PrintServiceLookup` 查找可用打印机
- 支持属性查询（双面打印、颜色等）

---

## 11.10 剪贴板

### 数据传输机制

- `Clipboard` 类管理系统剪贴板
- `Transferable` 接口封装可传输的数据
- `DataFlavor` 描述数据类型

### 常用操作

- **文本传输**：`StringSelection` 用于文本的复制粘贴
- **图像传输**：自定义 `Transferable` 实现
- **本地对象传输**：在同一 JVM 内传输 Java 对象

---

## 11.11 拖放（Drag and Drop）

### 基本概念

- **拖动源（Drag Source）**：数据的来源组件
- **放置目标（Drop Target）**：接收数据的组件
- `TransferHandler` 是 Swing 中实现拖放的核心类

### TransferHandler

- `createTransferable()`：创建可传输数据
- `canImport()`：判断是否可以接收数据
- `importData()`：执行数据导入
- `getSourceActions()`：返回支持的操作（`COPY`, `MOVE`, `LINK`）

---

## 本章小结

<aside>
📌

本章涵盖了 Swing 的高级 UI 组件（列表、表格、树、文本）、布局管理器（GridBagLayout、GroupLayout）、2D 图形绘制（Graphics2D、变换、合成）、图像处理、打印 API，以及剪贴板和拖放机制。这些内容是构建功能丰富的桌面 GUI 应用的核心知识。

</aside>