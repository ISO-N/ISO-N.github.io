---
title: 第10章 图形用户界面程序设计
date: 2026-02-17 03:20:31
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷一
---
## 10.1 Java 用户界面工具包发展简史

- Java 1.0 提供了 **AWT（Abstract Window Toolkit）**，使用对等体（peer）方式将每个 UI 组件映射到目标平台的本地组件
- AWT 的问题：不同平台表现不一致，可用组件是各平台的**最小公分母**
- **Swing** 在 Java 1.2 引入，采用**纯 Java 绘制**组件，不依赖本地对等体，外观一致且功能更丰富
- Java 8 引入了 **JavaFX** 作为现代 GUI 工具包，但 Swing 仍然被广泛使用

<aside>
📌

本章聚焦 **Swing** 框架的核心概念：窗口、绘图、事件处理。Swing 组件的详细用法将在第 11 章展开。

</aside>

## 10.2 显示窗体

### 创建窗体（JFrame）

- Swing 的顶层窗口称为**窗体（frame）**，对应类 `JFrame`
- `JFrame` 是少数几个不绘制在画布上的 Swing 组件之一，它由操作系统的窗口管理器绘制

```java
import javax.swing.*;

public class SimpleFrameTest {
    public static void main(String[] args) {
        EventQueue.invokeLater(() -> {
            var frame = new JFrame("My Frame");
            frame.setSize(400, 300);
            frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            frame.setVisible(true);
        });
    }
}
```

### 关键要点

- 所有 Swing 组件必须在**事件分派线程（Event Dispatch Thread, EDT）**中配置，使用 `EventQueue.invokeLater()` 确保线程安全
- `setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE)` 使关闭窗口时程序退出（默认行为仅隐藏窗口）
- `setVisible(true)` 使窗体可见

### 窗体属性

| **方法** | **说明** |
| --- | --- |
| `setSize(width, height)` | 设置窗体大小（像素） |
| `setLocation(x, y)` | 设置窗体左上角位置 |
| `setBounds(x, y, w, h)` | 同时设置位置和大小 |
| `setLocationRelativeTo(null)` | 窗体居中显示 |
| `setTitle(String)` | 设置标题栏文字 |
| `setResizable(boolean)` | 是否允许用户调整大小 |
| `setIconImage(Image)` | 设置窗体图标 |

### 确定合适的窗体大小

- 使用 `Toolkit` 获取**屏幕分辨率**，根据分辨率动态设置窗体大小

```java
Toolkit kit = Toolkit.getDefaultToolkit();
Dimension screenSize = kit.getScreenSize();
int screenWidth = screenSize.width;
int screenHeight = screenSize.height;
frame.setSize(screenWidth / 2, screenHeight / 2);
```

## 10.3 在组件中显示信息

### 不要在 JFrame 上直接绘图

- 窗体由标题栏、菜单栏和**内容窗格（content pane）**组成
- 应该在**内容窗格**中添加组件来显示内容，而不是直接在窗体上绘图

### 自定义 JComponent

- 继承 `JComponent` 或 `JPanel`，覆盖 `paintComponent` 方法来自定义绘图

```java
class MyComponent extends JComponent {
    @Override
    public void paintComponent(Graphics g) {
        // 绘图代码
        g.drawString("Hello, World!", 75, 100);
    }
    
    @Override
    public Dimension getPreferredSize() {
        return new Dimension(300, 200);
    }
}
```

<aside>
⚠️

**不要自己调用 `paintComponent`！** 该方法由系统在需要重绘时自动调用（如窗口被遮挡后恢复、窗口大小改变等）。如需手动触发重绘，调用 `repaint()` 方法。

</aside>

### 将组件添加到窗体

```java
var component = new MyComponent();
frame.add(component);
frame.pack();  // 根据组件首选大小自动调整窗体大小
```

## 10.4 2D 图形

### Graphics2D

- `paintComponent` 的参数 `Graphics` 实际上是 `Graphics2D` 的实例
- `Graphics2D` 提供了更强大的 2D 绘图能力

```java
public void paintComponent(Graphics g) {
    var g2 = (Graphics2D) g;  // 向下转型
    // 使用 g2 进行绘制
}
```

### 几何形状类（java.awt.geom）

| **类** | **说明** |
| --- | --- |
| `Line2D.Double` | 线段 |
| `Rectangle2D.Double` | 矩形 |
| `Ellipse2D.Double` | 椭圆 |
| `Arc2D.Double` | 圆弧 |
| `Point2D.Double` | 点 |
- 每个形状类都有 `Double` 和 `Float` 两个内部类版本，通常使用 `Double`

### 绘制与填充

```java
// 绘制矩形轮廓
var rect = new Rectangle2D.Double(100, 100, 200, 150);
g2.draw(rect);

// 填充椭圆
var ellipse = new Ellipse2D.Double(100, 100, 200, 150);
g2.fill(ellipse);

// 绘制线段
g2.draw(new Line2D.Double(0, 0, 100, 200));
```

