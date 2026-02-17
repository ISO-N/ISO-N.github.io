---
title: 第14章 记分
date: 2026-02-17 15:04:56
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 14.1 添加 Play 按钮

目前游戏在运行 `alien_invasion.py` 时立即开始，我们需要添加一个 **Play 按钮**，让玩家可以选择何时开始游戏，并在游戏结束后重新开始。

### 创建 Button 类

创建 `button.py` 模块，用于在屏幕上绘制带文本的按钮：

```python
import pygame.font

class Button:
    def __init__(self, ai_game, msg):
        """初始化按钮的属性"""
        self.screen = ai_game.screen
        self.screen_rect = self.screen.get_rect()

        # 设置按钮的尺寸和其他属性
        self.width, self.height = 200, 50
        self.button_color = (0, 255, 0)
        self.text_color = (255, 255, 255)
        self.font = pygame.font.SysFont(None, 48)

        # 创建按钮的 rect 对象，并使其居中
        self.rect = pygame.Rect(0, 0, self.width, self.height)
        self.rect.center = self.screen_rect.center

        # 按钮的标签只需创建一次
        self._prep_msg(msg)

    def _prep_msg(self, msg):
        """将 msg 渲染为图像，并使其在按钮上居中"""
        self.msg_image = self.font.render(msg, True,
                self.text_color, self.button_color)
        self.msg_image_rect = self.msg_image.get_rect()
        self.msg_image_rect.center = self.rect.center

    def draw_button(self):
        """绘制一个用颜色填充的按钮，再绘制文本"""
        self.screen.fill(self.button_color, self.rect)
        self.screen.blit(self.msg_image, self.msg_image_rect)
```

<aside>
💡

`pygame.font.SysFont(None, 48)` 使用默认字体、48 号字。`font.render()` 将文本转换为图像，便于在屏幕上绘制。

</aside>

### 在屏幕上绘制按钮

在 `AlienInvasion` 类中创建按钮实例，并在游戏未激活时绘制：

```python
from button import Button

class AlienInvasion:
    def __init__(self):
        --snip--
        self.play_button = Button(self, "Play")
```

在 `_update_screen()` 中，当游戏处于非活动状态时绘制按钮：

```python
def _update_screen(self):
    --snip--
    if not self.stats.game_active:
        self.play_button.draw_button()
    pygame.display.flip()
```

### 开始游戏

在 `_check_events()` 中检测玩家是否点击了 Play 按钮：

```python
def _check_play_button(self, mouse_pos):
    """在玩家单击 Play 按钮时开始新游戏"""
    if self.play_button.rect.collidepoint(mouse_pos):
        self.stats.reset_stats()
        self.stats.game_active = True

        # 清空余下的外星人和子弹
        self.aliens.empty()
        self.bullets.empty()

        # 创建一群新的外星人并让飞船居中
        self._create_fleet()
        self.ship.center_ship()
```

### 重置游戏

每次点击 Play 按钮时，需要重置游戏的统计信息和速度设置，以便玩家可以重新开始。

### 将 Play 按钮切换为非活动状态

当游戏正在进行时，即使玩家不小心点到了 Play 按钮区域，游戏也不应重置：

```python
def _check_play_button(self, mouse_pos):
    button_clicked = self.play_button.rect.collidepoint(mouse_pos)
    if button_clicked and not self.stats.game_active:
        self.stats.reset_stats()
        self.stats.game_active = True
        --snip--
```

### 隐藏光标

游戏活动时隐藏鼠标光标，游戏结束时重新显示：

```python
# 游戏开始时隐藏光标
pygame.mouse.set_visible(False)

# 游戏结束时显示光标
pygame.mouse.set_visible(True)
```

---

## 14.2 提高等级

每当玩家将屏幕上的外星人消灭干净后，游戏应 **加快节奏**，提升难度。

### 修改速度设置

在 `Settings` 类中添加动态速度设置，区分 **静态设置** 和 **动态设置**：

