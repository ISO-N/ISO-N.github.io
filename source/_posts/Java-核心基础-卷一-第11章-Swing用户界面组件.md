---
title: 第11章 Swing用户界面组件
date: 2026-02-17 03:20:32
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷一
---
## 11.1 布局管理概述

- Swing 组件放置在容器中，由**布局管理器**（Layout Manager）决定组件的位置和大小
- 每个容器都有一个默认的布局管理器，可以通过 `setLayout()` 方法更改
- 不建议使用绝对定位（`setLayout(null)`），因为无法适应窗口大小变化

```java
panel.setLayout(new BorderLayout());
```

### FlowLayout（流布局）

- **默认**用于 `JPanel`
- 组件按添加顺序从左到右排列，一行放不下自动换行
- 可设置对齐方式：`LEFT`、`CENTER`（默认）、`RIGHT`

```java
panel.setLayout(new FlowLayout(FlowLayout.LEFT, 20, 10));
// 参数：对齐方式、水平间距、垂直间距
```

### BorderLayout（边框布局）

- **默认**用于 `JFrame` 的内容窗格
- 将容器分为 5 个区域：`NORTH`、`SOUTH`、`EAST`、`WEST`、`CENTER`
- `CENTER` 区域会扩展填满剩余空间

```java
frame.add(button, BorderLayout.SOUTH);
frame.add(panel, BorderLayout.CENTER);
```

### GridLayout（网格布局）

- 将容器分成等大小的网格，组件按行优先顺序放置
- 所有组件大小相同

```java
panel.setLayout(new GridLayout(4, 3));
// 4 行 3 列
```

---

## 11.2 GridBagLayout（网格包布局）

- 最灵活也最复杂的布局管理器，组件可以**跨行跨列**
- 通过 `GridBagConstraints` 对象指定每个组件的约束

### GridBagConstraints 关键属性

| 属性 | 说明 |
| --- | --- |
| `gridx`, `gridy` | 组件所在的列和行（从 0 开始） |
| `gridwidth`, `gridheight` | 组件跨越的列数和行数 |
| `weightx`, `weighty` | 额外空间分配权重（0.0 ~ 1.0） |
| `fill` | 填充方式：`NONE`、`HORIZONTAL`、`VERTICAL`、`BOTH` |
| `anchor` | 对齐方式：`CENTER`、`NORTH`、`SOUTHEAST` 等 |
| `insets` | 组件外部的填充（上、左、下、右） |

```java
var layout = new GridBagLayout();
panel.setLayout(layout);

var constraints = new GridBagConstraints();
constraints.gridx = 0;
constraints.gridy = 0;
constraints.gridwidth = 2;  // 跨 2 列
constraints.fill = GridBagConstraints.HORIZONTAL;
constraints.weightx = 1.0;
panel.add(textField, constraints);
```

> 💡 实际开发中，通常会编写一个辅助方法来简化 `GridBagConstraints` 的设置
> 

### GroupLayout

- 由 GUI 构建工具（如 NetBeans）自动生成的布局
- 分别定义**水平**和**垂直**方向的组件组
- 手写代码时较少使用，了解即可

---

## 11.3 文本输入

### JTextField（文本域）

- 单行文本输入组件
- 构造时指定**列数**（不是像素宽度），列数是根据字符平均宽度计算

```java
var textField = new JTextField("默认文本", 20); // 20 列宽
String text = textField.getText().trim(); // 获取输入内容
```

### JTextArea（文本区）

- 多行文本输入组件
- 可设置行数和列数、是否自动换行

```java
var textArea = new JTextArea(8, 40); // 8 行 40 列
textArea.setLineWrap(true);          // 开启自动换行
textArea.setWrapStyleWord(true);     // 按单词换行

// 通常放在滚动窗格中
var scrollPane = new JScrollPane(textArea);
```

### JPasswordField（密码域）

- 输入内容显示为回显字符（默认 `•`）
- 使用 `getPassword()` 获取密码（返回 `char[]`，而非 `String`，更安全）

```java
var passwordField = new JPasswordField(20);
char[] password = passwordField.getPassword();
```

### JFormattedTextField（格式化文本域）

- 支持限制输入格式，如数字、日期等
- 常用格式器：`NumberFormatter`、`DateFormatter`、`MaskFormatter`

```java
var intField = new JFormattedTextField(NumberFormat.getIntegerInstance());
intField.setValue(100);
int value = ((Number) intField.getValue()).intValue();
```

### JSpinner（微调器）

- 通过上下箭头按钮调整值
- 支持数字、日期、列表等模型

```java
var spinner = new JSpinner(new SpinnerNumberModel(5, 0, 100, 1));
// 参数：初始值、最小值、最大值、步长
```

---

## 11.4 选择组件

### JCheckBox（复选框）

- 用于开/关选择，可以独立勾选多个

```java
var bold = new JCheckBox("Bold");
bold.setSelected(true); // 设置为选中
bold.addActionListener(event -> {
    boolean selected = bold.isSelected();
});
```

### JRadioButton（单选按钮）

- 一组中只能选一个，需要用 `ButtonGroup` 分组

```java
var small = new JRadioButton("Small");
var medium = new JRadioButton("Medium", true); // 默认选中
var large = new JRadioButton("Large");

var group = new ButtonGroup();
group.add(small);
group.add(medium);
group.add(large);
```

### JComboBox（组合框）

- 下拉选择列表，可设置为**可编辑**或**不可编辑**

```java
var combo = new JComboBox<String>();
combo.addItem("选项1");
combo.addItem("选项2");
combo.setEditable(true); // 允许用户输入

String selected = (String) combo.getSelectedItem();
```

