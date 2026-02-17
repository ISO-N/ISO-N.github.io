---
title: 第10章 安全
date: 2026-02-17 15:12:07
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 10.1 类加载器

- **类加载器**负责将类的字节码加载到 JVM 中，是 Java 安全模型的基础组件
- Java 中有三种主要的类加载器：
    - **引导类加载器（Bootstrap ClassLoader）**：加载 `java.base` 等核心模块，由 C++ 实现
    - **平台类加载器（Platform ClassLoader）**：加载 Java 平台模块（取代了旧版的扩展类加载器）
    - **应用类加载器（Application/System ClassLoader）**：加载应用程序类路径上的类
- **双亲委派模型**：类加载器在加载类时，先委托父加载器尝试加载，只有父加载器无法加载时才自己加载
    - 保证核心类库不被篡改（如 `java.lang.String` 只能由引导类加载器加载）

```java
// 获取类加载器
ClassLoader loader = MyClass.class.getClassLoader();
System.out.println(loader);                  // AppClassLoader
System.out.println(loader.getParent());      // PlatformClassLoader
System.out.println(loader.getParent().getParent()); // null (Bootstrap)
```

### 自定义类加载器

- 继承 `ClassLoader`，重写 `findClass` 方法
- 典型用途：从网络、数据库或加密文件中加载类

```java
public class MyClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] bytes = loadClassBytes(name); // 自定义字节码来源
        return defineClass(name, bytes, 0, bytes.length);
    }
}
```

---

## 10.2 安全管理器与权限

> ⚠️ 注意：安全管理器（SecurityManager）在 Java 17 中已被标记为 **deprecated**，并在后续版本中逐步移除。了解其原理仍有学习价值。
> 
- **安全管理器**用于控制代码可以执行的操作（文件访问、网络连接等）
- 通过 `System.setSecurityManager()` 设置
- 启动时可以使用 `-[Djava.security](http://Djava.security).manager` 参数启用

### 权限（Permission）

- Java 使用**权限类**来描述对资源的访问控制
- 常见权限类：

| 权限类 | 用途 |
| --- | --- |
| `FilePermission` | 文件读写 |
| `SocketPermission` | 网络连接 |
| `PropertyPermission` | 系统属性访问 |
| `RuntimePermission` | 运行时操作（如退出 JVM） |
| `AllPermission` | 所有权限 |

```java
// 权限示例
FilePermission fp = new FilePermission("/tmp/*", "read,write");
SocketPermission sp = new SocketPermission("example.com:80", "connect");
```

---

## 10.3 策略文件

- 策略文件定义了不同代码源（CodeSource）所拥有的权限
- 默认策略文件位于 `$JAVA_HOME/conf/security/java.policy`

```
// 示例策略文件
grant codeBase "file:/myapp/*" {
    permission java.io.FilePermission "/tmp/-", "read,write";
    permission java.net.SocketPermission "*.example.com", "connect";
};

grant {
    permission java.util.PropertyPermission "java.version", "read";
};
```

- `codeBase`：指定代码来源（URL 形式）
- 无 `codeBase` 的 `grant` 块表示对所有代码生效

---

## 10.4 数字签名

### 消息摘要（Message Digest）

- 消息摘要是数据的**固定长度指纹**，用于验证数据完整性
- 常用算法：`SHA-256`、`SHA-512`

```java
MessageDigest md = MessageDigest.getInstance("SHA-256");
md.update(data);
byte[] digest = md.digest();
```

### 数字签名原理

- **签名过程**：发送方用**私钥**对消息摘要进行加密，生成数字签名
- **验证过程**：接收方用**公钥**解密签名，与自己计算的摘要对比
- 保证了消息的**完整性**和**不可否认性**

```java
// 生成签名
Signature sig = Signature.getInstance("SHA256withRSA");
sig.initSign(privateKey);
sig.update(data);
byte[] signature = sig.sign();

// 验证签名
sig.initVerify(publicKey);
sig.update(data);
boolean valid = sig.verify(signature);
```

---

## 10.5 证书与认证

### 数字证书

- 数字证书将公钥与身份信息绑定，由**证书颁发机构（CA）**签发
- 使用 **X.509** 标准格式
- 证书链：根 CA → 中间 CA → 终端实体证书

### 密钥库（KeyStore）

- Java 使用 `KeyStore` 存储密钥和证书
- 默认类型为 `PKCS12`（旧版为 `JKS`）

```java
KeyStore ks = KeyStore.getInstance("PKCS12");
ks.load(new FileInputStream("keystore.p12"), password);
Certificate cert = ks.getCertificate("alias");
```

### keytool 工具

- `keytool` 是 JDK 自带的密钥和证书管理工具

```bash
# 生成密钥对
keytool -genkeypair -alias mykey -keyalg RSA -keystore mykeystore.p12

# 导出证书
keytool -exportcert -alias mykey -keystore mykeystore.p12 -file mycert.cer

# 导入证书
keytool -importcert -alias trustedCA -keystore truststore.p12 -file ca.cer

# 查看密钥库
keytool -list -keystore mykeystore.p12
```

---

## 10.6 加密

### 对称加密

- 同一密钥用于加密和解密
- 常用算法：**AES**

```java
// AES 加密
KeyGenerator keygen = KeyGenerator.getInstance("AES");
keygen.init(256);
SecretKey key = keygen.generateKey();

Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, key);
byte[] encrypted = cipher.doFinal(plainText);

// AES 解密
cipher.init(Cipher.DECRYPT_MODE, key, new IvParameterSpec(iv));
byte[] decrypted = cipher.doFinal(encrypted);
```

### 非对称加密（公钥加密）

- 使用一对密钥：**公钥加密，私钥解密**
- 常用算法：**RSA**
- 速度较慢，通常用于加密对称密钥或数字签名

```java
KeyPairGenerator kpg = KeyPairGenerator.getInstance("RSA");
kpg.initialize(2048);
KeyPair kp = kpg.generateKeyPair();

Cipher cipher = Cipher.getInstance("RSA");
cipher.init(Cipher.ENCRYPT_MODE, kp.getPublic());
byte[] encrypted = cipher.doFinal(data);
```

### 密码流（CipherStream）

- `CipherInputStream` / `CipherOutputStream` 实现透明加解密

```java
CipherOutputStream cos = new CipherOutputStream(
    new FileOutputStream("secret.dat"), cipher);
cos.write(data);
cos.close();
```

---

## 10.7 小结

<aside>
📌

**核心要点回顾**

- **类加载器**通过双亲委派模型保护核心类库安全
- **安全管理器**和**策略文件**控制代码的权限（已逐步弃用）
- **消息摘要**验证数据完整性，**数字签名**保证不可否认性
- **数字证书**绑定公钥与身份，由 CA 签发和管理
- **对称加密**（AES）速度快，适合大量数据；**非对称加密**（RSA）安全性高，适合密钥交换
- Java 通过 [`java.security`](http://java.security) 和 `javax.crypto` 包提供完整的安全 API
</aside>