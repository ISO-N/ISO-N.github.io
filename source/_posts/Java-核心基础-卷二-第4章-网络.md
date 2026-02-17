---
title: 第4章 网络
date: 2026-02-17 15:12:02
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 4.1 连接到服务器

### Socket 套接字

- `Socket` 类用于建立客户端与服务器之间的 TCP 连接
- 构造方法：`Socket(String host, int port)` 创建连接到指定主机和端口的套接字
- 连接成功后可通过 `getInputStream()` 和 `getOutputStream()` 获取 I/O 流

```java
try (Socket socket = new Socket("time-a.nist.gov", 13);
     Scanner in = new Scanner(socket.getInputStream(), StandardCharsets.UTF_8)) {
    while (in.hasNextLine()) {
        System.out.println(in.nextLine());
    }
}
```

### 套接字超时

- 默认情况下，`Socket` 构造器会无限期阻塞直到连接建立或抛出异常
- 可以先构造无连接的 Socket，再用 `connect` 方法设置超时：

```java
Socket socket = new Socket();
socket.connect(new InetSocketAddress(host, port), timeout);
```

- `setSoTimeout(int ms)` 设置读取数据的超时时间，超时抛出 `SocketTimeoutException`

---

## 4.2 实现服务器

### ServerSocket

- `ServerSocket(int port)` 建立监听指定端口的服务器套接字
- `accept()` 方法阻塞等待客户端连接，返回一个 `Socket` 对象

```java
try (ServerSocket server = new ServerSocket(8189)) {
    try (Socket incoming = server.accept()) {
        // 处理客户端连接
        InputStream inStream = incoming.getInputStream();
        OutputStream outStream = incoming.getOutputStream();
    }
}
```

### 为多个客户端服务

- 使用多线程，每当 `accept()` 返回一个 Socket，就启动一个新线程处理该连接
- 更好的做法是使用**线程池**（`ExecutorService`）避免创建过多线程

```java
ExecutorService pool = Executors.newCachedThreadPool();
while (true) {
    Socket incoming = server.accept();
    pool.submit(() -> handleClient(incoming));
}
```

---

## 4.3 可中断套接字

- 传统的 Socket I/O 在阻塞时**无法被线程中断**
- 使用 `SocketChannel` 可实现可中断的套接字操作：

```java
SocketChannel channel = SocketChannel.open(
    new InetSocketAddress(host, port));
Scanner in = new Scanner(channel, StandardCharsets.UTF_8);
```

- 当线程被中断时，通道会关闭并抛出 `ClosedByInterruptException`

---

## 4.4 获取 Web 数据

### URL 和 URI

- **URI**（统一资源标识符）：纯粹的语法结构，用于标识资源
    - 格式：`scheme:schemeSpecificPart#fragment`
    - 例：[`http://example.com/path?query=1#section`](http://example.com/path?query=1#section)
- **URL**（统一资源定位符）：URI 的子集，能实际定位并获取资源
- `URI` 类：负责解析、组合 URI 字符串
- `URL` 类：可以打开资源的输入流

### 使用 URLConnection 获取信息

- `URLConnection` 提供比 `URL.openStream()` 更多的控制
- 基本步骤：

```java
URL url = new URL("https://example.com");
URLConnection connection = url.openConnection();
connection.setRequestProperty("Accept-Charset", "UTF-8");
connection.connect();

// 读取响应头
Map<String, List<String>> headers = connection.getHeaderFields();

// 读取内容
try (InputStream in = connection.getInputStream()) {
    // 处理数据
}
```

### HttpURLConnection

- `HttpURLConnection` 是 `URLConnection` 的子类，专用于 HTTP 协议

```java
HttpURLConnection httpConn = (HttpURLConnection) url.openConnection();
httpConn.setRequestMethod("GET"); // GET、POST、PUT、DELETE 等
int responseCode = httpConn.getResponseCode();
```

### 提交表单数据（POST）

- 向服务器发送 POST 请求：

```java
HttpURLConnection connection = (HttpURLConnection) url.openConnection();
connection.setDoOutput(true);
connection.setRequestMethod("POST");
try (var out = new PrintWriter(connection.getOutputStream())) {
    out.print("key1=" + URLEncoder.encode(val1, "UTF-8")
            + "&key2=" + URLEncoder.encode(val2, "UTF-8"));
}
```

- 使用 `URLEncoder.encode()` 对参数值进行编码

---

## 4.5 HTTP 客户端（Java 11+）

### HttpClient API

- Java 11 引入了现代化的 [`java.net](http://java.net).http` 包，替代传统 `HttpURLConnection`
- 核心类：`HttpClient`、`HttpRequest`、`HttpResponse`

```java
HttpClient client = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com"))
    .header("Accept", "application/json")
    .GET()
    .build();

HttpResponse<String> response =
    client.send(request, HttpResponse.BodyHandlers.ofString());

System.out.println(response.statusCode());
System.out.println(response.body());
```

### 异步请求

- `sendAsync()` 返回 `CompletableFuture`，实现非阻塞请求

```java
client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println);
```

---

## 4.6 发送 E-mail

<aside>
⚠️

Java 原生的 `javax.mail` 不在标准库中，需要添加 **Jakarta Mail** 依赖。

</aside>

- 通过 SMTP 协议发送邮件的基本步骤：
    1. 建立与 SMTP 服务器的连接会话
    2. 创建 `MimeMessage` 对象，设置发件人、收件人、主题、正文
    3. 调用 `Transport.send()` 发送

```java
Properties props = new Properties();
props.put("mail.smtp.host", "smtp.example.com");
props.put("mail.smtp.port", "587");
props.put("mail.smtp.auth", "true");
props.put("mail.smtp.starttls.enable", "true");

Session session = Session.getInstance(props, new Authenticator() {
    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication("user", "password");
    }
});

MimeMessage msg = new MimeMessage(session);
msg.setFrom(new InternetAddress("sender@example.com"));
msg.setRecipient(Message.RecipientType.TO,
    new InternetAddress("receiver@example.com"));
msg.setSubject("主题");
msg.setText("正文内容", "UTF-8");

Transport.send(msg);
```

---

## 本章重点总结

| **概念** | **核心类/方法** | **要点** |
| --- | --- | --- |
| TCP 客户端 | `Socket` | 连接服务器，获取 I/O 流通信 |
| TCP 服务器 | `ServerSocket.accept()` | 阻塞等待连接，配合线程池使用 |
| 可中断 I/O | `SocketChannel` | 线程中断时自动关闭通道 |
| URL 读取 | `URLConnection` | 可设置请求头、读取响应头 |
| 现代 HTTP | `HttpClient`（Java 11+） | 支持同步/异步、链式构建请求 |
| 发送邮件 | Jakarta Mail | 通过 SMTP 协议，需额外依赖 |