### JSlider（滑动条）

- 在一个范围内拖动选择数值

```java
var slider = new JSlider(0, 100, 50); // 最小值、最大值、初始值
slider.setMajorTickSpacing(20);       // 大刻度间距
slider.setMinorTickSpacing(5);        // 小刻度间距
slider.setPaintTicks(true);           // 显示刻度
slider.setPaintLabels(true);          // 显示标签
slider.setSnapToTicks(true);          // 对齐到刻度

slider.addChangeListener(event -> {
    int value = slider.getValue();
});
```

---

## 11.5 菜单

### 菜单栏结构

- `JMenuBar` → `JMenu` → `JMenuItem`
- 菜单项可以是普通项、复选框项、单选按钮项、分隔符或子菜单

```java
var menuBar = new JMenuBar();
frame.setJMenuBar(menuBar);

var fileMenu = new JMenu("File");
menuBar.add(fileMenu);

var openItem = new JMenuItem("Open");
fileMenu.add(openItem);
fileMenu.addSeparator();  // 添加分隔线
fileMenu.add(new JMenuItem("Exit"));
```

### 快捷键与加速器

- **助记符**（Mnemonic）：`Alt + 字母` 打开菜单
- **加速器**（Accelerator）：直接触发菜单项，无需打开菜单

```java
// 助记符
fileMenu.setMnemonic('F');      // Alt+F 打开菜单

// 加速器
openItem.setAccelerator(
    KeyStroke.getKeyStroke("ctrl O")  // Ctrl+O 直接触发
);
```

### JCheckBoxMenuItem 和 JRadioButtonMenuItem

```java
var readOnly = new JCheckBoxMenuItem("Read-only");
fileMenu.add(readOnly);

var group = new ButtonGroup();
var insertItem = new JRadioButtonMenuItem("Insert");
var overtypeItem = new JRadioButtonMenuItem("Overtype");
group.add(insertItem);
group.add(overtypeItem);
```

### 弹出菜单（JPopupMenu）

- 右键弹出的上下文菜单

```java
var popup = new JPopupMenu();
popup.add(new JMenuItem("Cut"));
popup.add(new JMenuItem("Copy"));
popup.add(new JMenuItem("Paste"));

component.setComponentPopupMenu(popup);
```

---

## 11.6 工具栏（JToolBar）

- 可包含按钮和其他组件的可拖拽栏
- 默认可以拖拽到窗口的各个边缘

```java
var toolBar = new JToolBar("Tools");
toolBar.add(openButton);
toolBar.addSeparator();
toolBar.add(saveButton);

frame.add(toolBar, BorderLayout.NORTH);
```

- 使用 `Action` 对象可以让菜单项和工具栏按钮共享行为

```java
var openAction = new AbstractAction("Open", openIcon) {
    public void actionPerformed(ActionEvent e) {
        // 打开文件
    }
};
menuItem = new JMenuItem(openAction);
toolBar.add(openAction);
```

---

## 11.7 对话框

### JOptionPane（选项面板）

- 提供快速创建常见对话框的静态方法

| 方法 | 用途 |
| --- | --- |
| `showMessageDialog` | 显示消息 |
| `showConfirmDialog` | 确认/取消选择 |
| `showInputDialog` | 获取用户输入 |
| `showOptionDialog` | 自定义按钮选项 |

```java
// 消息对话框
JOptionPane.showMessageDialog(parent, "操作完成！", "提示",
    JOptionPane.INFORMATION_MESSAGE);

// 确认对话框
int result = JOptionPane.showConfirmDialog(parent, "确定删除？",
    "确认", JOptionPane.YES_NO_OPTION);
if (result == JOptionPane.YES_OPTION) { /* 删除 */ }

// 输入对话框
String name = JOptionPane.showInputDialog("请输入姓名：");
```

### 自定义对话框（JDialog）

- 继承 `JDialog` 创建自定义对话框
- 可设置**模态**（modal）：阻塞其他窗口输入

```java
var dialog = new JDialog(owner, "设置", true); // true = 模态
dialog.setSize(300, 200);
dialog.setVisible(true);
```

### JFileChooser（文件选择器）

```java
var chooser = new JFileChooser();
chooser.setCurrentDirectory(new File("."));

// 添加文件过滤器
chooser.addChoosableFileFilter(
    new FileNameExtensionFilter("图片文件", "jpg", "png", "gif")
);

int result = chooser.showOpenDialog(parent);
if (result == JFileChooser.APPROVE_OPTION) {
    File selectedFile = chooser.getSelectedFile();
}
```

### JColorChooser（颜色选择器）

```java
Color selectedColor = JColorChooser.showDialog(
    parent, "选择颜色", Color.RED  // 初始颜色
);
```

---

## 11.8 本章要点总结

<aside>
📌

- **布局管理器**控制组件排列：`FlowLayout`、`BorderLayout`、`GridLayout` 最常用，`GridBagLayout` 最灵活
- **文本组件**：`JTextField` 单行、`JTextArea` 多行、`JPasswordField` 密码输入
- **选择组件**：`JCheckBox` 多选、`JRadioButton`+`ButtonGroup` 单选、`JComboBox` 下拉、`JSlider` 滑动
- **菜单系统**：`JMenuBar` → `JMenu` → `JMenuItem`，支持快捷键、弹出菜单
- **对话框**：`JOptionPane` 快速对话框、`JDialog` 自定义、`JFileChooser` 文件选择、`JColorChooser` 颜色选择
- 使用 **Action** 对象可以统一管理菜单和工具栏的行为
</aside>