---
title: 第12章 本地方法
date: 2026-02-17 15:12:08
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷二
---
## 12.1 从 Java 程序中调用 C 函数

- Java 使用 **JNI（Java Native Interface）** 与本地代码（C/C++）交互
- 声明本地方法使用 `native` 关键字：

```java
public class HelloNative {
    public static native void greeting();
}
```

- 调用流程：
    1. 编写 Java 类，声明 `native` 方法
    2. 使用 `javac -h` 生成 C 头文件（`.h`）
    3. 用 C/C++ 实现该头文件中声明的函数
    4. 编译为共享库（`.dll` / `.so`）
    5. Java 中通过 `System.loadLibrary("库名")` 加载

```java
static {
    System.loadLibrary("HelloNative");
}
```

## 12.2 数值参数与返回值

- JNI 定义了一组与 Java 类型对应的 C 类型：

| **Java 类型** | **JNI C 类型** | **字节数** |
| --- | --- | --- |
| boolean | jboolean | 1 |
| byte | jbyte | 1 |
| char | jchar | 2 |
| short | jshort | 2 |
| int | jint | 4 |
| long | jlong | 8 |
| float | jfloat | 4 |
| double | jdouble | 8 |
- 本地方法的 C 函数签名规则：`Java_完整类名_方法名`
- 每个本地方法至少有两个参数：
    - `JNIEnv *env`：指向 JNI 函数表的指针
    - `jobject thisObj`（实例方法）或 `jclass cls`（静态方法）

## 12.3 字符串参数

- Java 字符串 → C 字符串需要转换，**不能直接当 C 字符串使用**
- 常用函数：
    - `GetStringUTFChars(env, jstr, NULL)` — 获取 UTF-8 编码的 C 字符串
    - `ReleaseStringUTFChars(env, jstr, cstr)` — 释放字符串，**必须调用，防止内存泄漏**
    - `NewStringUTF(env, cstr)` — 从 C 字符串创建 Java 字符串

```c
JNIEXPORT jstring JNICALL Java_HelloNative_getGreeting(JNIEnv *env, jclass cls) {
    return (*env)->NewStringUTF(env, "Hello, Native World!");
}
```

## 12.4 访问域

- 本地代码可以**读写 Java 对象的字段**
- 步骤：
    1. 获取类引用：`GetObjectClass(env, obj)`
    2. 获取字段 ID：`GetFieldID(env, cls, "fieldName", "签名")`
    3. 读取/设置字段值：`Get<Type>Field` / `Set<Type>Field`
- **字段签名表示法**：

| **类型** | **签名** |
| --- | --- |
| int | `I` |
| long | `J` |
| double | `D` |
| String | `Ljava/lang/String;` |
| int[] | `\[I` |

## 12.5 编码签名

- 用 `javap -s 类名` 可查看方法和字段的签名
- 方法签名格式：`(参数签名)返回值签名`
- 示例：
    - `void f(int, String)` → `(ILjava/lang/String;)V`
    - `int[] sort(double[])` → `([D)[I`

## 12.6 调用 Java 方法

- 本地代码可以**回调 Java 方法**
- 步骤：
    1. 获取类引用
    2. 获取方法 ID：`GetMethodID(env, cls, "方法名", "签名")`
    3. 调用方法：`Call<Type>Method(env, obj, methodID, args...)`
- 调用静态方法使用 `CallStatic<Type>Method`
- 调用父类方法使用 `CallNonvirtual<Type>Method`

## 12.7 访问数组元素

- 获取数组长度：`GetArrayLength(env, arr)`
- 获取数组元素指针：`Get<Type>ArrayElements(env, arr, NULL)`
- 释放数组：`Release<Type>ArrayElements(env, arr, elems, 0)`
- 创建新数组：`New<Type>Array(env, length)`

<aside>
⚠️

操作数组元素后**必须调用 Release 函数**，否则可能导致内存泄漏或数组修改不同步。

</aside>

## 12.8 错误处理

- 本地方法中可以**抛出 Java 异常**：

```c
jclass excCls = (*env)->FindClass(env, "java/lang/IllegalArgumentException");
(*env)->ThrowNew(env, excCls, "参数错误");
```

- 检测是否有挂起的异常：`ExceptionCheck(env)` 或 `ExceptionOccurred(env)`
- 清除异常：`ExceptionClear(env)`
- 本地方法中发生异常后，应尽快 `return`，由 Java 端处理

## 12.9 使用调用 API

- **调用 API** 允许从 C/C++ 程序中**嵌入 JVM**
- 核心步骤：
    1. 使用 `JNI_CreateJavaVM` 创建虚拟机
    2. 通过 JNI 环境指针调用 Java 方法
    3. 使用 `DestroyJavaVM` 销毁虚拟机

```c
JavaVM *jvm;
JNIEnv *env;
JavaVMInitArgs vm_args;
JNI_CreateJavaVM(&jvm, (void **)&env, &vm_args);
// ... 调用 Java 方法 ...
(*jvm)->DestroyJavaVM(jvm);
```

- 可通过 `JavaVMOption` 设置类路径等参数

## 12.10 完整的编译流程示例

1. 编写 Java 类并声明 `native` 方法
2. `javac -h . [HelloNative.java](http://HelloNative.java)` → 生成 `.h` 头文件
3. 编写 C 实现文件
4. 编译为共享库：
    - **Linux**：`gcc -shared -o [libHelloNative.so](http://libHelloNative.so) -I$JAVA_HOME/include ...`
    - **Windows**：生成 `.dll` 文件
5. 运行：`java -Djava.library.path=. HelloNative`

---

<aside>
📝

**本章要点总结**

- `native` 关键字声明本地方法，通过 JNI 与 C/C++ 交互
- JNI 提供了完整的类型映射、字符串转换、字段/方法访问、数组操作和异常处理机制
- 操作字符串和数组后**必须释放资源**
- 调用 API 可以在 C 程序中嵌入 JVM
- 本地方法破坏了 Java 的跨平台性和安全性，**仅在必要时使用**
</aside>