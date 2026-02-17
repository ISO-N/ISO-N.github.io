---
title: 第12章 武装飞船
date: 2026-02-17 15:04:54
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 12.1 规划项目

> 开发大型项目时，最好先做好规划，以便编写代码时有清晰的方向。
> 

### 项目描述：外星人入侵

- 玩家控制一艘出现在屏幕底部中央的飞船
- 玩家可以使用**方向键**左右移动飞船，使用**空格键**射击
- 游戏开始时，一群外星人出现在天空中，并向屏幕下方移动
- 玩家的任务是射杀这些外星人
- 玩家消灭所有外星人后，将出现一群新的、移动速度更快的外星人
- 如果有外星人撞到玩家的飞船或到达屏幕底部，玩家将损失一艘飞船
- 玩家损失三艘飞船后，游戏结束

<aside>
📁

本章开始构建游戏的基本框架：创建游戏窗口、添加飞船、实现飞船移动和射击子弹。

</aside>

---

## 12.2 安装 Pygame

使用 `pip` 安装 Pygame：

```bash
python -m pip install pygame
```

<aside>
💡

如果使用多个 Python 版本，确保使用正确的 `pip`，如 `python3 -m pip install pygame`。

</aside>

---

## 12.3 开始游戏项目

### 12.3.1 创建 Pygame 窗口及响应用户输入

创建一个空的 Pygame 窗口，这是游戏的基础：

```python
import sys
import pygame

class AlienInvasion:
    """管理游戏资源和行为的类"""

    def __init__(self):
        """初始化游戏并创建游戏资源"""
        pygame.init()

        self.screen = pygame.display.set_mode((1200, 800))
        pygame.display.set_caption("Alien Invasion")

    def run_game(self):
        """开始游戏的主循环"""
        while True:
            # 监视键盘和鼠标事件
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    sys.exit()

            # 让最近绘制的屏幕可见
            pygame.display.flip()

if __name__ == '__main__':
    ai = AlienInvasion()
    ai.run_game()
```

| **关键概念** | **说明** |
| --- | --- |
| `pygame.init()` | 初始化 Pygame 的所有模块 |
| `pygame.display.set_mode()` | 创建显示窗口，参数为元组 `(宽, 高)` |
| `pygame.event.get()` | 获取事件队列中的所有事件 |
| `pygame.display.flip()` | 刷新屏幕，显示最新绘制的内容 |

### 12.3.2 设置背景色

在 `__init__()` 中设置背景色，并在每次循环中填充屏幕：

```python
# 设置背景色（RGB 元组）
self.bg_color = (230, 230, 230)
```

在 `run_game()` 的主循环中调用 `fill()`：

```python
# 每次循环时都重绘屏幕
self.screen.fill(self.bg_color)
```

<aside>
🎨

Pygame 中颜色使用 **RGB** 值表示，每个分量范围为 0~255。`(230, 230, 230)` 是浅灰色。

</aside>

### 12.3.3 创建设置类

为了将所有设置集中管理，创建一个 `Settings` 类：

```python
# settings.py
class Settings:
    """存储游戏所有设置的类"""

    def __init__(self):
        """初始化游戏的设置"""
        # 屏幕设置
        self.screen_width = 1200
        self.screen_height = 800
        self.bg_color = (230, 230, 230)
```

在 `AlienInvasion` 中使用设置：

```python
from settings import Settings

class AlienInvasion:
    def __init__(self):
        pygame.init()
        self.settings = Settings()
        self.screen = pygame.display.set_mode(
            (self.settings.screen_width, self.settings.screen_height))
```

<aside>
📌

将设置抽取到独立类中，方便后续统一修改游戏参数，无需在代码中到处查找。

</aside>

---

## 12.4 添加飞船图像

### 12.4.1 创建 Ship 类

```python
# ship.py
import pygame

class Ship:
    """管理飞船的类"""

    def __init__(self, ai_game):
        """初始化飞船并设置其初始位置"""
        self.screen = ai_game.screen
        self.screen_rect = ai_game.screen.get_rect()

        # 加载飞船图像并获取其外接矩形
        self.image = pygame.image.load('images/ship.bmp')
        self.rect = self.image.get_rect()

        # 每艘新飞船都放在屏幕底部的中央
        self.rect.midbottom = self.screen_rect.midbottom

    def blitme(self):
        """在指定位置绘制飞船"""
        self.screen.blit(self.image, self.rect)
```

### 12.4.2 在屏幕上绘制飞船

在 `AlienInvasion` 的 `__init__()` 中创建飞船实例，并在主循环中绘制：