```python
class Settings:
    def __init__(self):
        """初始化游戏的静态设置"""
        # 屏幕设置
        self.screen_width = 1200
        self.screen_height = 800
        self.bg_color = (230, 230, 230)

        # 飞船设置
        self.ship_limit = 3

        # 子弹设置
        self.bullet_width = 3
        self.bullet_height = 15
        self.bullet_color = (60, 60, 60)
        self.bullets_allowed = 3

        # 外星人设置
        self.fleet_drop_speed = 10

        # 以什么样的速度加快游戏节奏
        self.speedup_scale = 1.1

        self.initialize_dynamic_settings()

    def initialize_dynamic_settings(self):
        """初始化随游戏进行而变化的设置"""
        self.ship_speed = 1.5
        self.bullet_speed = 3.0
        self.alien_speed = 1.0

        # fleet_direction 为 1 表示向右，为 -1 表示向左
        self.fleet_direction = 1

    def increase_speed(self):
        """提高速度设置"""
        self.ship_speed *= self.speedup_scale
        self.bullet_speed *= self.speedup_scale
        self.alien_speed *= self.speedup_scale
```

<aside>
⚡

`speedup_scale` 控制游戏加速的幅度。值为 **1.1** 表示每过一关，速度提高 10%。可以调整此值来控制难度增长的快慢。

</aside>

### 重置速度

每次开始新游戏时，需要将速度重置为初始值：

```python
def _check_play_button(self, mouse_pos):
    button_clicked = self.play_button.rect.collidepoint(mouse_pos)
    if button_clicked and not self.stats.game_active:
        # 重置游戏设置
        self.settings.initialize_dynamic_settings()
        --snip--
```

在消灭完一波外星人后调用 `increase_speed()`：

```python
def _check_bullet_alien_collisions(self):
    --snip--
    if not self.aliens:
        # 删掉现有的子弹并提高速度，再创建一群新的外星人
        self.bullets.empty()
        self._create_fleet()
        self.settings.increase_speed()
```

---

## 14.3 记分

实现一个 **记分系统**，实时跟踪玩家的得分，并显示最高得分、等级和剩余飞船数。

### 显示得分

创建 `Scoreboard` 类（`scoreboard.py`）来显示当前得分：

```python
import pygame.font

class Scoreboard:
    """显示得分信息的类"""
    def __init__(self, ai_game):
        """初始化显示得分涉及的属性"""
        self.screen = ai_game.screen
        self.screen_rect = self.screen.get_rect()
        self.settings = ai_game.settings
        self.stats = ai_game.stats

        # 显示得分信息时使用的字体设置
        self.text_color = (30, 30, 30)
        self.font = pygame.font.SysFont(None, 48)

        # 准备初始得分图像
        self.prep_score()

    def prep_score(self):
        """将得分转换为一幅渲染的图像"""
        score_str = str(self.stats.score)
        self.score_image = self.font.render(score_str, True,
                self.text_color, self.settings.bg_color)

        # 在屏幕右上角显示得分
        self.score_rect = self.score_image.get_rect()
        self.score_rect.right = self.screen_rect.right - 20
        self.score_rect.top = 20

    def show_score(self):
        """在屏幕上显示得分"""
        self.screen.blit(self.score_image, self.score_rect)
```

### 更新得分

在 `Settings` 中添加外星人点数：

```python
self.alien_points = 50
```

每当击落一个外星人时，更新得分：

```python
def _check_bullet_alien_collisions(self):
    collisions = pygame.sprite.groupcollide(
            self.bullets, self.aliens, True, True)

    if collisions:
        for aliens in collisions.values():
            self.stats.score += self.settings.alien_points * len(aliens)
        self.sb.prep_score()
```

<aside>
📌

使用 `len(aliens)` 确保 **一颗子弹同时击中多个外星人** 时，每个外星人都会计分。`collisions` 字典的每个值都是一个列表，包含被该子弹击中的所有外星人。

</aside>

### 提高点数

随着等级的提高，外星人的分值也应增加：

```python
class Settings:
    def __init__(self):
        --snip--
        self.score_scale = 1.5

    def initialize_dynamic_settings(self):
        --snip--
        self.alien_points = 50

    def increase_speed(self):
        --snip--
        self.alien_points = int(self.alien_points * self.score_scale)
```

### 将得分圆整

为了让得分更整洁，将其圆整为 10 的整数倍：

```python
def prep_score(self):
    """将得分转换为一幅渲染的图像"""
    rounded_score = round(self.stats.score, -1)
    score_str = "{:,}".format(rounded_score)
    self.score_image = self.font.render(score_str, True,
            self.text_color, self.settings.bg_color)
    --snip--
```

> `round(score, -1)` 将得分圆整到最近的 10 的整数倍。`"{:,}".format()` 在数字中插入千位分隔符（如 `1,000,000`）。
> 

### 最高得分

在 `GameStats` 中跟踪最高得分（最高得分 **不会** 在游戏重置时被清零）：

```python
class GameStats:
    def __init__(self, ai_game):
        --snip--
        # 最高得分不应被重置
        self.high_score = 0
```

