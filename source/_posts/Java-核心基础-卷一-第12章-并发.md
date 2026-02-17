---
title: 第12章 并发
date: 2026-02-17 03:20:33
categories:
- [编程技术, Java]
tags:
- Java-核心基础-卷一
---
## 12.1 什么是线程

- **多线程程序**可以同时执行多个任务，每个任务在一个**线程（thread）** 中运行
- 线程与进程的区别：每个**进程**拥有独立的变量集合，而**线程**共享同一进程的数据
- 线程比进程更"轻量级"，创建和销毁的开销更小

### 创建线程

```java
// 方式一：实现 Runnable 接口（推荐）
Runnable task = () -> {
    // 任务代码
    System.out.println("Running in: " + Thread.currentThread().getName());
};
Thread t = new Thread(task);
t.start(); // 启动线程

// 方式二：继承 Thread 类（不推荐，Java 只支持单继承）
class MyThread extends Thread {
    public void run() {
        // 任务代码
    }
}
```

<aside>
⚠️

不要直接调用 `run()` 方法！直接调用只会在当前线程执行，不会启动新线程。必须调用 `start()` 方法。

</aside>

---

## 12.2 线程状态

线程有以下 **6 种状态**：

| **状态** | **说明** |
| --- | --- |
| `NEW`（新建） | 线程被创建但尚未调用 `start()` |
| `RUNNABLE`（可运行） | 调用 `start()` 后，可能正在运行也可能等待 CPU 时间片 |
| `BLOCKED`（阻塞） | 等待获取一个**内部对象锁** |
| `WAITING`（等待） | 等待另一个线程的通知，如调用了 `Object.wait()` 或 `Thread.join()` |
| `TIMED_WAITING`（计时等待） | 带有超时的等待，如 `Thread.sleep(millis)`、`Object.wait(millis)` |
| `TERMINATED`（终止） | 线程执行完毕或因异常退出 |

### 状态转换关系

```
NEW → (start) → RUNNABLE ⇄ BLOCKED / WAITING / TIMED_WAITING → RUNNABLE → TERMINATED
```

- `RUNNABLE` → `BLOCKED`：试图获取被其他线程持有的锁
- `RUNNABLE` → `WAITING`：调用 `wait()`、`join()` 或 `LockSupport.park()`
- `RUNNABLE` → `TIMED_WAITING`：调用 `sleep()`、`wait(timeout)` 或 `join(timeout)`

---

## 12.3 线程属性

### 中断线程

- 通过 `interrupt()` 方法请求中断线程（这是一种**协作式**机制，不会强制终止线程）
- 线程应定期检查中断状态并做出响应

```java
// 检查中断状态
while (!Thread.currentThread().isInterrupted()) {
    // 执行工作
}

// 如果线程处于阻塞状态（sleep/wait/join），
// 调用 interrupt() 会抛出 InterruptedException
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // 线程被中断，进行清理或退出
    Thread.currentThread().interrupt(); // 重新设置中断标志
}
```

<aside>
🚨

**不要忽略 `InterruptedException`！** 两种正确处理方式：

- 在 `catch` 中调用 `Thread.currentThread().interrupt()` 重新设置中断状态
- 直接将异常向上传播（在方法签名中声明 `throws InterruptedException`）
</aside>

### 守护线程

```java
t.setDaemon(true); // 必须在 start() 之前设置
```

- 守护线程是为其他线程提供服务的线程（如垃圾回收器）
- 当只剩下守护线程时，JVM 会自动退出

### 线程优先级

- 线程优先级范围：`Thread.MIN_PRIORITY`（1）~ `Thread.MAX_PRIORITY`（10），默认 `NORM_PRIORITY`（5）
- **不要依赖优先级来保证执行顺序**，优先级高度依赖操作系统

### 未捕获异常处理器

```java
// 为线程设置异常处理器
t.setUncaughtExceptionHandler((thread, ex) -> {
    System.err.println(thread.getName() + " 发生异常: " + ex.getMessage());
});

// 设置全局默认处理器
Thread.setDefaultUncaughtExceptionHandler((thread, ex) -> {
    // 记录日志等
});
```

