---
title: 第16章 下载数据
date: 2026-02-17 15:04:57
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 16.1 CSV 文件格式

**CSV**（Comma-Separated Values，逗号分隔值）是一种简单的数据格式，将数据作为一系列以逗号分隔的值写入文件。

### 解析 CSV 文件头

使用 Python 标准库中的 `csv` 模块读取 CSV 文件：

```python
from pathlib import Path
import csv

path = Path('weather_data/sitka_weather_07-2021_simple.csv')
lines = path.read_text().splitlines()

reader = csv.reader(lines)
header_row = next(reader)
print(header_row)
```

<aside>
💡

`csv.reader()` 创建一个与文件关联的阅读器对象。`next()` 函数返回文件中的下一行，第一次调用时返回**表头行**（第一行）。

</aside>

使用 `enumerate()` 获取每个表头及其索引位置：

```python
for index, column_header in enumerate(header_row):
    print(index, column_header)
```

### 提取并读取数据

获取表头后，可以按列索引提取所需数据：

```python
path = Path('weather_data/sitka_weather_07-2021_simple.csv')
lines = path.read_text().splitlines()

reader = csv.reader(lines)
header_row = next(reader)

# 提取最高温度（TMAX 列）
highs = []
for row in reader:
    high = int(row[4])
    highs.append(high)

print(highs)
```

### 绘制温度图表

使用 **Matplotlib** 将提取的数据可视化：

```python
import matplotlib.pyplot as plt

plt.style.use('seaborn-v0_8')
fig, ax = plt.subplots()
ax.plot(highs, color='red')

# 设置图表格式
ax.set_title("2021年7月每日最高温度", fontsize=24)
ax.set_xlabel('', fontsize=16)
ax.set_ylabel("温度 (°F)", fontsize=16)
ax.tick_params(labelsize=16)

plt.show()
```

---

## 16.2 绘制日期

### datetime 模块

使用 `datetime` 模块处理日期数据：

```python
from datetime import datetime

first_date = datetime.strptime('2021-07-01', '%Y-%m-%d')
print(first_date)
```

<aside>
📌

**常用日期格式化参数：**

- `%Y` — 四位数年份（如 2021）
- `%m` — 两位数月份（01~12）
- `%d` — 两位数日期（01~31）
- `%A` — 星期几的完整名称（如 Monday）
- `%B` — 月份的完整名称（如 January）
</aside>

### 在图表中添加日期

将日期字符串转换为 `datetime` 对象后，Matplotlib 能正确渲染日期坐标轴：

```python
from datetime import datetime
import csv
from pathlib import Path
import matplotlib.pyplot as plt

path = Path('weather_data/sitka_weather_07-2021_simple.csv')
lines = path.read_text().splitlines()

reader = csv.reader(lines)
header_row = next(reader)

dates, highs = [], []
for row in reader:
    current_date = datetime.strptime(row[2], '%Y-%m-%d')
    high = int(row[4])
    dates.append(current_date)
    highs.append(high)

plt.style.use('seaborn-v0_8')
fig, ax = plt.subplots()
ax.plot(dates, highs, color='red')

ax.set_title("2021年7月每日最高温度", fontsize=24)
ax.set_xlabel('', fontsize=16)
fig.autofmt_xdate()  # 自动格式化日期标签（斜向显示）
ax.set_ylabel("温度 (°F)", fontsize=16)
ax.tick_params(labelsize=16)

plt.show()
```

<aside>
💡

`fig.autofmt_xdate()` 会自动旋转日期标签，防止重叠，使图表更易读。

</aside>

### 绘制更长时间段的数据

只需更换数据文件即可绘制更长时间跨度的图表，例如全年数据。

### 绘制最高温度和最低温度

在同一张图中同时展示最高温和最低温：

```python
dates, highs, lows = [], [], []
for row in reader:
    current_date = datetime.strptime(row[2], '%Y-%m-%d')
    high = int(row[4])
    low = int(row[5])
    dates.append(current_date)
    highs.append(high)
    lows.append(low)

ax.plot(dates, highs, color='red', alpha=0.5)
ax.plot(dates, lows, color='blue', alpha=0.5)
```

### 给图表区域着色

使用 `fill_between()` 在两条折线之间**填充颜色**，直观展示温差范围：

```python
ax.plot(dates, highs, color='red', alpha=0.5)
ax.plot(dates, lows, color='blue', alpha=0.5)
ax.fill_between(dates, highs, lows, facecolor='blue', alpha=0.1)
```

<aside>
💡

`alpha` 参数控制颜色透明度（0 为全透明，1 为不透明），使填充区域不会遮挡数据线。

</aside>

---

## 16.3 错误检查

有些数据集中可能包含**缺失数据**或**格式错误**的数据。使用 `try-except` 跳过无效行：

```python
dates, highs, lows = [], [], []
for row in reader:
    current_date = datetime.strptime(row[2], '%Y-%m-%d')
    try:
        high = int(row[4])
        low = int(row[5])
    except ValueError:
        print(f"Missing data for {current_date}")
    else:
        dates.append(current_date)
        highs.append(high)
        lows.append(low)
```

<aside>
⚠️

处理真实世界数据时，**错误检查不可或缺**。缺失值、格式不一致、损坏的数据都可能导致程序崩溃。使用异常处理可以让程序在遇到坏数据时跳过并继续。

</aside>

---

## 16.4 自己动手下载数据

数据可以从多种在线来源获取。书中使用的天气数据来自 **NOAA**（美国国家海洋和大气管理局）的气候数据在线服务。

<aside>
📌

**获取数据的常见来源：**

