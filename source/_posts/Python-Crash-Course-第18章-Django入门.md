---
title: 第18章 Django入门
date: 2026-02-17 15:04:59
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 18.1 建立项目

### 项目简介：学习笔记（Learning Log）

- 本章开始构建一个名为 **"学习笔记"（Learning Log）** 的 Web 应用程序
- 用户可以记录感兴趣的主题，并在学习每个主题的过程中添加日志条目
- 学习笔记的主页将描述该网站，并邀请用户注册或登录
- 登录后，用户可以创建新主题、添加新条目以及阅读已有条目

### 建立虚拟环境

创建项目目录，并在其中创建虚拟环境：

```bash
mkdir ll_project
cd ll_project
python -m venv ll_env
```

### 激活虚拟环境

```bash
# Windows
ll_env\Scripts\activate

# macOS / Linux
source ll_env/bin/activate
```

<aside>
💡

要停止使用虚拟环境，可执行命令 `deactivate`。

</aside>

### 安装 Django

在虚拟环境激活的状态下安装 Django：

```bash
pip install django
```

### 在 Django 中创建项目

```bash
django-admin startproject ll_project .
```

<aside>
📌

末尾的**句点（`.`）** 非常重要，它让 Django 在当前目录中创建项目，而不是新建一个子目录。

</aside>

项目结构：

```
ll_project/
├── ll_project/
│   ├── __init__.py
│   ├── settings.py     # 项目整体设置
│   ├── urls.py         # 项目级 URL 配置
│   ├── wsgi.py         # 部署相关
│   └── asgi.py
├── manage.py           # 管理工具
└── ll_env/             # 虚拟环境
```

### 创建数据库

```bash
python manage.py migrate
```

<aside>
💡

Django 默认使用 **SQLite** 数据库，首次运行 `migrate` 会创建数据库文件 `db.sqlite3`，并建立存储项目信息所需的数据表。

</aside>

### 查看项目

```bash
python manage.py runserver
```

在浏览器中访问 `http://localhost:8000/`，如果看到 Django 欢迎页面，说明项目创建成功。

---

## 18.2 创建应用程序

### 创建应用

在虚拟环境激活的状态下，保持开发服务器运行，再开一个终端窗口：

```bash
python manage.py startapp learning_logs
```

应用目录结构：

```
learning_logs/
├── __init__.py
├── admin.py        # 管理站点配置
├── apps.py
├── models.py       # 定义模型（数据）
├── tests.py
└── views.py        # 定义视图（页面逻辑）
```

### 定义模型

#### Topic 模型

编辑 `learning_logs/models.py`：

```python
from django.db import models

class Topic(models.Model):
    """用户学习的主题"""
    text = models.CharField(max_length=200)
    date_added = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        """返回模型的字符串表示"""
        return self.text
```

| **字段类型** | **说明** |
| --- | --- |
| `CharField` | 字符字段，需指定 `max_length` |
| `DateTimeField` | 日期时间字段，`auto_now_add=True` 表示自动记录创建时间 |
| `TextField` | 文本字段，不限长度 |
| `ForeignKey` | 外键，建立模型之间的关联 |

### 激活模型

1. 将应用添加到项目的 `settings.py` 中：

```python
# ll_project/settings.py
INSTALLED_APPS = [
    # 我的应用程序
    'learning_logs',
    # Django 默认应用
    'django.contrib.admin',
    ...
]
```

1. 让 Django 修改数据库，使其能够存储 Topic 模型的信息：

```bash
python manage.py makemigrations learning_logs
python manage.py migrate
```

<aside>
📌

每次修改模型后，都需要执行 `makemigrations` 和 `migrate` 两步操作来更新数据库。

</aside>

### Django 管理网站

#### 创建超级用户

```bash
python manage.py createsuperuser
```

按提示输入用户名、邮箱和密码。

#### 向管理网站注册模型

编辑 `learning_logs/admin.py`：

```python
from django.contrib import admin
from .models import Topic

admin.site.register(Topic)
```

访问 `http://localhost:8000/admin/` 即可管理 Topic 数据。

### 定义 Entry 模型

```python
class Entry(models.Model):
    """学到的有关某个主题的具体知识"""
    topic = models.ForeignKey(Topic, on_delete=models.CASCADE)
    text = models.TextField()
    date_added = models.DateTimeField(auto_now_add=True)

    class Meta:
        verbose_name_plural = 'entries'

    def __str__(self):
        """返回一个表示条目的简单字符串"""
        return f"{self.text[:50]}..."
```

<aside>
💡

- `ForeignKey` 将每个条目关联到特定主题，`on_delete=models.CASCADE` 表示删除主题时，所有关联条目也会被删除（**级联删除**）。
- `class Meta` 中的 `verbose_name_plural` 用于设置复数形式，避免 Django 自动生成的 `Entrys`。
</aside>

迁移并注册 Entry 模型：

```bash
python manage.py makemigrations learning_logs
python manage.py migrate
```

```python
# admin.py
from .models import Topic, Entry

admin.site.register(Topic)
admin.site.register(Entry)
```

---

## 18.3 创建网页：学习笔记主页

Django 创建网页的过程分为三步：**定义 URL → 编写视图 → 编写模板**。

### 映射 URL

#### 项目级 URL 配置

