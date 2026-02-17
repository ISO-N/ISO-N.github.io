---
title: 第17章 使用API
date: 2026-02-17 15:04:58
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 17.1 使用 Web API

<aside>
💡

**API（Application Programming Interface）** 是应用程序编程接口。**Web API** 是网站的一部分，用于与使用特定 URL 请求数据的程序交互。这种请求称为 **API 调用**。请求的数据通常以 **JSON** 或 **CSV** 等易于处理的格式返回。

</aside>

### Git 和 GitHub

- **Git** 是一个分布式版本控制系统，帮助管理项目代码的不同版本
- **GitHub** 是一个基于 Git 的代码托管平台，提供了 Web API 供开发者使用
- GitHub 上星标（star）最多的项目代表最受关注的项目

### 使用 API 调用请求数据

GitHub 的 API 调用示例：

```
https://api.github.com/search/repositories?q=language:python&sort=stars
```

- `https://api.github.com/` — API 的基础 URL
- `search/repositories` — 搜索仓库的端点
- `?` — 表示后面是查询参数
- `q=language:python` — 查询条件：语言为 Python
- `&sort=stars` — 按星标数排序

---

## 17.2 安装 Requests

<aside>
📦

**Requests** 是一个第三方 Python 库，让程序能够轻松地向网站请求信息并检查返回的响应。

</aside>

```bash
pip install requests
```

---

## 17.3 处理 API 响应

```python
import requests

# 执行 API 调用并存储响应
url = "https://api.github.com/search/repositories"
url += "?q=language:python+sort:stars+stars:>10000"

headers = {"Accept": "application/vnd.github.v3+json"}
r = requests.get(url, headers=headers)

# 查看状态码（200 表示成功）
print(f"Status code: {r.status_code}")

# 将响应转换为字典
response_dict = r.json()

print(f"Total repositories: {response_dict['total_count']}")
```

<aside>
✅

**状态码 200** 表示请求成功。常见状态码：

- `200` — 请求成功
- `403` — 超过 API 速率限制
- `404` — 资源不存在
</aside>

---

## 17.4 处理响应字典

响应字典包含三个键：

```python
print(response_dict.keys())
# dict_keys(['total_count', 'incomplete_results', 'items'])
```

- `total_count` — 符合条件的仓库总数
- `incomplete_results` — 请求是否超时（布尔值）
- `items` — 包含仓库详细信息的列表

### 探索仓库信息

```python
repo_dicts = response_dict['items']

# 查看第一个仓库的信息
repo_dict = repo_dicts[0]
print(f"\nKeys: {len(repo_dict)}")
for key in sorted(repo_dict.keys()):
    print(key)
```

### 提取关键字段

```python
for repo_dict in repo_dicts:
    print(f"\nName: {repo_dict['name']}")
    print(f"Owner: {repo_dict['owner']['login']}")
    print(f"Stars: {repo_dict['stargazers_count']}")
    print(f"Repository: {repo_dict['html_url']}")
    print(f"Description: {repo_dict['description']}")
```

---

## 17.5 监视 API 的速率限制

<aside>
⚠️

GitHub 的 API 速率限制：**未认证用户每分钟最多 60 次请求**，认证后可达 5000 次/小时。可通过以下 URL 查看当前限制状态：

`https://api.github.com/rate_limit`

</aside>

---

## 17.6 使用 Plotly 可视化仓库

```python
import requests
from plotly.graph_objs import Bar
from plotly import offline

# 执行 API 调用
url = "https://api.github.com/search/repositories"
url += "?q=language:python+sort:stars+stars:>10000"

headers = {"Accept": "application/vnd.github.v3+json"}
r = requests.get(url, headers=headers)
print(f"Status code: {r.status_code}")

response_dict = r.json()
repo_dicts = response_dict['items']

# 提取数据
repo_names, stars, hover_texts = [], [], []
for repo_dict in repo_dicts:
    repo_names.append(repo_dict['name'])
    stars.append(repo_dict['stargazers_count'])
    
    # 悬停文本
    owner = repo_dict['owner']['login']
    description = repo_dict['description']
    hover_text = f"{owner}<br />{description}"
    hover_texts.append(hover_text)
```

