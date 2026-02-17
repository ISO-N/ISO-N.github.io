---
title: 第20章 设置应用程序的样式并部署
date: 2026-02-17 15:05:01
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 20.1 设置项目"学习笔记"的样式

### 使用 django-bootstrap5

**Bootstrap** 是一个大型的样式工具集，可用于为 Web 应用程序设置样式，使其外观更专业，且在不同设备上表现一致。

使用 `django-bootstrap5` 应用程序将 Bootstrap 集成到 Django 项目中：

```bash
pip install django-bootstrap5
```

在 `settings.py` 中注册应用：

```python
INSTALLED_APPS = [
    # 我的应用程序
    'learning_logs',
    'accounts',

    # 第三方应用程序
    'django_bootstrap5',

    # Django 默认添加的应用程序
    'django.contrib.admin',
    # ...
]
```

---

### 修改 base.html 模板

使用 Bootstrap 重写基础模板，需要引入 Bootstrap 模板标签并使用 Bootstrap 的导航栏组件：

```html
{% load django_bootstrap5 %}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Learning Log</title>
    {% bootstrap_css %}
    {% bootstrap_javascript %}
</head>

<body>
    <nav class="navbar navbar-expand-md navbar-light bg-light mb-4 border-bottom">
        <div class="container-fluid">
            <a class="navbar-brand" href="{% url 'learning_logs:index' %}">
                Learning Log</a>

            <button class="navbar-toggler" type="button"
                    data-bs-toggle="collapse"
                    data-bs-target="#navbarCollapse"
                    aria-controls="navbarCollapse"
                    aria-expanded="false"
                    aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
            </button>

            <div class="collapse navbar-collapse" id="navbarCollapse">
                <ul class="navbar-nav me-auto mb-2 mb-md-0">
                    <li class="nav-item">
                        <a class="nav-link" href="{% url 'learning_logs:topics' %}">
                            Topics</a>
                    </li>
                </ul>

                <!-- 账户相关链接 -->
                <ul class="navbar-nav ms-auto mb-2 mb-md-0">
                    {% if user.is_authenticated %}
                        <li class="nav-item">
                            <span class="navbar-text me-2">
                                Hello,  user.username .</span>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="{% url 'accounts:logout' %}">
                                Log out</a>
                        </li>
                    {% else %}
                        <li class="nav-item">
                            <a class="nav-link" href="{% url 'accounts:register' %}">
                                Register</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="{% url 'accounts:login' %}">
                                Log in</a>
                        </li>
                    {% endif %}
                </ul>
            </div>
        </div>
    </nav>

    <main class="container">
        <div class="pb-2 mb-2 border-bottom">
            {% block page_header %}{% endblock page_header %}
        </div>
        <div>
            {% block content %}{% endblock content %}
        </div>
    </main>
</body>
</html>
```

<aside>
💡

**关键元素说明：**

- `{% bootstrap_css %}` 和 `{% bootstrap_javascript %}`：加载 Bootstrap 的 CSS 和 JS 文件
- `navbar-expand-md`：在中等及以上屏幕展开导航栏，在小屏幕折叠
- `me-auto` / `ms-auto`：Bootstrap 的 margin 工具类，用于控制对齐
- `container`：Bootstrap 的容器类，让内容居中且有合适的边距
</aside>

---

### 使用 Jumbotron 设置主页样式

使用 Bootstrap 的样式为主页创建一个醒目的大标题区域：

```html
{% extends "learning_logs/base.html" %}

{% block page_header %}
    <div class="p-3 mb-4 bg-light border rounded-3">
        <div class="container-fluid py-3">
            <h1 class="display-3">Track your learning.</h1>
            <p class="lead">Make your own Learning Log, and keep a list of the
                topics you're learning about. Whenever you learn something new
                about a topic, make an entry summarizing what you've
                learned.</p>
            <a class="btn btn-primary btn-lg mt-1"
                href="{% url 'accounts:register' %}">Register &raquo;</a>
        </div>
    </div>
{% endblock page_header %}
```