编辑 `ll_project/urls.py`：

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('learning_logs.urls')),
]
```

#### 应用级 URL 配置

创建 `learning_logs/urls.py`：

```python
"""定义 learning_logs 的 URL 模式"""
from django.urls import path
from . import views

app_name = 'learning_logs'
urlpatterns = [
    # 主页
    path('', views.index, name='index'),
]
```

### 编写视图

编辑 `learning_logs/views.py`：

```python
from django.shortcuts import render

def index(request):
    """学习笔记的主页"""
    return render(request, 'learning_logs/index.html')
```

### 编写模板

创建目录 `learning_logs/templates/learning_logs/`，然后创建 `index.html`：

```html
<p>Learning Log</p>
<p>Learning Log helps you keep track of your learning,
   for any topic you're learning about.</p>
```

---

## 18.4 创建其他网页

### 模板继承

#### 父模板 `base.html`

```html
<p>
  <a href="{% url 'learning_logs:index' %}">Learning Log</a>
</p>

{% block content %}{% endblock content %}
```

#### 子模板 `index.html`（改写）

```html
{% extends "learning_logs/base.html" %}

{% block content %}
  <p>Learning Log helps you keep track of your learning,
     for any topic you're learning about.</p>
{% endblock content %}
```

<aside>
💡

- `{% extends %}` 让子模板继承父模板的结构
- `{% block content %}...{% endblock %}` 定义子模板可覆盖的区域
- `{% url %}` 模板标签根据 URL 配置自动生成链接
</aside>

### 显示所有主题的页面

#### URL 模式

```python
# learning_logs/urls.py
urlpatterns = [
    path('', views.index, name='index'),
    path('topics/', views.topics, name='topics'),
]
```

#### 视图

```python
from .models import Topic

def topics(request):
    """显示所有主题"""
    topics = Topic.objects.order_by('date_added')
    context = {'topics': topics}
    return render(request, 'learning_logs/topics.html', context)
```

#### 模板 `topics.html`

```html
{% extends "learning_logs/base.html" %}

{% block content %}
  <p>Topics</p>
  <ul>
    {% for topic in topics %}
      <li>
        <a href="{% url 'learning_logs:topic' topic.id %}">
           topic.text 
        </a>
      </li>
    {% empty %}
      <li>No topics have been added yet.</li>
    {% endfor %}
  </ul>
{% endblock content %}
```

### 显示特定主题的页面

#### URL 模式

```python
# learning_logs/urls.py
urlpatterns = [
    path('', views.index, name='index'),
    path('topics/', views.topics, name='topics'),
    path('topics/<int:topic_id>/', views.topic, name='topic'),
]
```

<aside>
📌

`<int:topic_id>` 是 Django 的**路径转换器**，会匹配一个整数并将其作为参数传递给视图函数。

</aside>

#### 视图

```python
def topic(request, topic_id):
    """显示单个主题及其所有条目"""
    topic = Topic.objects.get(id=topic_id)
    entries = topic.entry_set.order_by('-date_added')
    context = {'topic': topic, 'entries': entries}
    return render(request, 'learning_logs/topic.html', context)
```

#### 模板 `topic.html`

```html
{% extends "learning_logs/base.html" %}

{% block content %}
  <p>Topic:  topic.text </p>

  <p>Entries:</p>
  <ul>
    {% for entry in entries %}
      <li>
        <p> entry.date_added|date:'M d, Y H:i' </p>
        <p> entry.text|linebreaks </p>
      </li>
    {% empty %}
      <li>There are no entries for this topic yet.</li>
    {% endfor %}
  </ul>
{% endblock content %}
```

<aside>
💡

- `entry.date_added|date:'M d, Y H:i'`：使用 Django 的 **date 过滤器** 格式化日期
- `entry.text|linebreaks`：将文本中的换行符转换为 HTML 换行标签
- `entry_set`：通过外键反向访问关联的 Entry 对象集合
</aside>

---

## 本章小结

| **核心概念** | **说明** |
| --- | --- |
| 虚拟环境 | 使用 `python -m venv` 创建隔离的 Python 环境 |
| 项目 vs 应用 | 一个 Django **项目**可包含多个**应用**，`startproject` 创建项目，`startapp` 创建应用 |
| 模型（Model） | 用 Python 类定义数据结构，Django 自动管理数据库 |
| 迁移（Migration） | `makemigrations`  • `migrate` 将模型变更同步到数据库 |
| 管理网站（Admin） | Django 自带的后台管理界面，方便管理数据 |
| URL 映射 | 通过 `urls.py` 定义 URL 模式，将请求路由到对应视图 |
| 视图（View） | 处理请求、获取数据、返回响应的函数 |
| 模板（Template） | HTML 文件 + Django 模板语言，用于呈现页面 |
| 模板继承 | 使用 `{% extends %}` 和 `{% block %}` 实现模板复用 |

---

- [x]  使用虚拟环境搭建 Django 开发环境
- [x]  使用 `django-admin startproject` 创建项目
- [x]  定义模型（`Topic`、`Entry`）并迁移数据库
- [x]  创建超级用户并使用 Django 管理网站管理数据
- [x]  掌握 **URL → 视图 → 模板** 的网页创建流程
- [x]  使用模板继承减少重复代码
- [x]  创建主页、主题列表页和主题详情页