### 设置图表样式

```python
# 创建可视化
data = [{
    'type': 'bar',
    'x': repo_names,
    'y': stars,
    'hovertext': hover_texts,
    'marker': {
        'color': 'rgb(60, 100, 150)',
        'line': {'width': 1.5, 'color': 'rgb(25, 25, 25)'}
    },
    'opacity': 0.6,
}]

my_layout = {
    'title': 'GitHub上最受欢迎的Python项目',
    'titlefont': {'size': 28},
    'xaxis': {
        'title': 'Repository',
        'titlefont': {'size': 24},
        'tickfont': {'size': 14},
    },
    'yaxis': {
        'title': 'Stars',
        'titlefont': {'size': 24},
        'tickfont': {'size': 14},
    },
}

fig = {'data': data, 'layout': my_layout}
offline.plot(fig, filename='python_repos.html')
```

### 添加可点击的链接

```python
repo_links = []
for repo_dict in repo_dicts:
    repo_name = repo_dict['name']
    repo_url = repo_dict['html_url']
    repo_link = f"<a href='{repo_url}'>{repo_name}</a>"
    repo_links.append(repo_link)

# 将 x 轴的 repo_names 替换为 repo_links
data = [{
    'type': 'bar',
    'x': repo_links,   # 使用带链接的名称
    'y': stars,
    'hovertext': hover_texts,
    ...
}]
```

---

## 17.7 Hacker News API

<aside>
📰

**Hacker News**（[https://news.ycombinator.com/）是一个技术新闻网站，其](https://news.ycombinator.com/）是一个技术新闻网站，其) API 无需注册即可使用。

</aside>

### API 端点

```
https://hacker-news.firebaseio.com/v0/item/31353677.json
```

- 每篇文章都有唯一 ID
- 返回的字典包含标题、URL、评分、评论等信息

### 获取热门文章

```python
from operator import itemgetter
import requests

# 获取当前热门文章的 ID 列表
url = "https://hacker-news.firebaseio.com/v0/topstories.json"
r = requests.get(url)
print(f"Status code: {r.status_code}")

# 获取每篇文章的详细信息
submission_ids = r.json()

submission_dicts = []
for submission_id in submission_ids[:30]:
    url = f"https://hacker-news.firebaseio.com/v0/item/{submission_id}.json"
    r = requests.get(url)
    print(f"  id: {submission_id}\tstatus: {r.status_code}")
    response_dict = r.json()
    
    # 创建文章信息字典
    submission_dict = {
        'title': response_dict['title'],
        'hn_link': f"https://news.ycombinator.com/item?id={submission_id}",
        'comments': response_dict.get('descendants', 0),
    }
    submission_dicts.append(submission_dict)

# 按评论数排序
submission_dicts = sorted(submission_dicts, 
                          key=itemgetter('comments'),
                          reverse=True)

for submission_dict in submission_dicts:
    print(f"\nTitle: {submission_dict['title']}")
    print(f"Discussion link: {submission_dict['hn_link']}")
    print(f"Comments: {submission_dict['comments']}")
```

<aside>
🔑

**`dict.get()` 方法**：使用 `response_dict.get('descendants', 0)` 而非直接访问键，可以在键不存在时返回默认值 `0`，避免 `KeyError`。

</aside>

---

## 本章要点总结

| **概念** | **说明** |
| --- | --- |
| API 调用 | 通过 URL 向 Web 服务请求数据 |
| Requests 库 | `requests.get(url, headers)` 发起 GET 请求 |
| JSON 响应 | `r.json()` 将响应体转换为 Python 字典/列表 |
| 状态码 | `r.status_code` 检查请求是否成功（200 = 成功） |
| 速率限制 | API 对请求频率有限制，需注意避免超限 |
| Plotly 可视化 | 将 API 数据用 Plotly 绘制为交互式图表 |
| Hacker News API | 无需认证即可使用的公开 API 示例 |