---
title: 第10章 文件和异常
date: 2026-02-17 15:04:53
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 10.1 读取文件

### 读取整个文件

使用 `open()` 函数打开文件，`read()` 方法读取全部内容。推荐使用 `with` 关键字，它会在代码块执行完毕后**自动关闭文件**。

```python
from pathlib import Path

path = Path('pi_digits.txt')
contents = path.read_text()
print(contents)
```

<aside>
💡

`Path` 对象来自 `pathlib` 模块（Python 3.4+），是处理文件路径的现代方式。`read_text()` 会读取文件的全部内容并作为字符串返回。

</aside>

> **注意：** `read_text()` 在文件末尾会多返回一个空行，可以用 `rstrip()` 去除末尾空白。
> 

### 相对路径与绝对路径

- **相对路径：** 相对于当前工作目录，如 `Path('text_files/filename.txt')`
- **绝对路径：** 从系统根目录开始的完整路径，如 `Path('/home/user/data/filename.txt')`

### 访问文件中的各行

使用 `splitlines()` 方法将文件内容按行分割为列表：

```python
path = Path('pi_digits.txt')
contents = path.read_text()
lines = contents.splitlines()

for line in lines:
    print(line)
```

### 使用文件内容

读取文件后，可以对内容进行任意处理：

```python
path = Path('pi_million_digits.txt')
contents = path.read_text()
lines = contents.splitlines()

pi_string = ''
for line in lines:
    pi_string += line.lstrip()

print(f"{pi_string[:52]}...")
print(len(pi_string))
```

<aside>
⚠️

Python 读取文本文件时，所有内容都被解释为**字符串**。如果需要数值运算，需要用 `int()` 或 `float()` 转换。

</aside>

---

## 10.2 写入文件

### 写入一行或多行

使用 `write_text()` 方法将字符串写入文件。如果文件不存在会自动创建，如果已存在则**覆盖**原内容：

```python
from pathlib import Path

path = Path('programming.txt')
path.write_text("I love programming.")
```

### 写入多行

写入多行内容时，需要在字符串中包含换行符 `\n`：

```python
path = Path('programming.txt')
contents = "I love programming.\n"
contents += "I love creating new games.\n"
contents += "I also love working with data.\n"
path.write_text(contents)
```

<aside>
💡

`write_text()` 一次性写入整个字符串，因此需要先构建好完整的字符串内容再写入。

</aside>

---

## 10.3 异常

异常是 Python 用来管理程序执行期间发生的错误的特殊对象。使用 `try-except` 代码块处理异常，可以让程序在遇到错误时**优雅地继续运行**，而不是崩溃。

### 处理 ZeroDivisionError

```python
try:
    print(5/0)
except ZeroDivisionError:
    print("You can't divide by zero!")
```

### try-except-else 代码块

`else` 块中的代码**仅在 `try` 块成功执行后**才会运行：

```python
print("Give me two numbers, and I'll divide them.")
print("Enter 'q' to quit.")

while True:
    first_number = input("\nFirst number: ")
    if first_number == 'q':
        break
    second_number = input("Second number: ")
    if second_number == 'q':
        break
    try:
        answer = int(first_number) / int(second_number)
    except ZeroDivisionError:
        print("You can't divide by zero!")
    else:
        print(answer)
```

<aside>
📌

**try-except-else 结构：**

- **`try`** — 尝试执行可能出错的代码
- **`except`** — 捕获并处理特定异常
- **`else`** — 仅在 try 成功时执行
</aside>

### 处理 FileNotFoundError

```python
from pathlib import Path

path = Path('alice.txt')
try:
    contents = path.read_text(encoding='utf-8')
except FileNotFoundError:
    print(f"Sorry, the file {path} does not exist.")
```

### 分析文本

结合异常处理分析文件内容，例如统计单词数：

```python
from pathlib import Path

path = Path('alice.txt')
try:
    contents = path.read_text(encoding='utf-8')
except FileNotFoundError:
    print(f"Sorry, the file {path} does not exist.")
else:
    words = contents.split()
    num_words = len(words)
    print(f"The file {path} has about {num_words} words.")
```

### 静默失败（pass）

有时你希望程序在发生异常时**什么都不做**，使用 `pass` 语句：

```python
try:
    contents = path.read_text(encoding='utf-8')
except FileNotFoundError:
    pass
```

<aside>
💡

`pass` 语句也可作为占位符，提醒你在某个地方什么都没做，以后可能需要补充处理逻辑。

</aside>

---

## 10.4 存储数据

使用 `json` 模块可以方便地将 Python 数据结构存储为 **JSON 格式**文件，并在需要时重新加载。

### 使用 json.dumps() 和 json.loads()

- `json.dumps()` — 将 Python 对象转换为 JSON 格式的**字符串**
- `json.loads()` — 将 JSON 字符串转换回 **Python 对象**

```python
from pathlib import Path
import json

numbers = [2, 3, 5, 7, 11, 13]

path = Path('numbers.json')
contents = json.dumps(numbers)
path.write_text(contents)
```

```python
# 读取 JSON 数据
path = Path('numbers.json')
contents = path.read_text()
numbers = json.loads(contents)
print(numbers)
```

### 保存和读取用户生成的数据

一个实用示例 — 记住用户名：

```python
from pathlib import Path
import json

path = Path('username.json')

try:
    contents = path.read_text()
except FileNotFoundError:
    username = input("What is your name? ")
    contents = json.dumps(username)
    path.write_text(contents)
    print(f"We'll remember you when you come back, {username}!")
else:
    username = json.loads(contents)
    print(f"Welcome back, {username}!")
```

### 重构

将代码拆分为职责单一的函数，使逻辑更清晰、更易维护：

```python
from pathlib import Path
import json

def get_stored_username(path):
    """如果存储了用户名，就获取它。"""
    if path.exists():
        contents = path.read_text()
        username = json.loads(contents)
        return username
    return None

def get_new_username(path):
    """提示用户输入用户名。"""
    username = input("What is your name? ")
    contents = json.dumps(username)
    path.write_text(contents)
    return username

def greet_user():
    """问候用户，并指出其名字。"""
    path = Path('username.json')
    username = get_stored_username(path)
    if username:
        print(f"Welcome back, {username}!")
    else:
        username = get_new_username(path)
        print(f"We'll remember you when you come back, {username}!")

greet_user()
```

<aside>
📌

**重构原则：** 每个函数应只负责一项具体任务。函数名应具有描述性，让代码读起来像自然语言。

</aside>

---

## 本章小结

| **主题** | **关键知识点** |
| --- | --- |
| 读取文件 | `Path` 对象 + `read_text()`，`splitlines()` 按行读取 |
| 写入文件 | `write_text()` 写入内容（覆盖模式），需手动添加 `\n` 换行 |
| 异常处理 | `try-except-else` 结构，`pass` 静默失败 |
| 存储数据 | `json.dumps()` 序列化，`json.loads()` 反序列化 |
| 重构 | 将代码拆分为职责单一的函数，提高可读性和可维护性 |