---

### 设置登录页面的样式

使用 `{% bootstrap_form %}` 标签自动为表单添加 Bootstrap 样式：

```html
{% extends "learning_logs/base.html" %}
{% load django_bootstrap5 %}

{% block page_header %}
    <h2>Log in to your account.</h2>
{% endblock page_header %}

{% block content %}
    <form action="{% url 'accounts:login' %}" method="post">
        {% csrf_token %}
        {% bootstrap_form form %}
        {% bootstrap_button button_type="submit" content="Log in" %}
    </form>
{% endblock content %}
```

<aside>
📌

`{% bootstrap_form form %}` 会自动为表单字段添加 Bootstrap 样式，省去手动添加 CSS 类的麻烦。`{% bootstrap_button %}` 则用于渲染带样式的按钮。

</aside>

---

### 设置其他页面的样式

对 `topics.html`、`topic.html`、`new_topic.html`、`new_entry.html`、`edit_entry.html` 和 `register.html` 等页面应用类似的 Bootstrap 样式：

| **页面** | **样式要点** |
| --- | --- |
| topics.html | 使用 `page_header` 块添加标题，主题列表无需额外样式 |
| topic.html | 使用 `card` 组件展示每个条目，用 `card-header` 和 `card-body` 分区 |
| new_topic.html / new_entry.html | 使用 `{% bootstrap_form %}` 和 `{% bootstrap_button %}` |
| edit_entry.html | 同上，表单页面采用一致的 Bootstrap 表单样式 |
| register.html | 同登录页面风格，使用 `{% bootstrap_form %}` 渲染注册表单 |

---

## 20.2 部署"学习笔记"

### 创建 [Platform.sh](http://Platform.sh) 账户

<aside>
🌍

