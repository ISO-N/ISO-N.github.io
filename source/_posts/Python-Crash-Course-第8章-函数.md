---
title: 第8章 函数
date: 2026-02-17 15:04:51
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 8.1 定义函数

使用关键字 `def` 定义函数，后跟函数名和圆括号。

```python
def greet_user():
    """显示简单的问候语"""
    print("Hello!")

greet_user()
```

- **文档字符串（docstring）**：用三引号 `"""..."""` 括起的注释，描述函数的功能，Python 用它来生成文档。

### 向函数传递信息

```python
def greet_user(username):
    """显示简单的问候语"""
    print(f"Hello, {username.title()}!")

greet_user('jesse')
```

### 实参和形参

- **形参（parameter）**：函数定义中的变量，如 `username`
- **实参（argument）**：调用函数时传入的值，如 `'jesse'`

---

## 8.2 传递实参

### 位置实参

调用时按 **顺序** 将实参关联到形参：

```python
def describe_pet(animal_type, pet_name):
    """显示宠物的信息"""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet('hamster', 'harry')
describe_pet('dog', 'willie')  # 可多次调用
```

> ⚠️ 位置实参的 **顺序很重要**，务必确保实参与形参的顺序一致。
> 

### 关键字实参

传递时直接将 **名称—值** 对关联，无需考虑顺序：

```python
describe_pet(animal_type='hamster', pet_name='harry')
describe_pet(pet_name='harry', animal_type='hamster')  # 顺序无关
```

### 默认值

为形参指定默认值后，调用时可省略对应的实参：

```python
def describe_pet(pet_name, animal_type='dog'):
    """显示宠物的信息"""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet(pet_name='willie')        # 使用默认值
describe_pet('willie')                 # 更简洁的写法
describe_pet(pet_name='harry', animal_type='hamster')  # 覆盖默认值
```

> 💡 使用默认值时，应将 **没有默认值的形参放前面**，有默认值的放后面。
> 

---

## 8.3 返回值

使用 `return` 语句将值返回到调用函数的代码行。

### 返回简单值

```python
def get_formatted_name(first_name, last_name):
    """返回整洁的姓名"""
    full_name = f"{first_name} {last_name}"
    return full_name.title()

musician = get_formatted_name('jimi', 'hendrix')
print(musician)  # Jimi Hendrix
```

### 让实参变成可选的

```python
def get_formatted_name(first_name, last_name, middle_name=''):
    """返回整洁的姓名"""
    if middle_name:
        full_name = f"{first_name} {middle_name} {last_name}"
    else:
        full_name = f"{first_name} {last_name}"
    return full_name.title()

musician = get_formatted_name('jimi', 'hendrix')
musician = get_formatted_name('john', 'hooker', 'lee')
```

### 返回字典

```python
def build_person(first_name, last_name, age=None):
    """返回一个表示人的字典"""
    person = {'first': first_name, 'last': last_name}
    if age:
        person['age'] = age
    return person

musician = build_person('jimi', 'hendrix', age=27)
```

### 结合使用函数和 while 循环

```python
def get_formatted_name(first_name, last_name):
    full_name = f"{first_name} {last_name}"
    return full_name.title()

while True:
    print("\nPlease tell me your name:")
    print("(enter 'q' at any time to quit)")
    f_name = input("First name: ")
    if f_name == 'q':
        break
    l_name = input("Last name: ")
    if l_name == 'q':
        break
    formatted_name = get_formatted_name(f_name, l_name)
    print(f"\nHello, {formatted_name}!")
```

---

## 8.4 传递列表

将列表传递给函数后，函数可以直接访问和修改列表内容。

```python
def greet_users(names):
    """向列表中的每位用户发出问候"""
    for name in names:
        msg = f"Hello, {name.title()}!"
        print(msg)

usernames = ['hannah', 'ty', 'margot']
greet_users(usernames)
```

### 在函数中修改列表

函数对列表所做的修改是 **永久性** 的：

