---
title: 第15章 生成数据
date: 2026-02-17 15:04:57
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
本章介绍如何使用 **Matplotlib** 和 **Plotly** 生成数据可视化图表，包括折线图、散点图、随机漫步以及掷骰子的统计直方图。

---

## 15.1 安装 Matplotlib

使用 pip 安装：

```bash
python -m pip install matplotlib
```

---

## 15.2 绘制简单的折线图

使用 `matplotlib.pyplot` 模块绘制平方数折线图：

```python
import matplotlib.pyplot as plt

squares = [1, 4, 9, 16, 25]
fig, ax = plt.subplots()
ax.plot(squares)
plt.show()
```

### 修改标签文字和线条粗细

- `ax.set_title()` —— 设置图表标题
- `ax.set_xlabel()` / `ax.set_ylabel()` —— 设置坐标轴标签
- `ax.tick_params()` —— 设置刻度标记的样式
- `linewidth` 参数控制线条粗细

```python
fig, ax = plt.subplots()
ax.plot(squares, linewidth=3)

ax.set_title("Square Numbers", fontsize=24)
ax.set_xlabel("Value", fontsize=14)
ax.set_ylabel("Square of Value", fontsize=14)
ax.tick_params(labelsize=14)

plt.show()
```

### 校正图形

当向 `plot()` 只传递一个数据列表时，Matplotlib 默认从 **x=0** 开始绘制，但实际上第一个数据点对应 **x=1**。可以同时提供输入值和输出值：

```python
input_values = [1, 2, 3, 4, 5]
squares = [1, 4, 9, 16, 25]
ax.plot(input_values, squares, linewidth=3)
```

---

## 15.3 绘制散点图

### 使用 scatter() 绘制单个点和一系列点

```python
fig, ax = plt.subplots()
ax.scatter(2, 4)
plt.show()
```

绘制一系列点：

```python
x_values = [1, 2, 3, 4, 5]
y_values = [1, 4, 9, 16, 25]

fig, ax = plt.subplots()
ax.scatter(x_values, y_values, s=100)
plt.show()
```

### 自动计算数据

使用循环或列表推导式生成大量数据点：

```python
x_values = range(1, 1001)
y_values = [x**2 for x in x_values]

fig, ax = plt.subplots()
ax.scatter(x_values, y_values, s=10)

# 设置坐标轴范围
ax.axis([0, 1100, 0, 1_100_000])
plt.show()
```

### 自定义颜色

- 使用 `color` 参数设置颜色，如 `color='red'` 或 RGB 元组 `color=(0, 0.8, 0)`
- 使用 **颜色映射（colormap）** 根据 y 值变化着色：

```python
ax.scatter(x_values, y_values, c=y_values, cmap=plt.cm.Blues, s=10)
```

### 自动保存图表

使用 `plt.savefig()` 替代 `plt.show()` 将图表保存为文件：

```python
plt.savefig('squares_plot.png', bbox_inches='tight')
```

> `bbox_inches='tight'` 表示裁剪掉图表周围多余的空白区域。
> 

---

## 15.4 随机漫步

**随机漫步（Random Walk）** 指每一步的方向和距离都是随机决定的路径，在自然界、物理学和生物学中有广泛应用。

### 创建 RandomWalk 类

```python
from random import choice

class RandomWalk:
    """生成随机漫步数据的类"""

    def __init__(self, num_points=5000):
        self.num_points = num_points
        self.x_values = [0]
        self.y_values = [0]

    def fill_walk(self):
        while len(self.x_values) < self.num_points:
            # 决定方向和距离
            x_direction = choice([1, -1])
            x_distance = choice([0, 1, 2, 3, 4])
            x_step = x_direction * x_distance

            y_direction = choice([1, -1])
            y_distance = choice([0, 1, 2, 3, 4])
            y_step = y_direction * y_distance

            # 拒绝原地踏步
            if x_step == 0 and y_step == 0:
                continue

            x = self.x_values[-1] + x_step
            y = self.y_values[-1] + y_step

            self.x_values.append(x)
            self.y_values.append(y)
```