[**Platform.sh**](http://Platform.sh) 是一个现代化的云托管平台，支持 Django 项目的部署。部署过程由 YAML 配置文件控制，这与专业程序员部署现代 Django 项目的方式一致。

</aside>

1. 前往 [Platform.sh](http://Platform.sh) 注册账户
2. 免费试用期为 **30 天**，无需信用卡
3. 试用期内限制为 24 小时内最多 2 个应用

---

### 安装 [Platform.sh](http://Platform.sh) CLI

```bash
pip install platformshconfig
```

还需安装 [Platform.sh](http://Platform.sh) 命令行工具（CLI）：

```bash
curl -fsSL https://raw.githubusercontent.com/platformsh/cli/main/installer.sh | bash
```

安装后使用 CLI 登录：

```bash
platform login
```

---

### 安装必要的依赖

部署需要几个额外的包：

```bash
pip install platformshconfig gunicorn psycopg2
```

| **包名** | **用途** |
| --- | --- |
| `platformshconfig` | 帮助检测应用是否在 [Platform.sh](http://Platform.sh) 上运行，并读取部署环境信息 |
| `gunicorn` | 生产级的 Python WSGI HTTP 服务器，替代 Django 开发服务器 |
| `psycopg2` | PostgreSQL 数据库适配器，生产环境使用 PostgreSQL 替代 SQLite |

生成 `requirements.txt`：

```bash
pip freeze > requirements.txt
```

---

### 创建配置文件

部署需要 **三个 YAML 配置文件**：

### `.platform.app.yaml`（应用配置）

```yaml
name: "ll_project"
type: "python:3.10"
	
web:
    upstream:
        socket_family: unix
    commands:
        start: "gunicorn -w 4 -b unix:$SOCKET ll_project.wsgi:application"
    locations:
        "/":
            passthru: true
        "/static":
            root: "static"
            expires: 1h
            allow: true
	
hooks:
    build: |
        pip install -r requirements.txt
        python manage.py collectstatic --noinput
    deploy: |
        python manage.py migrate
	
disk: 512
	
mounts:
    "logs":
        source: local
        source_path: logs
```

### `.platform/routes.yaml`（路由配置）

```yaml
"https://{default}/":
    type: upstream
    upstream: "ll_project:http"
	
"https://www.{default}/":
    type: redirect
    to: "https://{default}/"
```

### `.platform/services.yaml`（服务配置）

```yaml
db:
    type: postgresql:12
    disk: 1024
```

<aside>
📂

**目录结构：** `.platform.app.yaml` 放在项目根目录，`routes.yaml` 和 `services.yaml` 放在项目根目录下的 `.platform/` 文件夹中。

</aside>

---

### 修改 [settings.py](http://settings.py) 以适配部署

在 `settings.py` 中根据环境动态配置：

```python
from platformshconfig import Config

config = Config()

if config.is_valid_platform():
    ALLOWED_HOSTS = ['.platformsh.site']
    DEBUG = False
    STATIC_ROOT = os.path.join(BASE_DIR, 'static')
```

<aside>
⚠️

**重要安全设置：**

- 部署时务必将 `DEBUG` 设为 `False`，否则错误页面会暴露项目的敏感信息
- `ALLOWED_HOSTS` 限制哪些主机可以访问应用
- 使用 `SECRET_KEY` 环境变量替代硬编码的密钥
</aside>

---

### 使用 Git 进行版本控制

[Platform.sh](http://Platform.sh) 使用 **Git** 管理部署：

```bash
# 初始化仓库
git init

# 创建 .gitignore 文件
echo "ll_env/
__pycache__/
*.sqlite3
.DS_Store" > .gitignore

# 提交代码
git add .
git commit -m "Ready for deployment to Platform.sh."
```

---

### 推送到 [Platform.sh](http://Platform.sh)

```bash
# 创建 Platform.sh 项目
platform create

# 推送部署
platform push
```

<aside>
🚀

推送后 [Platform.sh](http://Platform.sh) 会自动：

1. 安装 `requirements.txt` 中的依赖
2. 运行 `collectstatic` 收集静态文件
3. 运行 `migrate` 执行数据库迁移
4. 启动 `gunicorn` 服务器
</aside>

---

### 查看在线应用

```bash
platform url
```

此命令会输出部署后的应用 URL，在浏览器中打开即可查看。

---

### 完善部署

### 在 [Platform.sh](http://Platform.sh) 上创建超级用户

在 [Platform.sh](http://Platform.sh) 上通过 SSH 连接创建超级用户：

```bash
platform ssh
python manage.py createsuperuser
```

### 确保项目安全

生产环境的关键安全措施：

| **设置** | **说明** |
| --- | --- |
| `DEBUG = False` | 关闭调试模式，避免泄露敏感信息 |
| `SECRET_KEY` | 使用环境变量存储，不要硬编码在代码中 |
| `ALLOWED_HOSTS` | 只允许特定域名访问 |
| HTTPS | [Platform.sh](http://Platform.sh) 默认提供 HTTPS，确保数据传输安全 |

### 持续部署

后续更新只需：

```bash
git add .
git commit -m "Updated styling."
git push platform main
```

[Platform.sh](http://Platform.sh) 会自动重新构建和部署应用。

---

## 20.3 本章小结

- [x]  使用 **django-bootstrap5** 为 Django 项目添加 Bootstrap 5 样式
- [x]  修改基础模板 `base.html`，使用 Bootstrap 的**导航栏**和**容器**组件
- [x]  使用 `{% bootstrap_form %}` 和 `{% bootstrap_button %}` 模板标签为**表单**添加样式
- [x]  了解 [**Platform.sh**](http://Platform.sh) 作为部署平台的基本概念
- [x]  创建 YAML 配置文件（`.platform.app.yaml`、`routes.yaml`、`services.yaml`）
- [x]  使用 **Git** 管理代码并推送到 [Platform.sh](http://Platform.sh) 进行部署
- [x]  掌握生产环境的**安全设置**：`DEBUG`、`SECRET_KEY`、`ALLOWED_HOSTS`
- [x]  学会通过 SSH 在远程服务器上**创建超级用户**
- [x]  理解**持续部署**的工作流程