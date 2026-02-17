---
title: 第5章 if语句
date: 2026-02-17 15:04:49
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 5.1 一个简单示例

```python
cars = ['audi', 'bmw', 'subaru', 'toyota']

for car in cars:
    if car == 'bmw':
        print(car.upper())
    else:
        print(car.title())
```

## 5.2 条件测试

每条 `if` 语句的核心都是一个值为 `True` 或 `False` 的表达式，这个表达式称为 **条件测试**。

### 5.2.1 检查是否相等

```python
car = 'bmw'
car == 'bmw'   # True
car == 'audi'  # False
```

### 5.2.2 检查是否相等时忽略大小写

Python 中检查相等时**区分大小写**。可以使用 `.lower()` 方法进行不区分大小写的比较：

```python
car = 'Audi'
car.lower() == 'audi'  # True
```

> 💡 `.lower()` 不会修改原始变量的值，只是临时转换用于比较。
> 

### 5.2.3 检查是否不相等

使用 `!=` 运算符：

```python
requested_topping = 'mushrooms'
if requested_topping != 'anchovies':
    print("Hold the anchovies!")
```

### 5.2.4 数值比较

```python
age = 18
age == 18   # True
age != 18   # False
age < 21    # True
age <= 18   # True
age > 17    # True
age >= 18   # True
```

### 5.2.5 检查多个条件

**使用 `and` 检查多个条件（都满足）：**

```python
age_0 = 22
age_1 = 18
age_0 >= 21 and age_1 >= 21  # False
```

**使用 `or` 检查多个条件（至少一个满足）：**

```python
age_0 >= 21 or age_1 >= 21  # True
```

### 5.2.6 检查特定值是否在列表中

使用关键字 `in`：

```python
requested_toppings = ['mushrooms', 'onions', 'pineapple']
'mushrooms' in requested_toppings  # True
'pepperoni' in requested_toppings  # False
```

### 5.2.7 检查特定值是否不在列表中

使用关键字 `not in`：

```python
banned_users = ['andrew', 'carolina', 'david']
user = 'marie'
if user not in banned_users:
    print(f"{user.title()}, you can post a response if you wish.")
```

### 5.2.8 布尔表达式

布尔表达式是条件测试的别名，结果要么为 `True`，要么为 `False`。常用于记录条件状态：

```python
game_active = True
can_edit = False
```

## 5.3 if 语句

### 5.3.1 简单的 if 语句

```python
age = 19
if age >= 18:
    print("You are old enough to vote!")
```

### 5.3.2 if-else 语句

```python
age = 17
if age >= 18:
    print("You are old enough to vote!")
else:
    print("Sorry, you are too young to vote.")
```

### 5.3.3 if-elif-else 结构

用于检查**多个条件**：

```python
age = 12
if age < 4:
    price = 0
elif age < 18:
    price = 25
else:
    price = 40
print(f"Your admission cost is ${price}.")
```

### 5.3.4 使用多个 elif 代码块

```python
age = 12
if age < 4:
    price = 0
elif age < 18:
    price = 25
elif age < 65:
    price = 40
else:
    price = 20
```

### 5.3.5 省略 else 代码块

`else` 是一个包罗万象的语句，有时候用一个明确的 `elif` 来替代 `else` 更好：

```python
age = 12
if age < 4:
    price = 0
elif age < 18:
    price = 25
elif age < 65:
    price = 40
elif age >= 65:
    price = 20
```

> 💡 `else` 代码块并非必需。使用具体的 `elif` 条件可以让代码逻辑更清晰。
> 

### 5.3.6 测试多个条件

如果需要检查**多个独立条件**（而非互斥条件），应使用多个独立的 `if` 语句，而不是 `if-elif-else` 结构：

```python
requested_toppings = ['mushrooms', 'extra cheese']

if 'mushrooms' in requested_toppings:
    print("Adding mushrooms.")
if 'pepperoni' in requested_toppings:
    print("Adding pepperoni.")
if 'extra cheese' in requested_toppings:
    print("Adding extra cheese.")
```

<aside>
⚠️

**if-elif-else** 适用于只有一个条件会通过的场景；多个独立的 **if** 语句适用于可能有多个条件同时成立的场景。

</aside>

## 5.4 使用 if 语句处理列表

### 5.4.1 检查特殊元素

```python
requested_toppings = ['mushrooms', 'green peppers', 'extra cheese']

for requested_topping in requested_toppings:
    if requested_topping == 'green peppers':
        print("Sorry, we are out of green peppers right now.")
    else:
        print(f"Adding {requested_topping}.")
```

### 5.4.2 确定列表不是空的

在 `if` 语句中将列表名用作条件表达式时，**空列表返回 `False`**，**非空列表返回 `True`**：

```python
requested_toppings = []

if requested_toppings:
    for requested_topping in requested_toppings:
        print(f"Adding {requested_topping}.")
    print("\nFinished making your pizza!")
else:
    print("Are you sure you want a plain pizza?")
```

### 5.4.3 使用多个列表

```python
available_toppings = ['mushrooms', 'olives', 'green peppers',
                      'pepperoni', 'pineapple', 'extra cheese']
requested_toppings = ['mushrooms', 'french fries', 'extra cheese']

for requested_topping in requested_toppings:
    if requested_topping in available_toppings:
        print(f"Adding {requested_topping}.")
    else:
        print(f"Sorry, we don't have {requested_topping}.")
```

## 5.5 设置 if 语句的格式

<aside>
✅

PEP 8 建议：在比较运算符（如 `==`、`>=`、`<=` 等）两边各添加**一个空格**。例如：`if age < 4:` 比 `if age<4:` 可读性更好。

</aside>

## 📝 本章小结

| **概念** | **说明** |
| --- | --- |
| 条件测试 | 结果为 `True` 或 `False` 的表达式 |
| `==` / `!=` | 相等 / 不等运算符 |
| `and` / `or` | 组合多个条件 |
| `in` / `not in` | 检查值是否在列表中 |
| `if` / `if-else` / `if-elif-else` | 条件分支结构 |
| 独立 `if` vs `if-elif-else` | 独立条件用多个 `if`，互斥条件用 `if-elif-else` |
| 空列表检查 | 空列表在条件语句中为 `False` |
| PEP 8 格式 | 比较运算符两边加空格 |