---
title: 第2章 输入与输出
date: 2026-02-17 15:12:01
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 2.1 输入/输出流

### 概述

- Java 中的 **流（Stream）** 是对有序数据序列的抽象，分为 **输入流** 和 **输出流**
- 按处理数据单位分类：
    - **字节流**：以字节为单位，基类为 `InputStream` / `OutputStream`
    - **字符流**：以字符为单位，基类为 `Reader` / `Writer`
- 按功能分类：
    - **节点流**：直接连接数据源（如 `FileInputStream`）
    - **处理流（包装流）**：包装已有流，提供增强功能（如 `BufferedInputStream`）

### 读写字节

- `InputStream` 核心方法：
    - `int read()` — 读取一个字节，返回 0~255，到达末尾返回 -1
    - `int read(byte[] b)` — 读取多个字节到数组
    - `int read(byte[] b, int off, int len)` — 读取指定长度字节
    - `void close()` — 关闭流并释放资源
- `OutputStream` 核心方法：
    - `void write(int b)` — 写入一个字节
    - `void write(byte[] b)` — 写入整个字节数组
    - `void flush()` — 刷新输出流
    - `void close()` — 关闭流

### 完整的流家族

<aside>
📌

Java 的输入/输出流体系非常庞大，但核心思想是 **装饰器模式**：通过层层包装来增强流的功能。

</aside>

- **字节流层次结构：**
    - `InputStream` → `FileInputStream`、`ByteArrayInputStream`、`PipedInputStream`、`FilterInputStream`
    - `OutputStream` → `FileOutputStream`、`ByteArrayOutputStream`、`PipedOutputStream`、`FilterOutputStream`
- **字符流层次结构：**
    - `Reader` → `InputStreamReader`（`FileReader`）、`BufferedReader`、`StringReader`
    - `Writer` → `OutputStreamWriter`（`FileWriter`）、`BufferedWriter`、`PrintWriter`
- **桥梁类**（字节流 ↔ 字符流）：
    - `InputStreamReader` — 将字节输入流转为字符输入流
    - `OutputStreamWriter` — 将字符输出流转为字节输出流

### 组合输入/输出流过滤器

- Java 使用 **装饰器模式** 来组合流，例如：

```java
// 创建一个带缓冲的文件输入流
FileInputStream fis = new FileInputStream("data.txt");
BufferedInputStream bis = new BufferedInputStream(fis);

// 创建一个可以读取基本类型的输入流
DataInputStream dis = new DataInputStream(bis);
double value = dis.readDouble();
```

- 常见的过滤流：
    - `BufferedInputStream` / `BufferedOutputStream` — 缓冲，提升性能
    - `DataInputStream` / `DataOutputStream` — 读写 Java 基本数据类型
    - `PushbackInputStream` — 支持"回退"已读取的字节

---

## 2.2 文本输入与输出

### 文本输出：PrintWriter

- `PrintWriter` 是最常用的文本输出类
- 支持 `print()`、`println()`、`printf()` 方法

```java
PrintWriter out = new PrintWriter("output.txt", "UTF-8");
out.println("Hello, World!");
out.printf("价格: %.2f%n", 19.99);
out.close();
```

### 文本输入：Scanner / BufferedReader

- **Scanner** — 简单易用，支持正则分隔

```java
Scanner in = new Scanner(new File("data.txt"), "UTF-8");
while (in.hasNextLine()) {
    String line = in.nextLine();
}
```

- **BufferedReader** — 性能更好，适合逐行读取

```java
BufferedReader reader = new BufferedReader(
    new InputStreamReader(new FileInputStream("data.txt"), "UTF-8"));
String line;
while ((line = reader.readLine()) != null) {
    // 处理每一行
}
```

### 字符编码

- Java 使用 `Charset` 类表示字符编码
- 常用编码：`UTF-8`、`UTF-16`、`ISO-8859-1`、`GBK`
- `StandardCharsets` 提供常量：`StandardCharsets.UTF_8` 等
- 建议始终 **显式指定编码**，避免依赖平台默认编码

---

## 2.3 读写二进制数据

### DataInput 和 DataOutput 接口

- `DataInputStream` / `DataOutputStream` 可读写所有 Java 基本类型

```java
// 写入
DataOutputStream dos = new DataOutputStream(
    new FileOutputStream("data.bin"));
dos.writeInt(42);
dos.writeDouble(3.14);
dos.writeUTF("Hello");
dos.close();

// 读取（必须以相同顺序读）
DataInputStream dis = new DataInputStream(
    new FileInputStream("data.bin"));
int n = dis.readInt();
double d = dis.readDouble();
String s = dis.readUTF();
```

### 随机访问文件：RandomAccessFile

- 支持同时读写，可以跳转到文件任意位置
- `seek(long pos)` — 将指针移到指定位置
- `getFilePointer()` — 获取当前位置

```java
RandomAccessFile raf = new RandomAccessFile("data.bin", "rw");
raf.seek(100);       // 跳转到第100字节
int value = raf.readInt();
raf.close();
```

---

## 2.4 ZIP 文档

- `ZipInputStream` / `ZipOutputStream` 用于读写 ZIP 压缩文件
- 每个 ZIP 文件中的条目由 `ZipEntry` 表示