### 绘制随机漫步

```python
rw = RandomWalk()
rw.fill_walk()

fig, ax = plt.subplots()
ax.scatter(rw.x_values, rw.y_values, s=15)
plt.show()
```

### 模拟多次随机漫步

使用 `while` 循环反复模拟，让用户决定是否继续：

```python
while True:
    rw = RandomWalk()
    rw.fill_walk()
    # ...绘制代码...
    keep_running = input("Make another walk? (y/n): ")
    if keep_running == 'n':
        break
```

### 设置随机漫步的样式

<aside>
💡

关键美化技巧：

- 使用 `c=list(range(rw.num_points))` 加 `cmap` 表示**时间先后顺序**
- 突出显示**起点**和**终点**
- 使用 `ax.get_xaxis().set_visible(False)` 隐藏坐标轴
- 使用 `fig, ax = plt.subplots(figsize=(15, 9))` 调整窗口尺寸
</aside>

```python
point_numbers = list(range(rw.num_points))
ax.scatter(rw.x_values, rw.y_values,
           c=point_numbers, cmap=plt.cm.Blues,
           edgecolors='none', s=1)

# 突出起点和终点
ax.scatter(0, 0, c='green', edgecolors='none', s=100)
ax.scatter(rw.x_values[-1], rw.y_values[-1],
           c='red', edgecolors='none', s=100)
```

---

## 15.5 使用 Plotly 模拟掷骰子

**Plotly** 生成的可视化图表是交互式的，适合在浏览器中展示。

### 安装 Plotly

```bash
python -m pip install plotly
```

### 创建 Die 类

```python
from random import randint

class Die:
    """表示一个骰子的类"""

    def __init__(self, num_sides=6):
        self.num_sides = num_sides

    def roll(self):
        """返回一个介于 1 和面数之间的随机值"""
        return randint(1, self.num_sides)
```

### 掷骰子并分析结果

```python
die = Die()
results = [die.roll() for _ in range(1000)]

# 分析结果
frequencies = [results.count(value) for value in range(1, die.num_sides + 1)]
```

### 绘制直方图

使用 `plotly.express` 绘制交互式条形图：

```python
import plotly.express as px

die = Die()
results = [die.roll() for _ in range(1000)]
frequencies = [results.count(value) for value in range(1, die.num_sides + 1)]

title = "Results of Rolling One D6 1,000 Times"
labels = {'x': 'Result', 'y': 'Frequency of Result'}
fig = px.bar(x=list(range(1, 7)), y=frequencies,
             title=title, labels=labels)
fig.update_layout(xaxis_dtick=1)
fig.show()
```

### 同时掷两个骰子

将两个骰子的结果相加，分析其频率分布：

```python
die_1 = Die()
die_2 = Die()

results = [die_1.roll() + die_2.roll() for _ in range(1000)]
max_result = die_1.num_sides + die_2.num_sides
poss_results = range(2, max_result + 1)
frequencies = [results.count(value) for value in poss_results]
```

> 两个 D6 相加的结果中，**7** 出现的频率最高，**2** 和 **12** 出现频率最低。
> 

### 掷两个面数不同的骰子

可以创建不同面数的骰子来观察概率分布的变化：

```python
die_1 = Die()
die_2 = Die(10)  # 一个 D6 + 一个 D10

results = [die_1.roll() + die_2.roll() for _ in range(50_000)]
```

### 保存图表

使用 `fig.write_html()` 将 Plotly 图表保存为 HTML 文件：

```python
fig.write_html('dice_visual.html')
```

---

## 本章小结

| **主题** | **工具** | **要点** |
| --- | --- | --- |
| 折线图 / 散点图 | Matplotlib | `plot()` 绘制折线，`scatter()` 绘制散点；支持自定义颜色、颜色映射、坐标轴设置 |
| 随机漫步 | Matplotlib | 用类封装随机漫步逻辑，通过颜色映射展示时间序列，隐藏坐标轴聚焦路径 |
| 掷骰子模拟 | Plotly | 用 `plotly.express` 绘制交互式条形图，分析概率分布，支持导出 HTML |