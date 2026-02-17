---
title: 第11章 测试代码
date: 2026-02-17 15:04:54
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
<aside>
📖

本章学习如何使用 Python 的 **pytest** 库编写测试，验证函数和类的行为是否符合预期。测试让你确信代码在越来越多的人使用程序时依然能正确工作。

</aside>

## 11.1 使用 pip 安装 pytest

**pytest** 是一个第三方库，需要使用 **pip** 安装：

```bash
python -m pip install --user pytest
```

<aside>
💡

`pip` 是 Python 的包管理工具，用于安装第三方包。`--user` 标志表示仅为当前用户安装。

</aside>

## 11.2 测试函数

### 要测试的函数

假设有一个格式化名字的函数：

```python
# name_function.py
def get_formatted_name(first, last):
    """生成格式化的姓名"""
    full_name = f"{first} {last}"
    return full_name.title()
```

### 编写测试

测试文件必须以 **test_** 开头，测试函数也必须以 **test_** 开头：

```python
# test_name_function.py
from name_function import get_formatted_name

def test_first_last_name():
    """能够正确处理像 Janis Joplin 这样的名字吗？"""
    formatted_name = get_formatted_name('janis', 'joplin')
    assert formatted_name == 'Janis Joplin'
```

### 运行测试

```bash
python -m pytest
```

- pytest 会自动查找以 `test_` 开头的文件和函数
- 测试通过显示 **绿色 PASSED**，失败显示 **红色 FAILED**

### 使用 `assert` 语句

`assert` 是测试的核心。如果断言为真，测试通过；如果为假，测试失败：

```python
assert formatted_name == 'Janis Joplin'   # 如果相等，通过
assert formatted_name != ''                # 如果不为空，通过
```

## 11.3 测试失败时怎么办

<aside>
⚠️

**测试失败是好事！** 它意味着你的测试发现了问题。不要修改测试让它通过，而是修复代码中的 bug。

</aside>

假设给函数添加中间名参数后，原测试可能失败：

```python
def get_formatted_name(first, middle, last):
    """生成格式化的姓名"""
    full_name = f"{first} {middle} {last}"
    return full_name.title()
```

### 修复方法：让中间名变为可选

```python
def get_formatted_name(first, last, middle=''):
    """生成格式化的姓名"""
    if middle:
        full_name = f"{first} {middle} {last}"
    else:
        full_name = f"{first} {last}"
    return full_name.title()
```

### 添加新测试

```python
def test_first_last_middle_name():
    """能够正确处理像 Wolfgang Amadeus Mozart 这样的名字吗？"""
    formatted_name = get_formatted_name('wolfgang', 'mozart', 'amadeus')
    assert formatted_name == 'Wolfgang Amadeus Mozart'
```

## 11.4 测试类

### 常用的断言方法

| **断言语句** | **说明** |
| --- | --- |
| `assert a == b` | 断言两个值相等 |
| `assert a != b` | 断言两个值不相等 |
| `assert a` | 断言 a 为真 |
| `not assert a` | 断言 a 为假 |
| `assert element in list` | 断言元素在列表中 |
| `assert element not in list` | 断言元素不在列表中 |

### 要测试的类

```python
# survey.py
class AnonymousSurvey:
    """收集匿名调查问卷的答案"""
    
    def __init__(self, question):
        """存储一个问题，并为存储答案做准备"""
        self.question = question
        self.responses = []
    
    def show_question(self):
        """显示调查问卷"""
        print(self.question)
    
    def store_response(self, new_response):
        """存储单份调查答案"""
        self.responses.append(new_response)
    
    def show_results(self):
        """显示收集到的所有答案"""
        print("Survey results:")
        for response in self.responses:
            print(f"- {response}")
```

### 测试类

```python
# test_survey.py
from survey import AnonymousSurvey

def test_store_single_response():
    """测试单个答案会被妥善存储"""
    question = "What language did you first learn to speak?"
    my_survey = AnonymousSurvey(question)
    my_survey.store_response('English')
    assert 'English' in my_survey.responses

def test_store_three_responses():
    """测试三个答案会被妥善存储"""
    question = "What language did you first learn to speak?"
    my_survey = AnonymousSurvey(question)
    responses = ['English', 'Spanish', 'Mandarin']
    for response in responses:
        my_survey.store_response(response)
    for response in responses:
        assert response in my_survey.responses
```

## 11.5 使用夹具（Fixture）

<aside>
🔧

**夹具（fixture）** 用于创建测试中重复使用的资源。使用 `@pytest.fixture` 装饰器定义夹具，pytest 会在测试函数运行前自动调用它。

</aside>

```python
import pytest
from survey import AnonymousSurvey

@pytest.fixture
def language_survey():
    """创建一个可供所有测试函数使用的 AnonymousSurvey 实例"""
    question = "What language did you first learn to speak?"
    language_survey = AnonymousSurvey(question)
    return language_survey

def test_store_single_response(language_survey):
    """测试单个答案会被妥善存储"""
    language_survey.store_response('English')
    assert 'English' in language_survey.responses

def test_store_three_responses(language_survey):
    """测试三个答案会被妥善存储"""
    responses = ['English', 'Spanish', 'Mandarin']
    for response in responses:
        language_survey.store_response(response)
    for response in responses:
        assert response in language_survey.responses
```

### 夹具的工作原理

1. 使用 `@pytest.fixture` 装饰器标记函数
2. 测试函数的**参数名**与夹具函数名相同
3. pytest 自动将夹具的返回值注入到测试函数中
4. 每个测试函数都会获得一个**全新的**夹具实例，确保测试之间互不影响

---

## 📝 本章小结

| **概念** | **要点** |
| --- | --- |
| 安装 pytest | `python -m pip install --user pytest` |
| 命名规则 | 测试文件和函数均以 `test_` 开头 |
| 断言 | 使用 `assert` 语句验证预期结果 |
| 运行测试 | `python -m pytest` 或指定文件 `python -m pytest test_xxx.py` |
| 夹具 (fixture) | 用 `@pytest.fixture` 创建共享资源，减少重复代码 |
| 测试失败 | 修复代码而不是修改测试 |