```python
def print_models(unprinted_designs, completed_models):
    """模拟打印每个设计，直到没有未打印的设计为止"""
    while unprinted_designs:
        current_design = unprinted_designs.pop()
        print(f"Printing model: {current_design}")
        completed_models.append(current_design)

def show_completed_models(completed_models):
    """显示打印好的所有模型"""
    print("\nThe following models have been printed:")
    for completed_model in completed_models:
        print(completed_model)

unprinted_designs = ['phone case', 'robot pendant', 'dodecahedron']
completed_models = []
print_models(unprinted_designs, completed_models)
show_completed_models(completed_models)
```

### 禁止函数修改列表

使用 **切片** `[:]` 传递列表副本：

```python
print_models(unprinted_designs[:], completed_models)
```

> 💡 除非有充分理由，否则应将原始列表传给函数。避免创建副本可以节省时间和内存。
> 

---

## 8.5 传递任意数量的实参

### *args — 任意数量的位置实参

使用 `*` 让 Python 创建一个 **元组**：

```python
def make_pizza(*toppings):
    """打印顾客点的所有配料"""
    print("\nMaking a pizza with the following toppings:")
    for topping in toppings:
        print(f"- {topping}")

make_pizza('pepperoni')
make_pizza('mushrooms', 'green peppers', 'extra cheese')
```

### 结合位置实参和任意数量实参

`*args` 必须放在 **最后**：

```python
def make_pizza(size, *toppings):
    """概述要制作的比萨"""
    print(f"\nMaking a {size}-inch pizza with the following toppings:")
    for topping in toppings:
        print(f"- {topping}")

make_pizza(16, 'pepperoni')
make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```

### **kwargs — 任意数量的关键字实参

使用 `**` 让 Python 创建一个 **字典**：

```python
def build_profile(first, last, **user_info):
    """创建一个字典，其中包含我们知道的有关用户的一切"""
    user_info['first_name'] = first
    user_info['last_name'] = last
    return user_info

user_profile = build_profile('albert', 'einstein',
                             location='princeton',
                             field='physics')
print(user_profile)
```

---

## 8.6 将函数存储在模块中

### 导入整个模块

将函数保存在独立的 `.py` 文件（**模块**）中，然后用 `import` 导入：

```python
# pizza.py
def make_pizza(size, *toppings):
    """概述要制作的比萨"""
    print(f"\nMaking a {size}-inch pizza with the following toppings:")
    for topping in toppings:
        print(f"- {topping}")
```

```python
# making_pizzas.py
import pizza

pizza.make_pizza(16, 'pepperoni')
pizza.make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese')
```

### 导入特定函数

```python
from pizza import make_pizza

make_pizza(16, 'pepperoni')  # 无需使用模块名前缀
```

### 使用 as 给函数指定别名

```python
from pizza import make_pizza as mp

mp(16, 'pepperoni')
```

### 使用 as 给模块指定别名

```python
import pizza as p

p.make_pizza(16, 'pepperoni')
```

### 导入模块中的所有函数

```python
from pizza import *

make_pizza(16, 'pepperoni')
```

> ⚠️ 不推荐使用 `import *`，可能会导致命名冲突。最佳做法是 **导入整个模块** 或 **导入特定函数**。
> 

---

## 8.7 函数编写指南

<aside>
📝

- 函数名应使用 **小写字母和下划线**，如 `greet_user()`
- 每个函数都应包含简要的 **文档字符串（docstring）**
- 形参指定默认值、函数调用中的关键字实参，等号两边 **不要有空格**
- 如果形参很多，可在函数定义中换行，并缩进两次（8 个空格）
- 如果程序或模块包含多个函数，可用 **两个空行** 分隔
- 所有 `import` 语句应放在 **文件开头**（除非文件开头有注释）
</aside>

---

## 本章小结

| **概念** | **说明** |
| --- | --- |
| 定义函数 | `def` 关键字 + 函数名 + 括号 + 冒号 |
| 位置实参 | 按顺序传递，顺序必须正确 |
| 关键字实参 | 名称—值对传递，顺序无关 |
| 默认值 | 为形参指定默认值，调用时可省略 |
| 返回值 | `return` 语句返回值 |
| 传递列表 | 函数可直接修改原列表；用 `[:]` 传副本 |
| `*args` | 接收任意数量的位置实参（元组） |
| `**kwargs` | 接收任意数量的关键字实参（字典） |
| 模块 | `import` / `from...import` / `as` 别名 |