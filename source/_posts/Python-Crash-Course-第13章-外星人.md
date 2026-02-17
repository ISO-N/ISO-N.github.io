---
title: 第13章 外星人
date: 2026-02-17 15:04:55
categories:
- [编程技术, Python]
tags:
- Python-Crash-Course
---
## 13.1 回顾项目

在开始新功能之前，先回顾项目的整体结构和当前进展：

- 第 12 章完成了飞船的左右移动和射击功能
- 本章目标：**创建外星人群**、让它们移动、实现射杀外星人、处理游戏结束逻辑

<aside>
📁

项目当前文件结构：

- `alien_invasion.py`：主程序文件，包含 `AlienInvasion` 类
- `settings.py`：存储所有游戏设置
- `ship.py`：管理飞船的 `Ship` 类
- `bullet.py`：管理子弹的 `Bullet` 类
- `alien.py`：**本章新增**，管理外星人的 `Alien` 类
</aside>

---

## 13.2 创建第一个外星人

### 创建 Alien 类

新建文件 `alien.py`，编写 `Alien` 类，与 `Ship` 类类似：

```python
import pygame
from pygame.sprite import Sprite

class Alien(Sprite):
    """表示单个外星人的类"""

    def __init__(self, ai_game):
        """初始化外星人并设置其起始位置"""
        super().__init__()
        self.screen = ai_game.screen
        self.settings = ai_game.settings

        # 加载外星人图像并设置其 rect 属性
        self.image = pygame.image.load('images/alien.bmp')
        self.rect = self.image.get_rect()

        # 每个外星人最初都在屏幕左上角附近
        self.rect.x = self.rect.width
        self.rect.y = self.rect.height

        # 存储外星人的精确水平位置
        self.x = float(self.rect.x)
```

<aside>
💡

`Alien` 类继承自 `Sprite`，这样后续可以使用 Pygame 的**编组（Group）** 功能来统一管理所有外星人。

</aside>

### 创建 Alien 实例

在 `AlienInvasion` 的 `__init__()` 方法中创建外星人编组，并调用辅助方法：

```python
from alien import Alien

# 在 __init__() 中
self.aliens = pygame.sprite.Group()
self._create_fleet()
```

### 让外星人出现在屏幕上

在 `_update_screen()` 方法中绘制外星人：

```python
self.aliens.draw(self.screen)
```

<aside>
📌

对编组调用 `draw()` 时，Pygame 会自动在每个元素的 `rect` 指定的位置绘制其 `image`。

</aside>

---

## 13.3 创建外星人群

### 确定一行能容纳多少个外星人

需要计算可用的水平空间，并据此确定每行外星人的数量：

```python
available_space_x = self.settings.screen_width - (2 * alien_width)
number_aliens_x = available_space_x // (2 * alien_width)
```

- 屏幕两侧各留一个外星人宽度的**边距**
- 每个外星人之间留一个外星人宽度的**间距**

### 创建一行外星人

```python
def _create_fleet(self):
    """创建外星人群"""
    alien = Alien(self)
    alien_width = alien.rect.width

    available_space_x = self.settings.screen_width - (2 * alien_width)
    number_aliens_x = available_space_x // (2 * alien_width)

    for alien_number in range(number_aliens_x):
        self._create_alien(alien_number)

def _create_alien(self, alien_number):
    """创建一个外星人并将其放在当前行"""
    alien = Alien(self)
    alien_width = alien.rect.width
    alien.x = alien_width + 2 * alien_width * alien_number
    alien.rect.x = alien.x
    self.aliens.add(alien)
```

### 重构 `_create_fleet()`

将创建单个外星人的逻辑提取到 `_create_alien()` 方法中，使代码更清晰。

### 添加行

计算可用的垂直空间并创建多行外星人：

```python
def _create_fleet(self):
    """创建外星人群"""
    alien = Alien(self)
    alien_width, alien_height = alien.rect.size

    available_space_x = self.settings.screen_width - (2 * alien_width)
    number_aliens_x = available_space_x // (2 * alien_width)

    # 计算屏幕可容纳多少行外星人
    ship_height = self.ship.rect.height
    available_space_y = (self.settings.screen_height
                         - (3 * alien_height) - ship_height)
    number_rows = available_space_y // (2 * alien_height)

    # 创建外星人群
    for row_number in range(number_rows):
        for alien_number in range(number_aliens_x):
            self._create_alien(alien_number, row_number)

def _create_alien(self, alien_number, row_number):
    """创建一个外星人并将其加入当前行"""
    alien = Alien(self)
    alien_width, alien_height = alien.rect.size
    alien.x = alien_width + 2 * alien_width * alien_number
    alien.rect.x = alien.x
    alien.rect.y = alien_height + 2 * alien_height * row_number
    self.aliens.add(alien)
```