---

## 12.4 同步

- 多个线程共享数据时，可能产生**竞态条件（race condition）**，导致数据不一致
- 经典例子：多个线程同时对同一账户进行转账操作

### ReentrantLock（可重入锁）

```java
private Lock bankLock = new ReentrantLock();

public void transfer(int from, int to, double amount) {
    bankLock.lock();
    try {
        // 临界区：同一时刻只有一个线程能执行
        accounts[from] -= amount;
        accounts[to] += amount;
    } finally {
        bankLock.unlock(); // 确保锁一定被释放
    }
}
```

- 锁是**可重入**的：同一线程可以多次获取已持有的锁
- **必须在 `finally` 块中释放锁**，防止异常导致锁未释放

### 条件对象（Condition）

```java
private Condition sufficientFunds = bankLock.newCondition();

public void transfer(int from, int to, double amount) throws InterruptedException {
    bankLock.lock();
    try {
        while (accounts[from] < amount) {
            sufficientFunds.await();  // 释放锁并等待
        }
        // 执行转账
        accounts[from] -= amount;
        accounts[to] += amount;
        sufficientFunds.signalAll(); // 唤醒所有等待线程
    } finally {
        bankLock.unlock();
    }
}
```

- `await()`：当前线程等待并释放锁
- `signalAll()`：唤醒所有在该条件上等待的线程
- `signal()`：随机唤醒一个等待线程（有死锁风险，不推荐）

<aside>
💡

`await()` 和 `signalAll()` 必须在持有锁的情况下调用，否则抛出 `IllegalMonitorStateException`。

</aside>

### synchronized 关键字

- Java 每个对象都有一个**内部锁（intrinsic lock / monitor lock）**
- `synchronized` 是更简洁的同步方式：

```java
// 同步方法：自动获取 this 的内部锁
public synchronized void transfer(int from, int to, double amount)
        throws InterruptedException {
    while (accounts[from] < amount)
        wait();           // 等同于内部锁条件的 await()
    accounts[from] -= amount;
    accounts[to] += amount;
    notifyAll();          // 等同于 signalAll()
}

// 同步代码块：锁定指定对象
synchronized (obj) {
    // 临界区
}
```

### synchronized 与 Lock 对比

| **特性** | **synchronized** | **ReentrantLock** |
| --- | --- | --- |
| 使用便捷性 | 简洁，自动释放锁 | 需手动 lock/unlock |
| 条件变量 | 只有一个条件（`wait`/`notifyAll`） | 可创建多个 `Condition` |
| 尝试获取锁 | 不支持 | 支持 `tryLock()` |
| 可中断锁 | 不支持 | 支持 `lockInterruptibly()` |
| 公平性 | 不可设置 | 可选公平锁 `new ReentrantLock(true)` |

### volatile 关键字

```java
private volatile boolean done;

public boolean isDone() { return done; }
public void setDone() { done = true; }
```

- `volatile` 保证变量的**可见性**：一个线程的修改对其他线程立即可见
- **不提供原子性**：`volatile` 不能替代锁，不适用于 `count++` 等非原子操作

### 原子操作（java.util.concurrent.atomic）

```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet();  // 原子自增，线程安全
count.accumulateAndGet(5, Math::max); // 原子更新为 max(当前值, 5)

// 大量线程更新时，使用 LongAdder / LongAccumulator 性能更好
LongAdder adder = new LongAdder();
adder.increment();
long total = adder.sum();
```

### 死锁

- **死锁**：两个或多个线程互相等待对方释放锁，导致永久阻塞
- Java 没有内置的死锁检测和恢复机制

<aside>
🚨

**避免死锁的建议：**

- 尽量减少锁的持有时间
- 按固定顺序获取多个锁
- 使用 `tryLock()` 带超时机制
- 避免在持有锁时调用外部方法
</aside>

### 线程局部变量（ThreadLocal）

