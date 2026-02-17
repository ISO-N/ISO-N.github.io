---
title: 第3章 列表简介
date: 2026-02-17 15:04:48
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 3.1 列表是什么

列表（list）是由一系列按 **特定顺序** 排列的元素组成的集合。用方括号 `[]` 表示，元素之间用 **逗号** 分隔。

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles)
# ['trek', 'cannondale', 'redline', 'specialized']
```

### 访问列表元素

通过 **索引**（从 `0` 开始）访问元素：

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[0])   # trek
print(bicycles[1])   # cannondale
print(bicycles[-1])  # specialized（最后一个元素）
```

<aside>
💡

Python 中索引 `-1` 始终返回列表的 **最后一个元素**，`-2` 返回倒数第二个，以此类推。这在不知道列表长度时非常有用。

</aside>

### 使用列表中的各个值

可以像使用普通变量一样使用列表中的值：

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
message = f"My first bicycle was a {bicycles[0].title()}."
print(message)
# My first bicycle was a Trek.
```

---

## 3.2 修改、添加和删除元素

### 修改列表元素

通过索引直接赋值：

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
motorcycles[0] = 'ducati'
print(motorcycles)  # ['ducati', 'yamaha', 'suzuki']
```

### 添加元素

| 方法 | 说明 | 示例 |
| --- | --- | --- |
| `append()` | 在列表 **末尾** 添加元素 | `motorcycles.append('ducati')` |
| `insert()` | 在列表的 **任意位置** 插入元素 | `motorcycles.insert(0, 'ducati')` |

```python
# append：末尾添加
motorcycles = ['honda', 'yamaha', 'suzuki']
motorcycles.append('ducati')
print(motorcycles)  # ['honda', 'yamaha', 'suzuki', 'ducati']

# insert：指定位置插入
motorcycles.insert(0, 'ducati')
print(motorcycles)  # ['ducati', 'honda', 'yamaha', 'suzuki', 'ducati']
```

<aside>
✅

`append()` 常用于 **动态创建列表**：先创建空列表，再逐个添加元素。

</aside>

### 删除元素

| 方式 | 说明 | 是否可继续使用被删除的值 |
| --- | --- | --- |
| `del` 语句 | 根据 **索引** 删除元素 | ❌ 不可以 |
| `pop()` | 弹出 **末尾**（或指定位置）的元素 | ✅ 可以 |
| `remove()` | 根据 **值** 删除元素 | ✅ 可以（需提前存储） |

```python
motorcycles = ['honda', 'yamaha', 'suzuki']

# del：按索引删除
del motorcycles[0]
print(motorcycles)  # ['yamaha', 'suzuki']

# pop()：弹出末尾元素
popped = motorcycles.pop()
print(popped)        # suzuki

# pop(index)：弹出指定位置元素
first = motorcycles.pop(0)
print(first)         # yamaha

# remove()：按值删除（只删除第一个匹配项）
motorcycles = ['honda', 'yamaha', 'suzuki', 'honda']
motorcycles.remove('honda')
print(motorcycles)   # ['yamaha', 'suzuki', 'honda']
```

<aside>
⚠️

`remove()` **只删除第一个** 匹配的值。如果要删除所有同名元素，需要使用循环。

</aside>

---

## 3.3 组织列表

### 永久排序 — `sort()`

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']

# 按字母顺序排序（永久修改）
cars.sort()
print(cars)  # ['audi', 'bmw', 'subaru', 'toyota']

# 按字母逆序排序
cars.sort(reverse=True)
print(cars)  # ['toyota', 'subaru', 'bmw', 'audi']
```

### 临时排序 — `sorted()`

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']

print(sorted(cars))  # ['audi', 'bmw', 'subaru', 'toyota']
print(cars)          # ['bmw', 'audi', 'toyota', 'subaru']（原列表不变）
```

### 反转列表 — `reverse()`

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']
cars.reverse()
print(cars)  # ['subaru', 'toyota', 'audi', 'bmw']
```

<aside>
💡

`reverse()` **永久修改** 列表顺序，但可以再次调用 `reverse()` 恢复。

</aside>

### 确定列表长度 — `len()`

```python
cars = ['bmw', 'audi', 'toyota', 'subaru']
print(len(cars))  # 4
```

---

## 3.4 使用列表时避免索引错误

<aside>
🚫

访问不存在的索引会导致 **IndexError**：

```python
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles[3])  # ❌ IndexError: list index out of range
```

特别注意：**空列表** 没有任何元素，访问 `[-1]` 也会报错。

</aside>

---

## 📝 本章小结

| 概念 | 要点 |
| --- | --- |
| 列表定义 | 用 `[]` 定义，索引从 `0` 开始，`-1` 访问最后一个元素 |
| 修改元素 | 通过索引直接赋值 |
| 添加元素 | `append()`（末尾）、`insert()`（指定位置） |
| 删除元素 | `del`（索引，不保留）、`pop()`（索引，保留）、`remove()`（值，保留） |
| 排序 | `sort()`（永久）、`sorted()`（临时），均支持 `reverse=True` |
| 其他操作 | `reverse()` 反转、`len()` 获取长度 |
| 常见错误 | 注意 IndexError，尤其是空列表 |