| **计算项** | **公式** | **说明** |
| --- | --- | --- |
| 水平可用空间 | `screen_width - 2 * alien_width` | 两侧各留一个外星人宽度的边距 |
| 每行外星人数 | `available_space_x // (2 * alien_width)` | 每个外星人占两倍宽度（自身 + 间距） |
| 垂直可用空间 | `screen_height - 3 * alien_height - ship_height` | 顶部留空、飞船区域和缓冲区 |
| 行数 | `available_space_y // (2 * alien_height)` | 每行占两倍高度（自身 + 间距） |

---

## 13.4 让外星人群移动

### 向右移动外星人

在 `Alien` 类中添加 `update()` 方法：

```python
def update(self):
    """向右或向左移动外星人"""
    self.x += self.settings.alien_speed * self.settings.fleet_direction
    self.rect.x = self.x
```

在 `settings.py` 中添加外星人相关设置：

```python
# 外星人设置
self.alien_speed = 1.0
self.fleet_drop_speed = 10
# fleet_direction: 1 表示向右，-1 表示向左
self.fleet_direction = 1
```

### 检测外星人是否到达屏幕边缘

在 `Alien` 类中添加边缘检测方法：

```python
def check_edges(self):
    """如果外星人位于屏幕边缘，就返回 True"""
    screen_rect = self.screen.get_rect()
    return (self.rect.right >= screen_rect.right) or (self.rect.left <= 0)
```

### 向下移动并改变方向

在 `AlienInvasion` 类中添加方法处理外星人群的整体移动逻辑：

```python
def _check_fleet_edges(self):
    """有外星人到达边缘时采取相应的措施"""
    for alien in self.aliens.sprites():
        if alien.check_edges():
            self._change_fleet_direction()
            break

def _change_fleet_direction(self):
    """将整群外星人下移，并改变它们的方向"""
    for alien in self.aliens.sprites():
        alien.rect.y += self.settings.fleet_drop_speed
    self.settings.fleet_direction *= -1
```

在主循环中调用更新：

```python
self._check_fleet_edges()
self.aliens.update()
```

<aside>
🎮

外星人群的移动模式：**水平移动 → 到达边缘 → 整体下移一步 → 反转水平方向**，这就是经典太空入侵者游戏的移动方式。

</aside>

---

## 13.5 射杀外星人

### 检测子弹与外星人的碰撞

使用 `pygame.sprite.groupcollide()` 检测两个编组之间的碰撞：

```python
def _update_bullets(self):
    """更新子弹的位置并删除消失的子弹"""
    self.bullets.update()

    # 删除消失的子弹
    for bullet in self.bullets.copy():
        if bullet.rect.bottom <= 0:
            self.bullets.remove(bullet)

    self._check_bullet_alien_collisions()

def _check_bullet_alien_collisions(self):
    """响应子弹和外星人碰撞"""
    # 检查是否有子弹击中了外星人
    # 如果是，就删除相应的子弹和外星人
    collisions = pygame.sprite.groupcollide(
        self.bullets, self.aliens, True, True)
```

<aside>
💡

`groupcollide()` 的两个布尔参数分别控制是否删除发生碰撞的**第一组元素（子弹）** 和**第二组元素（外星人）**。设置为 `True, True` 表示子弹和外星人都会被删除。

</aside>

### 生成新的外星人群

当所有外星人被消灭后，清空子弹并重新创建外星人群：

```python
def _check_bullet_alien_collisions(self):
    """响应子弹和外星人碰撞"""
    collisions = pygame.sprite.groupcollide(
        self.bullets, self.aliens, True, True)

    if not self.aliens:
        # 删除现有的子弹并新建一群外星人
        self.bullets.empty()
        self._create_fleet()
```

### 重构 `_update_bullets()`