```java
// 每个线程拥有自己的 DateFormatter 实例，避免共享
public static final ThreadLocal<SimpleDateFormat> dateFormat =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

// 使用
String dateStamp = dateFormat.get().format(new Date());
```

---

## 12.5 线程安全的集合

### java.util.concurrent 中的集合

| **集合** | **说明** |
| --- | --- |
| `ConcurrentHashMap` | 线程安全的哈希映射，支持高并发读写 |
| `ConcurrentSkipListMap` | 线程安全的有序映射 |
| `ConcurrentSkipListSet` | 线程安全的有序集合 |
| `ConcurrentLinkedQueue` | 线程安全的无界队列 |
| `CopyOnWriteArrayList` | 写时复制的列表，适合读多写少场景 |
| `CopyOnWriteArraySet` | 写时复制的集合 |

### ConcurrentHashMap 原子操作

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();

// 原子更新
map.merge("word", 1L, Long::sum);      // 如果存在则累加，不存在则设为 1
map.compute("word", (k, v) -> v == null ? 1L : v + 1); // 类似效果
map.putIfAbsent("word", 0L);           // 不存在时才放入

// 批量操作（不会锁定整个 map）
map.forEach(1000, (k, v) -> System.out.println(k + " -> " + v));
long sum = map.reduceValues(1000, Long::sum);
```

### 阻塞队列（BlockingQueue）

- **生产者-消费者模式**的核心组件
- 当队列满时，`put()` 会阻塞；当队列空时，`take()` 会阻塞

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>(100);

// 生产者线程
queue.put("item");   // 队列满时阻塞

// 消费者线程
String item = queue.take();  // 队列空时阻塞
```

常用实现：

- `LinkedBlockingQueue`：基于链表，可选容量限制
- `ArrayBlockingQueue`：基于数组，固定容量
- `PriorityBlockingQueue`：带优先级的阻塞队列
- `DelayQueue`：延迟队列，元素只有到期后才能取出

---

## 12.6 任务和线程池

- 直接创建线程开销大，**线程池**可以复用线程来执行多个任务

### Executor 框架

```java
// 创建线程池
ExecutorService executor = Executors.newFixedThreadPool(4);      // 固定大小
ExecutorService executor2 = Executors.newCachedThreadPool();      // 按需创建
ExecutorService executor3 = Executors.newSingleThreadExecutor();  // 单线程

// 提交任务
executor.execute(task);                    // 提交 Runnable，无返回值
Future<String> future = executor.submit(() -> {
    // 有返回值的任务
    return "result";
});

// 关闭线程池
executor.shutdown();     // 不再接受新任务，等待已有任务完成
executor.shutdownNow();  // 尝试取消所有正在执行的任务
```

### 线程池类型

| **工厂方法** | **说明** |
| --- | --- |
| `newFixedThreadPool(n)` | 固定 n 个线程，适合负载较重的场景 |
| `newCachedThreadPool()` | 按需创建线程，空闲 60s 后回收，适合大量短任务 |
| `newSingleThreadExecutor()` | 单线程顺序执行任务 |
| `newScheduledThreadPool(n)` | 支持定时和周期性任务 |

### Fork/Join 框架

- 适用于可以递归拆分的**计算密集型**任务
- 采用**工作窃取（work-stealing）** 算法：空闲线程从其他线程的队列中"窃取"任务

```java
class CountTask extends RecursiveTask<Long> {
    private final int[] array;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // 基本情况：直接计算
            long sum = 0;
            for (int i = start; i < end; i++) sum += array[i];
            return sum;
        } else {
            // 递归拆分
            int mid = (start + end) / 2;
            CountTask left = new CountTask(array, start, mid);
            CountTask right = new CountTask(array, mid, end);
            left.fork();           // 异步执行左半部分
            Long rightResult = right.compute(); // 当前线程计算右半部分
            Long leftResult = left.join();      // 等待左半部分完成
            return leftResult + rightResult;
        }
    }
}

// 使用
ForkJoinPool pool = new ForkJoinPool();
Long result = pool.invoke(new CountTask(array, 0, array.length));
```

