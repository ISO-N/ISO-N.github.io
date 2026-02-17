---
title: 第7章 用户输入和while循环
date: 2026-02-17 15:04:51
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 7.1 函数 input() 的工作原理

函数 `input()` 让程序暂停运行，等待用户输入一些文本。获取用户输入后，Python 将其赋给一个变量：

```python
message = input("Tell me something, and I will repeat it back to you: ")
print(message)
```

### 7.1.1 编写清晰的提示

通过在提示末尾添加一个空格，可以将提示与用户输入分开，让用户清楚地知道其输入始于何处：

```python
name = input("Please enter your name: ")
print(f"\nHello, {name}!")
```

对于较长的提示，可以先将提示赋给一个变量，再将该变量传递给 `input()` 函数：

```python
prompt = "If you share your name, we can personalize the messages you see."
prompt += "\nWhat is your first name? "
name = input(prompt)
print(f"\nHello, {name}!")
```

### 7.1.2 使用 int() 来获取数值输入

`input()` 函数将用户输入解读为**字符串**。使用 `int()` 函数可将字符串转换为数值：

```python
age = input("How old are you? ")
age = int(age)
if age >= 18:
    print("\nYou are old enough to ride!")
```

> 💡 将数值输入用于计算和比较前，务必先用 `int()` 转换。
> 

### 7.1.3 求模运算符

**求模运算符**（`%`）将两个数相除并返回**余数**：

```python
4 % 3   # 1
5 % 3   # 2
6 % 3   # 0
```

可以利用求模运算符判断一个数是**奇数还是偶数**：

```python
number = input("Enter a number, and I'll tell you if it's even or odd: ")
number = int(number)
if number % 2 == 0:
    print(f"\nThe number {number} is even.")
else:
    print(f"\nThe number {number} is odd.")
```

## 7.2 while 循环简介

`for` 循环用于针对集合中的每个元素执行一个代码块，而 `while` 循环则**不断运行，直到指定的条件不满足为止**。

### 7.2.1 使用 while 循环

```python
current_number = 1
while current_number <= 5:
    print(current_number)
    current_number += 1
```

### 7.2.2 让用户选择何时退出

可以使用 `while` 循环让程序在用户愿意时不断运行：

```python
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program. "

message = ""
while message != 'quit':
    message = input(prompt)
    if message != 'quit':
        print(message)
```

### 7.2.3 使用标志

在要求很多条件都满足才继续运行的程序中，可定义一个变量，用于判断整个程序是否处于活动状态。这个变量称为**标志（flag）**：

```python
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program. "

active = True
while active:
    message = input(prompt)
    if message == 'quit':
        active = False
    else:
        print(message)
```

> 💡 标志非常有用：在任意一个事件导致活动标志变为 `False` 时，循环都将停止，让代码更清晰。
> 

### 7.2.4 使用 break 退出循环

要立即退出 `while` 循环，不再运行循环中余下的代码，也不管条件测试的结果如何，可使用 `break` 语句：

```python
prompt = "\nPlease enter the name of a city you have visited:"
prompt += "\n(Enter 'quit' when you are finished.) "

while True:
    city = input(prompt)
    if city == 'quit':
        break
    else:
        print(f"I'd love to go to {city.title()}!")
```

<aside>
⚠️

`break` 语句可用于任何 Python 循环中，包括 `for` 循环。

</aside>

### 7.2.5 在循环中使用 continue

要返回循环开头，并根据条件测试结果决定是否继续执行循环，可使用 `continue` 语句：

```python
current_number = 0
while current_number < 10:
    current_number += 1
    if current_number % 2 == 0:
        continue
    print(current_number)
```

> 💡 `continue` 跳过当前迭代的剩余代码，返回循环开头重新判断条件。
> 

### 7.2.6 避免无限循环

每个 `while` 循环都必须有**停止运行的途径**，确保循环条件最终会变为 `False`，或者有 `break` 语句可以退出：

```python
# ⚠️ 这是一个无限循环！
x = 1
while x <= 5:
    print(x)
    # 忘记了 x += 1
```

<aside>
💡

如果程序陷入无限循环，可按 **Ctrl+C** 或关闭终端窗口来停止。编写循环时务必检查是否有退出途径。

</aside>

## 7.3 使用 while 循环处理列表和字典

`for` 循环遍历列表时**不应修改列表**，否则会导致 Python 难以跟踪其中的元素。要在遍历列表的同时修改它，可使用 `while` 循环。

### 7.3.1 在列表之间移动元素

```python
# 首先，创建一个待验证的用户列表和一个已验证的用户列表
unconfirmed_users = ['alice', 'brian', 'candace']
confirmed_users = []

# 验证每个用户，直到没有未验证用户为止
while unconfirmed_users:
    current_user = unconfirmed_users.pop()
    print(f"Verifying user: {current_user.title()}")
    confirmed_users.append(current_user)

# 显示所有已验证的用户
print("\nThe following users have been confirmed:")
for confirmed_user in confirmed_users:
    print(confirmed_user.title())
```

### 7.3.2 删除为特定值的所有列表元素

使用 `while` 循环配合 `remove()` 删除列表中所有特定值：

```python
pets = ['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat']
print(pets)

while 'cat' in pets:
    pets.remove('cat')

print(pets)
```

> 💡 `remove()` 每次只删除第一个匹配的值，因此需要配合 `while` 循环来删除所有匹配项。
> 

### 7.3.3 使用用户输入来填充字典

```python
responses = {}

# 设置一个标志，指出调查是否继续
polling_active = True

while polling_active:
    # 提示输入被调查者的名字和回答
    name = input("\nWhat is your name? ")
    response = input("Which mountain would you like to climb someday? ")

    # 将回答存储在字典中
    responses[name] = response

    # 看看是否还有人要参与调查
    repeat = input("Would you like to let another person respond? (yes/no) ")
    if repeat == 'no':
        polling_active = False

# 调查结束，显示结果
print("\n--- Poll Results ---")
for name, response in responses.items():
    print(f"{name} would like to climb {response}.")
```

## 📝 本章小结

| **概念** | **说明** |
| --- | --- |
| `input()` | 获取用户输入（返回字符串） |
| `int()` | 将字符串转换为整数，用于数值计算和比较 |
| `%` 求模运算符 | 返回两数相除的余数 |
| `while` 循环 | 条件为 `True` 时不断循环 |
| 标志（flag） | 控制程序是否继续运行的布尔变量 |
| `break` | 立即退出循环 |
| `continue` | 跳过当前迭代，回到循环开头 |
| 避免无限循环 | 确保循环条件会变为 `False` 或有 `break` 退出 |
| `while`  • 列表 | 在列表之间移动元素、删除特定值 |
| `while`  • 字典 | 用用户输入填充字典 |