将碰撞检测逻辑提取到独立方法 `_check_bullet_alien_collisions()` 中，保持代码整洁。

---

## 13.6 结束游戏

### 检测外星人与飞船碰撞

使用 `pygame.sprite.spritecollideany()` 检测外星人与飞船之间的碰撞：

```python
def _update_aliens(self):
    """检查是否有外星人位于屏幕边缘，并更新外星人群中所有外星人的位置"""
    self._check_fleet_edges()
    self.aliens.update()

    # 检测外星人和飞船之间的碰撞
    if pygame.sprite.spritecollideany(self.ship, self.aliens):
        self._ship_hit()
```

<aside>
💡

`spritecollideany()` 接受一个精灵和一个编组，检查编组中是否有成员与精灵发生碰撞。找到就返回该成员，没有就返回 `None`。

</aside>

### 响应外星人与飞船碰撞

创建 `GameStats` 类来跟踪游戏统计信息，新建 `game_stats.py`：

```python
class GameStats:
    """跟踪游戏的统计信息"""

    def __init__(self, ai_game):
        """初始化统计信息"""
        self.settings = ai_game.settings
        self.reset_stats()

    def reset_stats(self):
        """初始化在游戏运行期间可能变化的统计信息"""
        self.ships_left = self.settings.ship_limit
```

在 `settings.py` 中添加：

```python
# 飞船设置
self.ship_limit = 3
```

编写 `_ship_hit()` 方法处理飞船被撞击后的逻辑：

```python
import time

def _ship_hit(self):
    """响应飞船被外星人撞到"""
    if self.stats.ships_left > 0:
        # 将 ships_left 减 1
        self.stats.ships_left -= 1

        # 清空余下的外星人和子弹
        self.aliens.empty()
        self.bullets.empty()

        # 创建一群新的外星人，并将飞船放到屏幕底端的中央
        self._create_fleet()
        self.ship.center_ship()

        # 暂停
        time.sleep(0.5)
    else:
        self.game_active = False
```

### 检测外星人到达屏幕底端

```python
def _check_aliens_bottom(self):
    """检查是否有外星人到达了屏幕底端"""
    for alien in self.aliens.sprites():
        if alien.rect.bottom >= self.settings.screen_height:
            # 像飞船被撞到一样处理
            self._ship_hit()
            break
```

在 `_update_aliens()` 中调用此方法：

```python
def _update_aliens(self):
    self._check_fleet_edges()
    self.aliens.update()

    if pygame.sprite.spritecollideany(self.ship, self.aliens):
        self._ship_hit()

    # 检查是否有外星人到达了屏幕底端
    self._check_aliens_bottom()
```

### Game Over

使用 `game_active` 标志来控制游戏状态：

```python
# 在 __init__() 中
self.game_active = True

# 在主循环 run_game() 中
while True:
    self._check_events()
    if self.game_active:
        self.ship.update()
        self._update_bullets()
        self._update_aliens()
    self._update_screen()
```

<aside>
⚠️

当 `game_active` 为 `False` 时，游戏仍然运行（可以响应退出事件），但不再更新飞船、子弹和外星人的位置——**游戏画面会冻结**。

</aside>

---

## 本章小结

| **知识点** | **核心内容** |
| --- | --- |
| 创建外星人 | 继承 `Sprite` 类，使用 `Group` 编组管理 |
| 创建外星人群 | 根据屏幕尺寸计算行数和列数，嵌套循环生成 |
| 外星人移动 | 水平移动 → 到达边缘 → 下移 → 反转方向 |
| 碰撞检测 | `groupcollide()`（组对组）、`spritecollideany()`（精灵对组） |
| 游戏状态管理 | `GameStats` 类跟踪统计信息，`game_active` 标志控制游戏流程 |
| 重构技巧 | 将复杂逻辑拆分为独立的辅助方法，保持代码整洁可维护 |
- [x]  学会使用 `Sprite` 和 `Group` 来管理大量相似的游戏元素
- [x]  掌握根据屏幕尺寸**动态计算**元素排列方式
- [x]  理解 Pygame 中两种**碰撞检测**方法的使用场景
- [x]  学会使用**标志变量**（如 `game_active`）控制游戏状态
- [x]  体会**重构**在游戏开发中的重要性——随着项目增长，保持代码简洁有序