---

## 12.7 异步计算

### Future

```java
Future<String> future = executor.submit(() -> {
    Thread.sleep(1000);
    return "done";
});

// 获取结果（阻塞）
String result = future.get();                    // 无限等待
String result2 = future.get(5, TimeUnit.SECONDS); // 超时等待

future.cancel(true);      // 取消任务
future.isDone();           // 是否完成
future.isCancelled();      // 是否被取消
```

### CompletableFuture

- `CompletableFuture` 是 Java 8 引入的强大异步编程工具，支持**链式调用**和**组合操作**

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchData())         // 异步执行
    .thenApply(data -> process(data))       // 对结果进行转换
    .thenApply(result -> format(result));   // 继续转换

// 组合多个 CompletableFuture
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");
future1.thenCombine(future2, (s1, s2) -> s1 + " " + s2); // "Hello World"

// 等待任意一个完成
CompletableFuture.anyOf(future1, future2).thenAccept(System.out::println);

// 等待所有完成
CompletableFuture.allOf(future1, future2).thenRun(() -> {
    System.out.println("All done!");
});
```

### CompletableFuture 常用方法

| **方法** | **说明** |
| --- | --- |
| `supplyAsync(supplier)` | 异步执行有返回值的任务 |
| `runAsync(runnable)` | 异步执行无返回值的任务 |
| `thenApply(fn)` | 对结果进行转换（同步） |
| `thenApplyAsync(fn)` | 对结果进行转换（异步） |
| `thenCompose(fn)` | 串联另一个异步操作（flatMap） |
| `thenCombine(other, fn)` | 组合两个结果 |
| `thenAccept(consumer)` | 消费结果，无返回值 |
| `exceptionally(fn)` | 异常处理，提供备用值 |
| `handle(fn)` | 处理结果或异常 |

### 异常处理

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        if (error) throw new RuntimeException("oops");
        return "ok";
    })
    .exceptionally(ex -> "fallback: " + ex.getMessage())
    .handle((result, ex) -> {
        if (ex != null) return "error";
        return result;
    });
```

---

## 12.8 进程

- Java 9 增强了对操作系统进程的控制能力
- 使用 `ProcessBuilder` 创建和管理外部进程

```java
// 启动外部进程
ProcessBuilder builder = new ProcessBuilder("ls", "-l");
builder.directory(new File("/tmp"));       // 设置工作目录
builder.redirectErrorStream(true);          // 合并标准错误到标准输出
Process process = builder.start();

// 读取进程输出
try (Scanner scanner = new Scanner(process.getInputStream())) {
    while (scanner.hasNextLine()) {
        System.out.println(scanner.nextLine());
    }
}

int exitCode = process.waitFor();  // 等待进程结束

// Java 9+ ProcessHandle：获取进程信息
ProcessHandle.current().pid();                  // 当前进程 PID
ProcessHandle.current().info().command();        // 进程命令
ProcessHandle.allProcesses()                    // 所有进程
    .filter(ph -> ph.info().command().isPresent())
    .forEach(ph -> System.out.println(ph.pid()));
```

---

## 本章小结

<aside>
✅

**核心要点回顾：**

- 通过实现 `Runnable` 接口并调用 `start()` 创建线程
- 线程有 6 种状态：NEW → RUNNABLE ⇄ BLOCKED/WAITING/TIMED_WAITING → TERMINATED
- **同步机制**：`ReentrantLock` + `Condition` 提供细粒度控制，`synchronized` 更简洁
- `volatile` 保证可见性但不保证原子性，原子操作用 `Atomic*` 类
- 优先使用 `java.util.concurrent` 中的**线程安全集合**而非手动同步
- **线程池**（`ExecutorService`）复用线程，避免频繁创建销毁的开销
- **Fork/Join** 框架适合可递归拆分的计算密集型任务
- **CompletableFuture** 是异步编程的首选工具，支持链式调用和组合
- 注意死锁问题：按固定顺序获取锁、减少锁持有时间、使用 `tryLock()`
</aside>