```python
from ship import Ship

# 在 __init__() 中
self.ship = Ship(self)

# 在 run_game() 中，fill 之后，flip 之前
self.ship.blitme()
```

<aside>
🖼️

**Pygame 中的 `rect`（矩形）** 是定位和操作游戏元素的核心工具。常用定位属性：

- `center`、`centerx`、`centery`
- `top`、`bottom`、`left`、`right`
- `topleft`、`topright`、`bottomleft`、`bottomright`
- `midbottom`、`midtop`、`midleft`、`midright`
</aside>

---

## 12.5 重构：方法 `_check_events()` 和 `_update_screen()`

随着代码增长，将 `run_game()` 中的逻辑拆分到辅助方法中：

```python
def run_game(self):
    """开始游戏的主循环"""
    while True:
        self._check_events()
        self._update_screen()

def _check_events(self):
    """响应按键和鼠标事件"""
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()

def _update_screen(self):
    """更新屏幕上的图像，并切换到新屏幕"""
    self.screen.fill(self.settings.bg_color)
    self.ship.blitme()
    pygame.display.flip()
```

<aside>
♻️

**重构** 是在不改变功能的前提下优化代码结构。以下划线 `_` 开头的方法名表示**辅助方法**（仅在类内部使用）。

</aside>

---

## 12.6 驾驶飞船

### 12.6.1 响应按键

在 `_check_events()` 中添加按键检测：

```python
def _check_events(self):
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            sys.exit()
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_RIGHT:
                self.ship.moving_right = True
        elif event.type == pygame.KEYUP:
            if event.key == pygame.K_RIGHT:
                self.ship.moving_right = False
```

### 12.6.2 允许持续移动

在 `Ship` 类中添加移动标志和 `update()` 方法：

```python
class Ship:
    def __init__(self, ai_game):
        # ...
        # 移动标志（飞船一开始不动）
        self.moving_right = False
        self.moving_left = False

    def update(self):
        """根据移动标志调整飞船位置"""
        if self.moving_right:
            self.rect.x += 1
        if self.moving_left:
            self.rect.x -= 1
```

在 `run_game()` 中调用：

```python
while True:
    self._check_events()
    self.ship.update()
    self._update_screen()
```

<aside>
💡

使用 `KEYDOWN` 和 `KEYUP` 事件配合布尔标志，实现按住方向键**持续移动**。

</aside>

### 12.6.3 调整飞船速度

使用浮点数控制速度（`rect` 只能存储整数，所以用单独的属性存储精确位置）：

```python
# settings.py
self.ship_speed = 1.5

# ship.py
def __init__(self, ai_game):
    # ...
    self.settings = ai_game.settings
    # 在飞船的属性 x 中存储一个浮点数
    self.x = float(self.rect.x)

def update(self):
    if self.moving_right:
        self.x += self.settings.ship_speed
    if self.moving_left:
        self.x -= self.settings.ship_speed
    # 根据 self.x 更新 rect 对象
    self.rect.x = self.x
```

### 12.6.4 限制飞船的活动范围

在 `update()` 中添加边界检查：

```python
def update(self):
    if self.moving_right and self.rect.right < self.screen_rect.right:
        self.x += self.settings.ship_speed
    if self.moving_left and self.rect.left > 0:
        self.x -= self.settings.ship_speed
    self.rect.x = self.x
```

### 12.6.5 重构 `_check_events()`

将按键处理逻辑拆分为 `_check_keydown_events()` 和 `_check_keyup_events()`：

```python
def _check_keydown_events(self, event):
    """响应按下"""
    if event.key == pygame.K_RIGHT:
        self.ship.moving_right = True
    elif event.key == pygame.K_LEFT:
        self.ship.moving_left = True
    elif event.key == pygame.K_q:
        sys.exit()

def _check_keyup_events(self, event):
    """响应释放"""
    if event.key == pygame.K_RIGHT:
        self.ship.moving_right = False
    elif event.key == pygame.K_LEFT:
        self.ship.moving_left = False
```

<aside>
⌨️

添加按 **Q** 键退出游戏的功能，方便开发时快速关闭窗口。

</aside>

### 12.6.6 在全屏模式下运行游戏

```python
def __init__(self):
    pygame.init()
    self.settings = Settings()
    self.screen = pygame.display.set_mode((0, 0), pygame.FULLSCREEN)
    self.settings.screen_width = self.screen.get_rect().width
    self.settings.screen_height = self.screen.get_rect().height
```

---

## 12.7 简单回顾

本项目的文件结构：

