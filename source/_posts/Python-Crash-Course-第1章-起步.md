---
title: 第1章 起步
date: 2026-02-17 15:04:46
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 1.1 搭建编程环境

### Python 版本选择

- 本书基于 **Python 3**，推荐使用最新的 Python 3 版本
- Python 2 和 Python 3 之间存在语法差异，Python 2 已于 2020 年停止维护

### 安装 Python

### Windows

1. 前往 [python.org](http://python.org) 下载 Python 安装程序
2. 运行安装程序，**务必勾选** `Add Python to PATH`
3. 安装完成后，在终端输入 `python --version` 验证安装

### macOS

1. macOS 可能自带 Python 2，需要额外安装 Python 3
2. 可以通过 [python.org](http://python.org) 下载安装，或使用 Homebrew：

```bash
brew install python
```

1. 验证：`python3 --version`

### Linux

1. 大多数 Linux 发行版预装了 Python
2. 验证：`python3 --version`
3. 如未安装，可通过包管理器安装：

```bash
sudo apt install python3
```

---

## 1.2 运行 Hello World 程序

### 在终端中运行

在终端中输入 `python`（或 `python3`）进入 Python 交互式解释器：

```python
>>> print("Hello World!")
Hello World!
```

<aside>
💡

`print()` 是 Python 的内置函数，用于将内容输出到屏幕。

</aside>

---

## 1.3 安装文本编辑器

### 推荐编辑器

- **VS Code**（Visual Studio Code）：免费、轻量、功能强大，支持 Python 扩展
- **Sublime Text**：轻量、快速，支持语法高亮

### VS Code 配置建议

1. 安装 **Python 扩展**（由 Microsoft 提供）
2. 配置自动保存、缩进为 4 个空格
3. 配置 Python 解释器路径

---

## 1.4 从文件运行 Python 程序

### 创建并运行 `.py` 文件

1. 创建文件 `hello_world.py`，写入：

```python
print("Hello Python world!")
```

1. 在终端中执行：

```bash
python hello_world.py
```

输出：

```
Hello Python world!
```

<aside>
📌

**Python 文件命名规范：** 使用小写字母和下划线（如 `hello_world.py`），避免使用空格和中文。

</aside>

---

## 1.5 疑难解答

常见问题及解决方法：

| **问题** | **可能原因** | **解决方法** |
| --- | --- | --- |
| `python` 不是内部命令 | Python 未加入系统 PATH | 重新安装并勾选 `Add to PATH`，或手动配置环境变量 |
| 语法错误 `SyntaxError` | 拼写错误、缺少引号或括号 | 仔细检查代码，注意英文标点符号 |
| 缩进错误 `IndentationError` | 缩进不一致（混用 Tab 和空格） | 统一使用 4 个空格进行缩进 |
| 文件找不到 | 终端工作目录不正确 | 使用 `cd` 切换到文件所在目录 |

---

## 本章小结

- [x]  Python 是一门**易于学习**且**功能强大**的编程语言
- [x]  搭建开发环境：安装 Python + 文本编辑器
- [x]  运行第一个程序：`print("Hello World!")`
- [x]  了解如何在**终端**和**文件**两种方式下运行 Python 代码
- [x]  掌握常见问题的排查方法