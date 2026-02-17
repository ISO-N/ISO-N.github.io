---
title: 第9章 类
date: 2026-02-17 15:04:52
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 9.1 创建和使用类

### 创建 Dog 类

<aside>
💡

根据约定，在 Python 中，**首字母大写**的名称指的是类（大驼峰命名法）。

</aside>

```python
class Dog:
    """一次模拟小狗的简单尝试"""

    def __init__(self, name, age):
        """初始化属性 name 和 age"""
        self.name = name
        self.age = age

    def sit(self):
        """模拟小狗收到命令时蹲下"""
        print(f"{self.name} is now sitting.")

    def roll_over(self):
        """模拟小狗收到命令时打滚"""
        print(f"{self.name} rolled over!")
```

- **`__init__()` 方法**：每次根据类创建新实例时，Python 都会自动运行它
- **`self` 参数**：必须位于其他形参前面。Python 调用方法时会自动传入 `self`，它是一个指向实例本身的引用
- 以 `self` 为前缀的变量可供类中所有方法使用，称为 **属性**

### 根据类创建实例

```python
my_dog = Dog('Willie', 6)

# 访问属性
print(f"My dog's name is {my_dog.name}.")

# 调用方法
my_dog.sit()
my_dog.roll_over()
```

- 可以根据一个类创建**任意数量**的实例

---

## 9.2 使用类和实例

### 给属性指定默认值

```python
class Car:
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year
        self.odometer_reading = 0  # 默认值
```

### 修改属性的值

- **方式一：直接修改**

```python
my_car.odometer_reading = 23
```

- **方式二：通过方法修改**

```python
def update_odometer(self, mileage):
    """设置里程表读数，禁止往回调"""
    if mileage >= self.odometer_reading:
        self.odometer_reading = mileage
    else:
        print("You can't roll back an odometer!")
```

- **方式三：通过方法递增**

```python
def increment_odometer(self, miles):
    """将里程表读数增加指定的量"""
    self.odometer_reading += miles
```

---

## 9.3 继承

### 子类的 `__init__()` 方法

<aside>
📌

创建子类时，父类必须包含在当前文件中，且位于子类前面。

</aside>

```python
class ElectricCar(Car):
    """电动汽车的独特之处"""

    def __init__(self, make, model, year):
        """初始化父类的属性，再初始化电动汽车特有的属性"""
        super().__init__(make, model, year)
        self.battery_size = 40
```

- **`super()`**：调用父类的 `__init__()` 方法，让子类继承父类的所有属性和方法

### 给子类定义属性和方法

子类可以添加区别于父类的**新属性**和**新方法**：

```python
class ElectricCar(Car):
    def __init__(self, make, model, year):
        super().__init__(make, model, year)
        self.battery_size = 40

    def describe_battery(self):
        """打印一条描述电池容量的消息"""
        print(f"This car has a {self.battery_size}-kWh battery.")
```

### 重写父类的方法

子类中定义一个与父类同名的方法，Python 会忽略父类方法，只关注子类中定义的版本：

```python
class ElectricCar(Car):
    # ...
    def fill_gas_tank(self):
        """电动汽车没有油箱"""
        print("This car doesn't need a gas tank!")
```

### 将实例用作属性

当类的属性和方法越来越多时，可以将其中一部分提取出来，作为一个独立的类：

```python
class Battery:
    """一次模拟电动汽车电池的简单尝试"""

    def __init__(self, battery_size=40):
        self.battery_size = battery_size

    def describe_battery(self):
        print(f"This car has a {self.battery_size}-kWh battery.")

    def get_range(self):
        if self.battery_size == 40:
            range = 150
        elif self.battery_size == 65:
            range = 225
        print(f"This car can go about {range} miles on a full charge.")

class ElectricCar(Car):
    def __init__(self, make, model, year):
        super().__init__(make, model, year)
        self.battery = Battery()  # 将 Battery 实例作为属性
```

```python
my_leaf = ElectricCar('nissan', 'leaf', 2024)
my_leaf.battery.describe_battery()
my_leaf.battery.get_range()
```

---

## 9.4 导入类

<aside>
📂

将类存储在**模块**中，然后在主程序中导入所需的模块，可以让文件更简洁清晰。

</aside>

### 导入单个类

```python
from car import Car
```

### 在一个模块中存储多个类

```python
from car import ElectricCar
```

### 从一个模块中导入多个类

```python
from car import Car, ElectricCar
```

### 导入整个模块

```python
import car

my_beetle = car.Car('volkswagen', 'beetle', 2019)
```

### 导入模块中的所有类

```python
from car import *
```

<aside>
⚠️

**不推荐**这种导入方式。可能引发名称冲突，且难以判断使用了哪些类。推荐导入整个模块后用 `module_name.ClassName` 访问。

</aside>

### 在一个模块中导入另一个模块

当一个模块中的类依赖另一个模块中的类时，可以在前一个模块中导入后一个模块：

```python
# electric_car.py
from car import Car

class ElectricCar(Car):
    # ...
```

### 使用别名

```python
from electric_car import ElectricCar as EC
```

---

## 9.5 Python 标准库

<aside>
📚

**Python 标准库**是一组模块，可以使用 `import` 语句导入。可以使用标准库中的任何函数和类。

</aside>

### `random` 模块示例

```python
from random import randint, choice

# 生成 1~6 之间的随机整数
randint(1, 6)

# 从列表中随机选取一个元素
players = ['charles', 'martina', 'michael', 'florence', 'eli']
choice(players)
```

---

## 9.6 类的编码风格

| **规则** | **说明** | **示例** |
| --- | --- | --- |
| 类名 | 采用**大驼峰命名法**（每个单词首字母大写，不使用下划线） | `ElectricCar` |
| 实例名和模块名 | 采用**小写 + 下划线**格式 | `my_car`、`electric_car.py` |
| 文档字符串 | 每个类和模块都应包含文档字符串，简要描述其功能 | `"""一次模拟汽车的简单尝试"""` |
| 空行 | 类中用**一个空行**分隔方法；模块中用**两个空行**分隔类 | — |
| 导入顺序 | 先导入**标准库模块**，空一行，再导入**自定义模块** | — |