```java
// 读取 ZIP
ZipInputStream zin = new ZipInputStream(new FileInputStream("test.zip"));
ZipEntry entry;
while ((entry = zin.getNextEntry()) != null) {
    System.out.println(entry.getName());
    // 读取当前条目内容...
    zin.closeEntry();
}
zin.close();

// 写入 ZIP
ZipOutputStream zout = new ZipOutputStream(new FileOutputStream("out.zip"));
zout.putNextEntry(new ZipEntry("file.txt"));
byte[] data = "Hello ZIP".getBytes();
zout.write(data);
zout.closeEntry();
zout.close();
```

---

## 2.5 对象输入/输出流与序列化

### 对象序列化

- **序列化**：将对象转为字节序列 → `ObjectOutputStream.writeObject()`
- **反序列化**：将字节序列恢复为对象 → `ObjectInputStream.readObject()`
- 类必须实现 `Serializable` 接口

```java
// 序列化
ObjectOutputStream oos = new ObjectOutputStream(
    new FileOutputStream("obj.dat"));
oos.writeObject(new Date());
oos.close();

// 反序列化
ObjectInputStream ois = new ObjectInputStream(
    new FileInputStream("obj.dat"));
Date date = (Date) ois.readObject();
ois.close();
```

### 关键机制

- `serialVersionUID` — 版本控制，建议显式声明
- `transient` 关键字 — 标记不需要序列化的字段
- 序列化会自动处理 **对象引用的共享和循环引用**

### 自定义序列化

- 实现 `readObject()` 和 `writeObject()` 私有方法
- 实现 `Externalizable` 接口 — 完全自定义序列化过程

<aside>
⚠️

序列化存在 **安全风险**，反序列化不受信任的数据可能导致任意代码执行。Java 9+ 引入了 **反序列化过滤器** 来缓解此问题。

</aside>

---

## 2.6 操作文件：Path 和 Files（NIO.2）

### Path

- `Path` 表示文件系统中的路径（Java 7+，替代 `File`）
- 创建：`Path p = Paths.get("/home/user/docs/file.txt");`
- 常用方法：
    - `getFileName()` — 文件名
    - `getParent()` — 父路径
    - `resolve(other)` — 拼接路径
    - `toAbsolutePath()` — 转为绝对路径

### Files 工具类

- **读写快捷方法（适合小文件）：**

```java
// 读取所有内容
byte[] bytes = Files.readAllBytes(path);
String content = Files.readString(path);          // Java 11+
List<String> lines = Files.readAllLines(path);

// 写入
Files.write(path, content.getBytes());
Files.writeString(path, "Hello", StandardCharsets.UTF_8);  // Java 11+
```

- **文件/目录操作：**
    - `Files.exists(path)` — 是否存在
    - `Files.createDirectory(path)` — 创建目录
    - `Files.createFile(path)` — 创建文件
    - `Files.copy(src, dest)` — 复制
    - `Files.move(src, dest)` — 移动 / 重命名
    - `Files.delete(path)` — 删除

### 遍历目录

```java
// 列出目录内容
try (Stream<Path> entries = Files.list(dirPath)) {
    entries.forEach(System.out::println);
}

// 递归遍历
try (Stream<Path> entries = Files.walk(dirPath)) {
    entries.filter(p -> p.toString().endsWith(".java"))
           .forEach(System.out::println);
}
```

---

## 2.7 内存映射文件

- 将文件映射到内存中，直接对内存操作，性能极高
- 使用 [`FileChannel.map](http://FileChannel.map)()` 方法创建 `MappedByteBuffer`

```java
FileChannel channel = FileChannel.open(path, StandardOpenOption.READ);
MappedByteBuffer buffer = channel.map(
    FileChannel.MapMode.READ_ONLY, 0, channel.size());
// 直接从 buffer 读取数据
```

- 映射模式：
    - `READ_ONLY` — 只读
    - `READ_WRITE` — 读写
    - `PRIVATE` — 写时复制（不影响原文件）

---

## 2.8 文件加锁

- `FileLock` 用于控制对文件的并发访问
- 通过 `FileChannel` 获取锁

```java
FileChannel channel = FileChannel.open(path, StandardOpenOption.WRITE);
FileLock lock = channel.lock();     // 阻塞式获取独占锁
// 或
FileLock lock = channel.tryLock();  // 非阻塞，获取不到返回 null

// 使用完毕释放
lock.release();
```

- `lock()` — 独占锁（阻塞）
- `tryLock()` — 尝试获取锁（非阻塞）
- `lock(pos, size, shared)` — 锁定文件部分区域，`shared=true` 为共享锁

---

## 2.9 正则表达式（附录）

- `Pattern` — 编译正则表达式
- `Matcher` — 对字符串执行匹配操作

```java
Pattern pattern = Pattern.compile("\\d{4}-\\d{2}-\\d{2}");
Matcher matcher = pattern.matcher("日期是 2026-02-13");
if (matcher.find()) {
    System.out.println(matcher.group()); // 2026-02-13
}
```

- 常用方法：
    - `matches()` — 整串匹配
    - `find()` — 查找下一个匹配
    - `group()` — 返回匹配到的内容
    - `replaceAll()` — 替换所有匹配

---

## 📝 本章小结

<aside>
✅

**核心要点回顾：**

- 字节流（`InputStream/OutputStream`）处理原始字节；字符流（`Reader/Writer`）处理文本
- 使用 **装饰器模式** 组合流以获得所需功能
- 优先使用 `NIO.2`（`Path` + `Files`）操作文件系统
- 序列化需注意 `serialVersionUID` 和安全风险
- 内存映射文件适合处理大文件
- 始终显式指定字符编码，推荐 `UTF-8`
- 使用 **try-with-resources** 确保流正确关闭
</aside>