### Float vs Double

- `Rectangle2D.Float` 和 `Rectangle2D.Double` 都继承自 `Rectangle2D`
- 方法参数和返回值类型统一为 `double`，编写通用代码时使用父类 `Rectangle2D` 引用

## 10.5 颜色

### 使用 Color 类

```java
g2.setPaint(Color.RED);        // 设置绘制颜色
g2.fill(rect);                 // 用当前颜色填充

g2.setPaint(new Color(0, 128, 255));  // 自定义 RGB 颜色
```

### 预定义颜色常量

| **常量** | **颜色** | **常量** | **颜色** |
| --- | --- | --- | --- |
| [`Color.BLACK`](http://Color.BLACK) | 黑 | `Color.WHITE` | 白 |
| [`Color.RED`](http://Color.RED) | 红 | [`Color.GREEN`](http://Color.GREEN) | 绿 |
| [`Color.BLUE`](http://Color.BLUE) | 蓝 | `Color.YELLOW` | 黄 |
| `Color.CYAN` | 青 | `Color.MAGENTA` | 品红 |
| [`Color.ORANGE`](http://Color.ORANGE) | 橙 | `Color.GRAY` | 灰 |

### 设置组件背景色

```java
var panel = new JPanel();
panel.setBackground(Color.LIGHT_GRAY);  // 设置背景颜色
```

## 10.6 字体

### 创建和使用字体

```java
var font = new Font("SansSerif", Font.BOLD, 14);
g2.setFont(font);
g2.drawString("Hello!", 100, 100);
```

### 字体样式常量

- `Font.PLAIN` — 普通
- `Font.BOLD` — 粗体
- `Font.ITALIC` — 斜体
- `Font.BOLD + Font.ITALIC` — 粗斜体

### 逻辑字体名

Java 保证以下 **5 种逻辑字体**在所有平台可用：

- `SansSerif`、`Serif`、`Monospaced`、`Dialog`、`DialogInput`

### 获取字体度量（FontMetrics）

- 用于精确控制文本的位置和对齐方式

```java
FontRenderContext context = g2.getFontRenderContext();
Rectangle2D bounds = font.getStringBounds("Hello", context);
double width = bounds.getWidth();      // 文本宽度
double height = bounds.getHeight();    // 文本高度
double ascent = -bounds.getY();        // 上坡度（基线到顶部）
```

### 文本居中绘制

```java
// 在组件中居中绘制文本
String message = "Hello, World!";
FontRenderContext context = g2.getFontRenderContext();
Rectangle2D bounds = font.getStringBounds(message, context);

double x = (getWidth() - bounds.getWidth()) / 2;
double y = (getHeight() - bounds.getHeight()) / 2 - bounds.getY();
g2.drawString(message, (int) x, (int) y);
```

## 10.7 图像

### 读取和显示图像

```java
// 读取图像文件
var image = new ImageIcon("blue-ball.gif").getImage();

// 在 paintComponent 中绘制图像
g2.drawImage(image, x, y, null);
```

### 平铺图像

```java
public void paintComponent(Graphics g) {
    for (int i = 0; i * imageWidth <= getWidth(); i++) {
        for (int j = 0; j * imageHeight <= getHeight(); j++) {
            g.drawImage(image, i * imageWidth, j * imageHeight, null);
        }
    }
}
```

## 10.8 事件处理

### 事件处理基本概念

- Java 使用**委派事件模型（delegation event model）**：事件源将事件通知发送给注册的**事件监听器**
- 事件监听器是一个实现了特定**监听器接口**的对象
- 核心流程：**事件源 → 事件对象 → 事件监听器**

### ActionEvent 示例

```java
// 按钮点击事件
var button = new JButton("Click Me");
button.addActionListener(event -> {
    System.out.println("Button clicked!");
});
```

### 使用 Lambda 表达式简化监听器

```java
// 传统写法：匿名内部类
button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent event) {
        System.out.println("Clicked!");
    }
});

// 简化写法：Lambda 表达式（推荐）
button.addActionListener(event -> System.out.println("Clicked!"));
```

### 多个事件源共享监听器

```java
var listener = new ActionListener() {
    public void actionPerformed(ActionEvent event) {
        // 通过 getSource() 区分事件源
        Object source = event.getSource();
        if (source == button1) { /* ... */ }
        else if (source == button2) { /* ... */ }
    }
};
button1.addActionListener(listener);
button2.addActionListener(listener);
```

### 改变观感（Look and Feel）

```java
// 切换为系统默认观感
String laf = UIManager.getSystemLookAndFeelClassName();
UIManager.setLookAndFeel(laf);
SwingUtilities.updateComponentTreeUI(frame);
```

## 10.9 适配器类

### 窗口事件与 WindowListener

- `WindowListener` 接口有 **7 个方法**需要实现（窗口打开、关闭、激活等）
- 当只需要处理其中一两个事件时，实现所有方法很繁琐

### 使用适配器类简化

- **适配器类**为接口中的每个方法提供了空实现，只需覆盖感兴趣的方法

```java
// 使用 WindowAdapter 只处理关闭事件
frame.addWindowListener(new WindowAdapter() {
    @Override
    public void windowClosing(WindowEvent e) {
        System.exit(0);
    }
});
```

### 常用适配器类

| **监听器接口** | **适配器类** | **典型用途** |
| --- | --- | --- |
| `WindowListener` | `WindowAdapter` | 窗口打开、关闭、最小化等 |
| `MouseListener` | `MouseAdapter` | 鼠标点击、按下、释放 |
| `MouseMotionListener` | `MouseMotionAdapter` | 鼠标移动、拖拽 |
| `KeyListener` | `KeyAdapter` | 键盘按下、释放、输入 |

## 10.10 动作（Action）

### Action 接口

- `Action` 接口扩展了 `ActionListener`，可以将同一个动作绑定到**按钮、菜单项和快捷键**
- 使用 `AbstractAction` 便捷实现

```java
var exitAction = new AbstractAction("Exit") {
    public void actionPerformed(ActionEvent event) {
        System.exit(0);
    }
};
exitAction.putValue(Action.SHORT_DESCRIPTION, "退出程序");

// 同一个 Action 可绑定到多个组件
var menuItem = new JMenuItem(exitAction);
var button = new JButton(exitAction);
```

### 键盘快捷键（KeyStroke）

- 通过 **输入映射（InputMap）**和**动作映射（ActionMap）**绑定快捷键

```java
// 绑定 Ctrl+Q 到退出动作
InputMap imap = panel.getInputMap(JComponent.WHEN_IN_FOCUSED_WINDOW);
imap.put(KeyStroke.getKeyStroke("ctrl Q"), "quit");
ActionMap amap = panel.getActionMap();
amap.put("quit", exitAction);
```

## 10.11 鼠标事件

### MouseListener 方法

| **方法** | **触发时机** |
| --- | --- |
| `mousePressed` | 鼠标按钮被按下 |
| `mouseReleased` | 鼠标按钮被释放 |
| `mouseClicked` | 按下并释放（完整点击） |
| `mouseEntered` | 鼠标进入组件区域 |
| `mouseExited` | 鼠标离开组件区域 |

### MouseMotionListener 方法

- `mouseMoved` — 鼠标移动（未按下按钮）
- `mouseDragged` — 鼠标拖拽（按下按钮并移动）

### 获取鼠标信息

```java
panel.addMouseListener(new MouseAdapter() {
    public void mouseClicked(MouseEvent e) {
        int x = e.getX();          // 鼠标 X 坐标
        int y = e.getY();          // 鼠标 Y 坐标
        int clicks = e.getClickCount();  // 点击次数（双击检测）
        
        if (SwingUtilities.isLeftMouseButton(e)) {
            // 左键点击
        }
    }
});
```

### 设置光标样式

```java
panel.setCursor(Cursor.getPredefinedCursor(Cursor.CROSSHAIR_CURSOR));
```

## 10.12 AWT 事件继承层次

### 事件分类

Java AWT 事件分为**低级事件**和**语义事件**两大类：

### 低级事件（与具体操作相关）

- `KeyEvent` — 键盘按键
- `MouseEvent` — 鼠标操作
- `MouseWheelEvent` — 鼠标滚轮
- `FocusEvent` — 组件获得 / 失去焦点
- `WindowEvent` — 窗口状态变化

### 语义事件（与业务含义相关）

- `ActionEvent` — 按钮点击、回车确认等
- `AdjustmentEvent` — 滚动条调整
- `ItemEvent` — 选项选中 / 取消
- `ChangeEvent` — 状态变化

<aside>
💡

**设计原则：** 优先监听**语义事件**（如 `ActionEvent`）而非低级事件。语义事件更抽象，代码不会因交互方式变化而受影响。例如，监听按钮的 `ActionEvent` 比监听鼠标点击更合理，因为用户也可能通过键盘触发按钮。

</aside>

---

<aside>
✅

**本章小结：**

- Swing 使用纯 Java 绘制组件，核心顶层窗口是 **`JFrame`**
- 自定义绘图需继承 `JComponent`，覆盖 **`paintComponent`** 方法
- **`Graphics2D`** 提供丰富的 2D 图形绘制能力（矩形、椭圆、线段等）
- 事件处理采用**委派模型**：事件源 → 事件对象 → 监听器
- **Lambda 表达式**大幅简化了事件监听器的编写
- **适配器类**（如 `WindowAdapter`）避免实现接口中所有方法
- **Action 接口**统一管理按钮、菜单和快捷键的动作
- 优先使用**语义事件**，代码更健壮、更易维护
</aside>