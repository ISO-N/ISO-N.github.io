---
title: 第6章 日期和时间API
date: 2026-02-17 15:12:04
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 6.1 时间线

- Java 的 `Instant` 表示时间线上的某个点（纪元：1970-01-01T00:00:00Z）
- [`Instant.now](http://Instant.now)()` 获取当前时刻
- `Duration` 表示两个 `Instant` 之间的时间量

```java
Instant start = Instant.now();
// 执行某些操作...
Instant end = Instant.now();
Duration elapsed = Duration.between(start, end);
long millis = elapsed.toMillis(); // 转换为毫秒
```

- `Duration` 常用方法：
    - `toNanos()`、`toMillis()`、`toSeconds()`、`toMinutes()`、`toHours()`、`toDays()`
    - `plus()`、`minus()`、`multipliedBy()`、`dividedBy()`、`negated()`
    - `isZero()`、`isNegative()`

> **注意：** `Instant` 和 `Duration` 都是**不可变**的，所有修改操作都会返回新对象。
> 

---

## 6.2 本地日期 LocalDate

- `LocalDate` 表示不带时区的日期（如 2026-02-13）
- 不包含时间信息，适用于生日、节假日等场景

```java
LocalDate today = LocalDate.now();
LocalDate birthday = LocalDate.of(1999, 7, 15);
// 也可以用 Month 枚举
LocalDate birthday2 = LocalDate.of(1999, Month.JULY, 15);
```

- 常用方法：
    - `getYear()`、`getMonthValue()`、`getDayOfMonth()`
    - `getDayOfWeek()` → 返回 `DayOfWeek` 枚举
    - `getDayOfYear()`
    - `plusDays()`、`plusWeeks()`、`plusMonths()`、`plusYears()`
    - `minusDays()`、`minusWeeks()` 等
    - `withYear()`、`withMonth()`、`withDayOfMonth()` — 修改某个字段
    - `isBefore()`、`isAfter()`、`isEqual()`
    - `until()` — 计算两个日期之间的 `Period`

---

## 6.3 日期调整器 TemporalAdjusters

- 用于常见的日期计算，如"下一个星期一"、"本月最后一天"

```java
LocalDate nextMonday = today.with(TemporalAdjusters.nextOrSame(DayOfWeek.MONDAY));
LocalDate lastDay = today.with(TemporalAdjusters.lastDayOfMonth());
LocalDate firstMonday = today.with(TemporalAdjusters.firstInMonth(DayOfWeek.MONDAY));
```

- 常用调整器：
    - `nextOrSame(dayOfWeek)` / `previousOrSame(dayOfWeek)`
    - `next(dayOfWeek)` / `previous(dayOfWeek)`
    - `firstDayOfMonth()` / `lastDayOfMonth()`
    - `firstDayOfNextMonth()` / `firstDayOfYear()`
    - `firstInMonth(dayOfWeek)` / `lastInMonth(dayOfWeek)`
- 可通过 `TemporalAdjuster` 接口自定义调整器

---

## 6.4 本地时间 LocalTime

- `LocalTime` 表示一天中的时间（如 15:30:00），不含日期和时区

```java
LocalTime now = LocalTime.now();
LocalTime bedTime = LocalTime.of(23, 30);     // 23:30
LocalTime wakeUp = LocalTime.of(7, 0, 0);     // 07:00:00
```

- 常用方法：`getHour()`、`getMinute()`、`getSecond()`、`plusHours()`、`minusMinutes()` 等

### LocalDateTime

- `LocalDateTime` = `LocalDate` + `LocalTime`

```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime dt = LocalDateTime.of(2026, 2, 13, 15, 30);
LocalDate date = dt.toLocalDate();
LocalTime time = dt.toLocalTime();
```

---

## 6.5 时区时间 ZonedDateTime

- `ZonedDateTime` = `LocalDateTime` + 时区信息
- **时区 (`ZoneId`)** 是处理跨地区时间的关键

```java
ZoneId shanghai = ZoneId.of("Asia/Shanghai");
ZonedDateTime now = ZonedDateTime.now(shanghai);
ZonedDateTime meeting = ZonedDateTime.of(2026, 2, 13, 14, 0, 0, 0, shanghai);
```

- 时区转换：

```java
ZonedDateTime inNewYork = now.withZoneSameInstant(ZoneId.of("America/New_York"));
```

- `ZoneId.getAvailableZoneIds()` 获取所有可用时区
- **夏令时注意：** 时区转换时 API 会自动处理夏令时偏移

> **最佳实践：** 存储和传输时间使用 `Instant`（UTC），展示时转换为 `ZonedDateTime`。
> 

---

## 6.6 格式化与解析

- `DateTimeFormatter` 用于日期/时间与字符串的互相转换

### 预定义格式化器

```java
String formatted = ZonedDateTime.now().format(DateTimeFormatter.ISO_DATE_TIME);
```

### 自定义模式

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm");
String text = LocalDateTime.now().format(formatter);
// "2026年02月13日 05:52"

LocalDateTime parsed = LocalDateTime.parse("2026年02月13日 05:52", formatter);
```

### 本地化格式

```java
DateTimeFormatter f = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL)
        .withLocale(Locale.CHINESE);
// "2026年2月13日星期五 中国标准时间 上午5:52:21"
```

- 常用模式符号：

| **符号** | **含义** | **示例** |
| --- | --- | --- |
| `yyyy` | 年 | 2026 |
| `MM` | 月 | 02 |
| `dd` | 日 | 13 |
| `HH` | 时（24小时） | 14 |
| `mm` | 分 | 30 |
| `ss` | 秒 | 45 |
| `E` | 星期 | 周五 |
| `z` | 时区名 | CST |

---

## 6.7 与遗留代码的互操作

- Java 8+ 在旧类上添加了转换方法，方便新旧 API 互转

| **旧类 → 新类** | **新类 → 旧类** |
| --- | --- |
| `Date.toInstant()` | `Date.from(instant)` |
| `Timestamp.toInstant()` | `Timestamp.from(instant)` |
| `Timestamp.toLocalDateTime()` | `Timestamp.valueOf(localDateTime)` |
| `Calendar.toInstant()` | — |
| `GregorianCalendar.toZonedDateTime()` | `GregorianCalendar.from(zonedDateTime)` |

```java
// Date ↔ Instant
Instant instant = new Date().toInstant();
Date date = Date.from(instant);

// Calendar → ZonedDateTime
GregorianCalendar cal = new GregorianCalendar();
ZonedDateTime zdt = cal.toZonedDateTime();
```

---

## 6.8 关键类速查

<aside>
📌

**核心类一览（均位于 `java.time` 包）**

- `Instant` — 时间戳（UTC 时间线上的点）
- `Duration` — 基于时间的时间量（秒、纳秒）
- `Period` — 基于日期的时间量（年、月、日）
- `LocalDate` — 不带时间的日期
- `LocalTime` — 不带日期的时间
- `LocalDateTime` — 日期 + 时间，无时区
- `ZonedDateTime` — 日期 + 时间 + 时区
- `ZoneId` / `ZoneOffset` — 时区标识 / 偏移量
- `DateTimeFormatter` — 格式化与解析
- `TemporalAdjusters` — 日期调整工具类
</aside>