---
title: 第2章 变量和简单数据类型
date: 2026-02-17 15:04:47
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 2.1 运行 hello_[world.py](http://world.py)

```python
print("Hello Python world!")
```

- `print()` 是 Python 的内置函数，用于将内容输出到屏幕
- 运行 `.py` 文件时，Python 解释器会逐行读取并执行代码

---

## 2.2 变量

```python
message = "Hello Python world!"
print(message)
```

- **变量**是可以赋值的标签（label），指向特定的值
- 每个变量都指向一个值——与该变量关联的信息

### 变量的命名规则

- 只能包含 **字母、数字和下划线**，且不能以数字开头
- 不能包含空格，可使用下划线分隔单词，如 `greeting_message`
- 避免使用 Python **关键字和内置函数名** 作为变量名（如 `print`、`input`）
- 变量名应简短且具有描述性，如 `name` 优于 `n`，`student_name` 优于 `s_n`
- 慎用小写字母 `l` 和大写字母 `O`，容易与数字 `1` 和 `0` 混淆

### 避免命名错误

- 变量名拼写错误会导致 `NameError`
- Python 的 traceback 信息会帮助定位错误

---

## 2.3 字符串

> 字符串是一系列字符，用引号括起来。可以使用单引号或双引号。
> 

```python
"This is a string."
'This is also a string.'
```

### 使用方法修改字符串大小写

```python
name = "ada lovelace"
print(name.title())   # Ada Lovelace
print(name.upper())   # ADA LOVELACE
print(name.lower())   # ada lovelace
```

| 方法 | 作用 |
| --- | --- |
| `.title()` | 每个单词首字母大写 |
| `.upper()` | 全部大写 |
| `.lower()` | 全部小写 |

### 在字符串中使用变量（f-string）

```python
first_name = "ada"
last_name = "lovelace"
full_name = f"{first_name} {last_name}"
print(full_name)       # ada lovelace
print(f"Hello, {full_name.title()}!")  # Hello, Ada Lovelace!
```

- **f-string**（格式化字符串字面量）：在引号前加 `f`，用 `{}` 插入变量
- Python 3.6+ 引入，之前的版本使用 `format()` 方法

### 使用制表符和换行符添加空白

```python
print("Python")
print("\tPython")       # 制表符（缩进）
print("Languages:\nPython\nC\nJavaScript")  # 换行符
print("Languages:\n\tPython\n\tC\n\tJavaScript")
```

| 转义字符 | 作用 |
| --- | --- |
| `\t` | 制表符（Tab） |
| `\n` | 换行 |

### 删除空白

```python
favorite_language = '  python  '
favorite_language.rstrip()   # '  python'  去右侧空白
favorite_language.lstrip()   # 'python  '  去左侧空白
favorite_language.strip()    # 'python'    去两侧空白
```

<aside>
💡

`strip()` 方法不会修改原变量的值，只是返回一个新字符串。要永久保存，需重新赋值：`favorite_language = favorite_language.strip()`

</aside>

### 删除前缀

```python
url = 'https://nostarch.com'
simple_url = url.removeprefix('https://')
print(simple_url)   # nostarch.com
```

- `.removeprefix()` 是 Python 3.9+ 新增的方法

---

## 2.4 数

### 整数（int）

```python
2 + 3     # 5
3 - 2     # 1
2 * 3     # 6
3 ** 2    # 9（乘方）
(2 + 3) * 4  # 20
```

| 运算符 | 含义 |
| --- | --- |
| `+` | 加 |
| `-` | 减 |
| `*` | 乘 |
| `/` | 除（返回浮点数） |
| `**` | 乘方 |

### 浮点数（float）

```python
0.1 + 0.1    # 0.2
0.2 + 0.1    # 0.30000000000000004（精度问题）
2 * 0.1      # 0.2
```

<aside>
⚠️

所有编程语言都存在浮点数精度问题，大多数情况下可以忽略。

</aside>

### 整数和浮点数混合运算

- 任何运算中，只要有一个操作数是浮点数，结果就是浮点数
- 除法 `/` 的结果**始终是浮点数**，即使两个整数相除：`4 / 2` → `2.0`

### 数中的下划线

```python
universe_age = 14_000_000_000
print(universe_age)   # 14000000000
```

- Python 3.6+ 支持用下划线分组，使大数更易读，存储时会忽略下划线

### 同时给多个变量赋值

```python
x, y, z = 0, 0, 0
```

### 常量

- Python 没有内置的常量类型
- 约定：**用全大写字母表示常量**

```python
MAX_CONNECTIONS = 5000
```

---

## 2.5 注释

```python
# 向大家问好
print("Hello Python people!")
```

- 注释以 `#` 开头，Python 会忽略 `#` 之后的内容
- 注释的主要目的是**阐述代码的意图**，而非描述代码做了什么

<aside>
✍️

**编写注释的建议：**

- 尽量编写有意义的变量名和清晰的代码结构，减少注释需求
- 当代码太复杂或含义不明显时，添加注释解释*为什么*这样写
- 写注释比不写好，但过时的注释比没有注释更糟
</aside>

---

## 2.6 Python 之禅

在解释器中运行 `import this` 可查看 **The Zen of Python**，以下是几条核心原则：

> Beautiful is better than ugly.
Simple is better than complex.
Complex is better than complicated.
Readability counts.
There should be one—and preferably only one—obvious way to do it.
Now is better than never.
> 

<aside>
🐍

Python 社区推崇**简洁、可读、优雅**的代码风格。当面对多种实现方式时，优先选择最简单、最易读的方案。

</aside>