在 `Scoreboard` 中添加最高得分的渲染和显示：

```python
def prep_high_score(self):
    """将最高得分转换为渲染的图像"""
    high_score = round(self.stats.high_score, -1)
    high_score_str = "{:,}".format(high_score)
    self.high_score_image = self.font.render(high_score_str, True,
            self.text_color, self.settings.bg_color)

    # 将最高得分放在屏幕顶部中央
    self.high_score_rect = self.high_score_image.get_rect()
    self.high_score_rect.centerx = self.screen_rect.centerx
    self.high_score_rect.top = self.score_rect.top
```

检查是否产生了新的最高得分：

```python
def check_high_score(self):
    """检查是否诞生了新的最高得分"""
    if self.stats.score > self.stats.high_score:
        self.stats.high_score = self.stats.score
        self.prep_high_score()
```

### 显示等级

在 `GameStats` 中添加 `level` 属性：

```python
def reset_stats(self):
    --snip--
    self.level = 1
```

在 `Scoreboard` 中渲染并显示等级：

```python
def prep_level(self):
    """将等级转换为渲染的图像"""
    level_str = str(self.stats.level)
    self.level_image = self.font.render(level_str, True,
            self.text_color, self.settings.bg_color)

    # 将等级放在得分下方
    self.level_rect = self.level_image.get_rect()
    self.level_rect.right = self.score_rect.right
    self.level_rect.top = self.score_rect.bottom + 10
```

消灭完一波外星人后，等级加 1：

```python
if not self.aliens:
    self.bullets.empty()
    self._create_fleet()
    self.settings.increase_speed()

    # 提高等级
    self.stats.level += 1
    self.sb.prep_level()
```

### 显示余下的飞船数

使用飞船图像在屏幕左上角显示玩家还剩多少条命：

```python
import pygame
from pygame.sprite import Sprite

class Ship(Sprite):
    """管理飞船的类"""
    def __init__(self, ai_game):
        super().__init__()
        --snip--
```

在 `Scoreboard` 中创建飞船编组来显示剩余生命数：

```python
from ship import Ship

def prep_ships(self):
    """显示还余下多少艘飞船"""
    self.ships = Group()
    for ship_number in range(self.stats.ships_left):
        ship = Ship(self.ai_game)
        ship.rect.x = 10 + ship_number * ship.rect.width
        ship.rect.y = 10
        self.ships.add(ship)
```

<aside>
🎮

让 `Ship` 继承 `Sprite` 类，以便将多个飞船图标加入 `Group` 中，统一绘制到屏幕上。

</aside>

---

## 14.4 文件结构总览

本项目最终涉及的 **主要文件和类**：

| **文件** | **类 / 功能** | **说明** |
| --- | --- | --- |
| `alien_invasion.py` | `AlienInvasion` | 主程序，管理游戏资源和整体行为 |
| `settings.py` | `Settings` | 存储静态和动态游戏设置 |
| `game_stats.py` | `GameStats` | 跟踪游戏统计信息（得分、等级、剩余飞船等） |
| `scoreboard.py` | `Scoreboard` | 渲染并显示得分、最高得分、等级、飞船数 |
| `button.py` | `Button` | 创建带文本标签的可点击按钮 |
| `ship.py` | `Ship` | 管理飞船行为（继承 Sprite） |
| `alien.py` | `Alien` | 管理单个外星人 |
| `bullet.py` | `Bullet` | 管理子弹 |

---

## 本章小结

| **概念** | **说明** |
| --- | --- |
| Play 按钮 | 使用 `pygame.font` 渲染文本，检测点击事件启动 / 重启游戏 |
| 隐藏光标 | `pygame.mouse.set_visible(False/True)` 控制光标显示 |
| 提高等级 | 区分静态 / 动态设置，`speedup_scale` 控制加速幅度 |
| 记分系统 | `Scoreboard` 类渲染并显示得分信息 |
| 确保全部计分 | 遍历 `collisions.values()` 避免遗漏同时击中的外星人 |
| 提高点数 | `score_scale` 控制分值随等级增长的倍率 |
| 圆整得分 | `round(score, -1)`  • `"{:,}".format()` 千位分隔 |
| 最高得分 | 在 `GameStats` 中独立存储，不随游戏重置而清零 |
| 显示等级 | 每消灭一波外星人，等级 +1 并更新渲染 |
| 显示飞船数 | 让 `Ship` 继承 `Sprite`，用编组在左上角绘制剩余飞船 |