| **文件** | **说明** |
| --- | --- |
| `alien_invasion.py` | 主程序文件，包含 `AlienInvasion` 类和游戏主循环 |
| `settings.py` | `Settings` 类，集中管理所有游戏设置 |
| `ship.py` | `Ship` 类，管理飞船的行为 |

---

## 12.8 射击

### 12.8.1 添加子弹设置

在 `Settings` 类中添加子弹相关设置：

```python
# 子弹设置
self.bullet_speed = 2.0
self.bullet_width = 3
self.bullet_height = 15
self.bullet_color = (60, 60, 60)
self.bullets_allowed = 3
```

### 12.8.2 创建 Bullet 类

```python
# bullet.py
import pygame
from pygame.sprite import Sprite

class Bullet(Sprite):
    """管理飞船所发射子弹的类"""

    def __init__(self, ai_game):
        """在飞船当前位置创建一个子弹对象"""
        super().__init__()
        self.screen = ai_game.screen
        self.settings = ai_game.settings
        self.color = self.settings.bullet_color

        # 在 (0,0) 处创建一个表示子弹的矩形，再设置正确的位置
        self.rect = pygame.Rect(0, 0,
            self.settings.bullet_width, self.settings.bullet_height)
        self.rect.midtop = ai_game.ship.rect.midtop

        # 用浮点数存储子弹的 y 坐标
        self.y = float(self.rect.y)

    def update(self):
        """向上移动子弹"""
        self.y -= self.settings.bullet_speed
        self.rect.y = self.y

    def draw_bullet(self):
        """在屏幕上绘制子弹"""
        pygame.draw.rect(self.screen, self.color, self.rect)
```

<aside>
🧩

`Bullet` 继承自 `pygame.sprite.Sprite`，这样可以使用 Pygame 的精灵分组功能来高效管理多个子弹。

</aside>

### 12.8.3 将子弹存储到编组中

在 `AlienInvasion` 中使用 `pygame.sprite.Group` 管理子弹：

```python
from pygame.sprite import Group

# 在 __init__() 中
self.bullets = Group()

# 在 run_game() 中
self.ship.update()
self.bullets.update()
self._update_screen()
```

### 12.8.4 开火

按空格键发射子弹：

```python
def _check_keydown_events(self, event):
    # ...
    elif event.key == pygame.K_SPACE:
        self._fire_bullet()

def _fire_bullet(self):
    """创建一颗子弹，并将其加入编组 bullets"""
    if len(self.bullets) < self.settings.bullets_allowed:
        new_bullet = Bullet(self)
        self.bullets.add(new_bullet)
```

在 `_update_screen()` 中绘制子弹：

```python
for bullet in self.bullets.sprites():
    bullet.draw_bullet()
```

### 12.8.5 删除消失的子弹

子弹飞出屏幕后需要删除，否则会占用越来越多的内存：

```python
# 在 run_game() 中，bullets.update() 之后
# 删除已消失的子弹
for bullet in self.bullets.copy():
    if bullet.rect.bottom <= 0:
        self.bullets.remove(bullet)
```

<aside>
⚠️

遍历列表时不能直接删除元素，需要对 `.copy()` 进行遍历，在原列表中删除。

</aside>

### 12.8.6 限制子弹数量

通过 `self.settings.bullets_allowed` 控制屏幕上同时存在的子弹数量，在 `_fire_bullet()` 中检查：

```python
if len(self.bullets) < self.settings.bullets_allowed:
    new_bullet = Bullet(self)
    self.bullets.add(new_bullet)
```

### 12.8.7 创建方法 `_update_bullets()`

将子弹相关逻辑封装到辅助方法中：

```python
def _update_bullets(self):
    """更新子弹的位置并删除消失的子弹"""
    self.bullets.update()
    # 删除消失的子弹
    for bullet in self.bullets.copy():
        if bullet.rect.bottom <= 0:
            self.bullets.remove(bullet)
```

最终 `run_game()` 简洁清晰：

```python
def run_game(self):
    while True:
        self._check_events()
        self.ship.update()
        self._update_bullets()
        self._update_screen()
```

---

## 本章小结

- [x]  学习了如何制定游戏开发计划
- [x]  安装 **Pygame** 并创建游戏窗口
- [x]  使用 `Settings` 类集中管理游戏设置
- [x]  添加飞船图像并使用 **rect** 定位游戏元素
- [x]  实现飞船的**左右移动**和**速度控制**
- [x]  掌握了**重构**代码的技巧，保持代码结构清晰
- [x]  创建 `Bullet` 类，实现**射击功能**
- [x]  使用 `pygame.sprite.Group` 管理子弹编组
- [x]  限制子弹数量并删除飞出屏幕的子弹