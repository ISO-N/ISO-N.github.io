---
title: 第7章 国际化
date: 2026-02-17 15:12:04
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 7.1 Locale（区域设置）

- **Locale** 是表示语言和地区的对象，用于控制格式化行为
- 由 **语言代码**（ISO 639）和 **国家/地区代码**（ISO 3166）组成，可选包含变体
- 常见构造方式：

```java
Locale locale = new Locale("zh", "CN");       // 中文-中国
Locale locale2 = Locale.US;                    // 英语-美国
Locale locale3 = Locale.forLanguageTag("en-US"); // BCP 47 标签
```

- `Locale.getDefault()` 获取系统默认区域
- `Locale.getAvailableLocales()` 获取所有可用区域
- Locale **本身不执行格式化**，只是传递给格式化器使用的参数

---

## 7.2 数字格式化

- 使用 `NumberFormat` 对数字、货币、百分比进行本地化格式化

```java
NumberFormat nf = NumberFormat.getNumberInstance(locale);
NumberFormat cf = NumberFormat.getCurrencyInstance(locale);
NumberFormat pf = NumberFormat.getPercentInstance(locale);

String formatted = cf.format(1234.56);  // ￥1,234.56（中国区域）
Number parsed = nf.parse("1,234.56");   // 解析回数字
```

- `parse()` 方法可能抛出 `ParseException`
- 不同区域对小数点（`.` vs `,`）和分组符号的使用不同

---

## 7.3 货币

- `Currency` 类表示 ISO 4217 货币代码

```java
Currency currency = Currency.getInstance("CNY");
String symbol = currency.getSymbol(locale);       // ￥
int digits = currency.getDefaultFractionDigits();  // 2
```

- `NumberFormat.getCurrencyInstance()` 自动根据 Locale 选择货币符号

---

## 7.4 日期和时间格式化

- Java 8+ 推荐使用 `DateTimeFormatter`

```java
DateTimeFormatter formatter = DateTimeFormatter
    .ofLocalizedDateTime(FormatStyle.MEDIUM)
    .withLocale(locale);

String text = ZonedDateTime.now().format(formatter);
```

- `FormatStyle` 四个级别：`FULL`、`LONG`、`MEDIUM`、`SHORT`
- 也可用自定义模式：`DateTimeFormatter.ofPattern("yyyy年MM月dd日", locale)`

---

## 7.5 排序与规范化

- 不同语言有不同的排序规则（如德语 ä 排在 a 之后，瑞典语 ä 排在 z 之后）
- 使用 `Collator` 进行本地化排序

```java
Collator collator = Collator.getInstance(locale);
collator.setStrength(Collator.SECONDARY);  // 忽略大小写

List<String> words = Arrays.asList("abc", "Abc", "äbc");
words.sort(collator);
```

- **强度级别**：
    - `PRIMARY`：忽略大小写和重音（a = A = ä）
    - `SECONDARY`：区分重音，忽略大小写（a = A ≠ ä）
    - `TERTIARY`（默认）：区分大小写和重音（a ≠ A ≠ ä）
- `CollationKey` 可提高重复比较的性能：

```java
CollationKey key = collator.getCollationKey("abc");
```

---

## 7.6 消息格式化

- `MessageFormat` 用于生成带占位符的本地化消息

```java
String pattern = "在 {1,date,long}，{0} 完成了 {2,number} 个任务。";
String result = MessageFormat.format(pattern, "小明", new Date(), 42);
```

- 占位符格式：`{index, type, style}`
    - **type**：`number`、`date`、`time`、`choice`
    - **style**：`short`、`medium`、`long`、`full`、自定义模式
- **ChoiceFormat** 处理复数形式：

```java
String pattern = "{0,choice,0#没有文件|1#一个文件|1<{0}个文件}";
```

---

## 7.7 资源包（Resource Bundle）

- **资源包** 是国际化的核心机制，将本地化字符串与代码分离
- 两种实现方式：
    - **属性文件**（`.properties`）：适合纯文本键值对
    - **ResourceBundle 类**：适合非字符串资源

### 属性文件方式

文件命名规则：`baseName_语言_[国家.properties](http://国家.properties)`

```
# Messages_zh_CN.properties
greeting=你好
farewell=再见

# Messages_en_US.properties
greeting=Hello
farewell=Goodbye
```

```java
ResourceBundle bundle = ResourceBundle.getBundle("Messages", locale);
String greeting = bundle.getString("greeting");
```

### 查找顺序（Fallback 机制）

以 `Locale("zh", "CN")` 为例，查找顺序：

1. `Messages_zh_[CN.properties](http://CN.properties)`
2. `Messages_[zh.properties](http://zh.properties)`
3. `Messages_默认语言_[默认国家.properties](http://默认国家.properties)`
4. `Messages_[默认语言.properties](http://默认语言.properties)`
5. [`Messages.properties`](http://Messages.properties)

---

## 7.8 一些常见注意事项

<aside>
⚠️

**编码问题**：`.properties` 文件在 Java 9+ 默认使用 **UTF-8** 编码。Java 8 及之前使用 ISO 8859-1，非 ASCII 字符需用 `\uXXXX` 转义。

</aside>

- 不要硬编码字符串，始终使用资源包
- 不要假设日期/数字格式，始终使用格式化器
- 不要假设字符串拼接顺序（不同语言语法顺序不同），使用 `MessageFormat`
- 使用 `Normalizer` 处理 Unicode 规范化（NFC、NFD 等）

---

## 本章小结

| **核心类** | **用途** |
| --- | --- |
| `Locale` | 表示语言和地区 |
| `NumberFormat` | 本地化数字/货币/百分比格式化 |
| `DateTimeFormatter` | 本地化日期时间格式化 |
| `Collator` | 本地化字符串排序比较 |
| `MessageFormat` | 带占位符的消息格式化 |
| `ResourceBundle` | 管理本地化资源（字符串等） |