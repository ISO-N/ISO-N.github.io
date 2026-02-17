---
title: 第6章 字典
date: 2026-02-17 15:04:50
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 6.1 一个简单的字典

字典是一系列 **键值对**（key-value pair）的集合，用花括号 `{}` 表示。每个键与一个值关联，可通过键来访问对应的值。

```python
alien_0 = {'color': 'green', 'points': 5}
print(alien_0['color'])   # green
print(alien_0['points'])  # 5
```

- 键和值之间用冒号 `:` 分隔，键值对之间用逗号 `,` 分隔
- 值可以是任意 Python 对象：数字、字符串、列表，甚至是另一个字典

---

## 6.2 使用字典

### 访问字典中的值

使用 `字典名[键]` 的语法获取值：

```python
alien_0 = {'color': 'green'}
print(alien_0['color'])  # green
```

### 添加键值对

字典是动态结构，可随时添加新的键值对：

```python
alien_0 = {'color': 'green', 'points': 5}
alien_0['x_position'] = 0
alien_0['y_position'] = 25
print(alien_0)
# {'color': 'green', 'points': 5, 'x_position': 0, 'y_position': 25}
```

### 先创建一个空字典

```python
alien_0 = {}
alien_0['color'] = 'green'
alien_0['points'] = 5
```

### 修改字典中的值

```python
alien_0 = {'color': 'green'}
alien_0['color'] = 'yellow'
```

### 删除键值对

使用 `del` 语句永久删除键值对：

```python
alien_0 = {'color': 'green', 'points': 5}
del alien_0['points']
print(alien_0)  # {'color': 'green'}
```

<aside>
⚠️

被删除的键值对会**永久消失**。

</aside>

### 使用 get() 来访问值

使用 `get()` 方法可以在键不存在时返回默认值，避免 `KeyError`：

```python
alien_0 = {'color': 'green', 'points': 5}
speed = alien_0.get('speed', 'No speed value assigned.')
print(speed)  # No speed value assigned.
```

- 第一个参数：要查找的键
- 第二个参数（可选）：键不存在时返回的默认值；如果省略，默认返回 `None`

---

## 6.3 遍历字典

### 遍历所有键值对：items()

```python
user_0 = {
    'username': 'efermi',
    'first': 'enrico',
    'last': 'fermi',
}

for key, value in user_0.items():
    print(f"\nKey: {key}")
    print(f"Value: {value}")
```

### 遍历所有键：keys()

```python
for name in user_0.keys():
    print(name)
```

<aside>
💡

遍历字典时，默认遍历的就是所有的键，因此 `for name in user_0:` 与 `for name in user_0.keys():` 效果相同。但显式使用 `keys()` 可读性更好。

</aside>

### 按特定顺序遍历键：sorted()

```python
for name in sorted(user_0.keys()):
    print(name)
```

### 遍历所有值：values()

```python
for value in user_0.values():
    print(value)
```

### 使用 set() 去重

如果值中有重复项，可使用 `set()` 剔除：

```python
favorite_languages = {
    'jen': 'python',
    'sarah': 'c',
    'edward': 'python',
    'phil': 'python',
}

for language in set(favorite_languages.values()):
    print(language)
```

---

## 6.4 嵌套

### 字典列表（列表中嵌套字典）

将多个字典存储在列表中：

```python
alien_0 = {'color': 'green', 'points': 5}
alien_1 = {'color': 'yellow', 'points': 10}
alien_2 = {'color': 'red', 'points': 15}

aliens = [alien_0, alien_1, alien_2]

for alien in aliens:
    print(alien)
```

使用代码批量生成：

```python
aliens = []
for alien_number in range(30):
    new_alien = {'color': 'green', 'points': 5, 'speed': 'slow'}
    aliens.append(new_alien)
```

### 在字典中存储列表

```python
pizza = {
    'crust': 'thick',
    'toppings': ['mushrooms', 'extra cheese'],
}

for topping in pizza['toppings']:
    print(f"\t{topping}")
```

```python
favorite_languages = {
    'jen': ['python', 'rust'],
    'sarah': ['c'],
    'edward': ['rust', 'go'],
    'phil': ['python', 'haskell'],
}

for name, languages in favorite_languages.items():
    print(f"\n{name.title()}'s favorite languages are:")
    for language in languages:
        print(f"\t{language.title()}")
```

### 在字典中存储字典

```python
users = {
    'aeinstein': {
        'first': 'albert',
        'last': 'einstein',
        'location': 'princeton',
    },
    'mcurie': {
        'first': 'marie',
        'last': 'curie',
        'location': 'paris',
    },
}

for username, user_info in users.items():
    full_name = f"{user_info['first']} {user_info['last']}"
    location = user_info['location']
    print(f"\nUsername: {username}")
    print(f"\tFull name: {full_name.title()}")
    print(f"\tLocation: {location.title()}")
```

<aside>
📌

当字典中嵌套字典时，代码会快速变复杂。如果结构变得太深，可以考虑用更简洁的方式组织数据。

</aside>

---

## 本章小结

| 概念 | 说明 |
| --- | --- |
| 字典基础 | 用 `{}` 创建，存储键值对，通过键访问值 |
| 增删改查 | `dict[key] = value` 添加/修改；`del dict[key]` 删除；`get()` 安全访问 |
| 遍历 | `items()` 遍历键值对；`keys()` 遍历键；`values()` 遍历值 |
| 排序与去重 | `sorted()` 按顺序遍历；`set()` 去除重复值 |
| 嵌套 | 字典列表、字典中存列表、字典中存字典 |