- NOAA 气候数据
- 世界银行公开数据
- 美国地质调查局（USGS）地震数据
- Kaggle 开放数据集
</aside>

---

## 16.5 映射全球数据集：GeoJSON 格式

### 下载地震数据

**GeoJSON** 是一种基于 JSON 的地理数据格式，常用于存储地震、地理边界等地理空间信息。美国地质调查局（USGS）提供 GeoJSON 格式的实时地震数据。

```python
from pathlib import Path
import json

# 将数据作为字符串读取并转换为 Python 对象
path = Path('eq_data/eq_data_1_day_m1.geojson')
contents = path.read_text()
all_eq_data = json.loads(contents)

# 以更易读的方式查看数据结构
path = Path('eq_data/readable_eq_data.geojson')
readable_contents = json.dumps(all_eq_data, indent=4)
path.write_text(readable_contents)
```

### 查看 GeoJSON 数据结构

GeoJSON 文件的核心结构：

```json
{
    "type": "FeatureCollection",
    "metadata": { "title": "...", "count": 158 },
    "features": [
        {
            "type": "Feature",
            "properties": {
                "mag": 1.6,
                "place": "22 km SE of Pahala, Hawaii",
                "title": "M 1.6 - 22 km SE of Pahala, Hawaii"
            },
            "geometry": {
                "type": "Point",
                "coordinates": [-155.21, 19.28, 0.07]
            }
        },
        ...
    ]
}
```

<aside>
💡

GeoJSON 中 `features` 列表包含所有地震记录。每条记录的 `properties` 包含震级和位置描述，`geometry` 包含经纬度坐标。

</aside>

### 创建地震列表

提取所有地震数据：

```python
all_eq_dicts = all_eq_data['features']
print(len(all_eq_dicts))
```

### 提取震级

遍历地震列表提取震级：

```python
mags = []
for eq_dict in all_eq_dicts:
    mag = eq_dict['properties']['mag']
    mags.append(mag)

print(mags[:10])
```

### 提取位置数据

从 `geometry` 键中提取经纬度：

```python
lons, lats = [], []
for eq_dict in all_eq_dicts:
    lon = eq_dict['geometry']['coordinates'][0]
    lat = eq_dict['geometry']['coordinates'][1]
    lons.append(lon)
    lats.append(lat)

print(lons[:5])
print(lats[:5])
```

<aside>
⚠️

注意 GeoJSON 中坐标顺序是 **[经度, 纬度, 深度]**，而不是常见的"纬度, 经度"顺序。

</aside>

---

## 16.6 绘制世界地图

### 使用 Plotly 绘制散点图

**Plotly** 是一个功能强大的可视化库，支持交互式图表。使用 `plotly.express` 绑定 Pandas DataFrame 绘图：

```python
import plotly.express as px
import pandas as pd

data = pd.DataFrame(
    data=zip(lons, lats, mags), columns=['经度', '纬度', '震级']
)
data.head()

fig = px.scatter(
    data,
    x='经度',
    y='纬度',
    range_x=[-200, 200],
    range_y=[-90, 90],
    width=800,
    height=800,
    title='全球地震散点图',
)
fig.write_html('global_earthquakes.html')
fig.show()
```

<aside>
💡

`plotly.express` 通常使用 **Pandas DataFrame** 作为数据源。`zip()` 函数将多个列表合并为元组序列，方便创建 DataFrame。

</aside>

### 自定义标记大小

根据震级调整散点大小，让可视化更有信息量：

```python
fig = px.scatter(
    data,
    x='经度',
    y='纬度',
    range_x=[-200, 200],
    range_y=[-90, 90],
    width=800,
    height=800,
    title='全球地震散点图',
    size='震级',
    size_max=10,
)
```

### 自定义标记颜色

使用**连续色标**根据震级设置颜色深浅：

```python
fig = px.scatter(
    data,
    x='经度',
    y='纬度',
    range_x=[-200, 200],
    range_y=[-90, 90],
    width=800,
    height=800,
    title='全球地震散点图',
    size='震级',
    size_max=10,
    color='震级',
)
```

### 添加悬停文本

添加鼠标悬停时显示的地震位置信息：

```python
# 提取位置描述
titles = []
for eq_dict in all_eq_dicts:
    title = eq_dict['properties']['title']
    titles.append(title)

data = pd.DataFrame(
    data=zip(lons, lats, mags, titles),
    columns=['经度', '纬度', '震级', '位置']
)

fig = px.scatter(
    data,
    x='经度',
    y='纬度',
    range_x=[-200, 200],
    range_y=[-90, 90],
    width=800,
    height=800,
    title='全球地震散点图',
    size='震级',
    size_max=10,
    color='震级',
    hover_name='位置',
)
fig.write_html('global_earthquakes.html')
fig.show()
```

<aside>
📌

**Plotly 交互功能：** 生成的 HTML 文件可在浏览器中打开，支持缩放、平移和鼠标悬停查看详情，非常适合探索地理数据。

</aside>

---

## 本章小结

| **主题** | **关键知识点** |
| --- | --- |
| CSV 文件 | `csv.reader()` 读取 CSV，`next()` 获取表头，按索引提取列数据 |
| 日期处理 | `datetime.strptime()` 解析日期字符串，`fig.autofmt_xdate()` 格式化日期轴 |
| Matplotlib 绑定 | `plot()` 折线图，`fill_between()` 区域着色，`alpha` 控制透明度 |
| 错误检查 | `try-except-else` 跳过缺失或异常数据行 |
| GeoJSON 格式 | JSON 结构存储地理数据，`features` → `properties`  • `geometry` |
| Plotly 可视化 | `plotly.express.scatter()` 绘制交互式散点图，支持大小、颜色、悬停文本 |