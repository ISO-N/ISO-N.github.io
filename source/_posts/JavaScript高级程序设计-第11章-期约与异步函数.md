---
title: 第11章 期约与异步函数
date: 2026-02-17 15:02:04
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 11.1 异步编程

### 11.1.1 同步与异步

**同步行为**：代码按顺序逐条执行，每条语句都能立即获得结果。

**异步行为**：类似于系统中断——当前进程外部的实体可以触发代码执行。异步操作不会阻塞主线程，而是在操作完成后通过回调等机制通知结果。

### 11.1.2 以往的异步编程模式

在早期 JavaScript 中，异步操作主要通过**回调函数**来实现：

```jsx
function double(value, callback) {
  setTimeout(() => callback(value * 2), 1000);
}

double(3, (result) => console.log(result)); // 6（约1秒后）
```

**回调地狱（Callback Hell）**：当多个异步操作嵌套时，代码可读性急剧下降：

```jsx
getData(function (a) {
  getMoreData(a, function (b) {
    getEvenMoreData(b, function (c) {
      console.log(c);
    });
  });
});
```

这种模式的主要问题：

- 嵌套层级深，可读性差
- 错误处理困难，每一层都需要单独处理
- 代码耦合度高，难以复用

---

## 11.2 期约（Promise）

### 11.2.1 Promise/A+ 规范

ECMAScript 6 新增的 `Promise` 引用类型，实现了 **Promises/A+** 规范。Promise 是对尚未完成的异步操作的一个"占位符"——它代表一个将来才会知道结果的值。

### 11.2.2 期约基础

**创建期约**：通过 `new Promise()` 构造函数，传入一个**执行器（executor）函数**：

```jsx
let p = new Promise((resolve, reject) => {
  // 执行器函数体
});
```

**三种状态**：

| 状态 | 含义 | 说明 |
| --- | --- | --- |
| **pending**（待定） | 初始状态 | 尚未兑现也未拒绝 |
| **fulfilled**（兑现） | 操作成功 | 也称 resolved |
| **rejected**（拒绝） | 操作失败 | 包含拒绝的理由 |

<aside>
⚠️

期约的状态是**不可逆**的。一旦从 pending 变为 fulfilled 或 rejected，就永远不会再改变。

</aside>

**控制期约状态**：通过执行器函数的两个参数来控制：

```jsx
// 兑现
let p1 = new Promise((resolve, reject) => resolve("成功"));

// 拒绝
let p2 = new Promise((resolve, reject) => reject("失败"));
```

**`Promise.resolve()` 与 `Promise.reject()`**：静态方法，用于快速创建已解决的期约：

```jsx
// 等价于 new Promise((resolve) => resolve("value"))
let p1 = Promise.resolve("value");

// 等价于 new Promise((_, reject) => reject("reason"))
let p2 = Promise.reject("reason");
```

<aside>
💡

`Promise.resolve()` 是**幂等**的：如果传入的参数本身就是一个 Promise，则直接返回该 Promise；而 `Promise.reject()` 不是幂等的，即使传入 Promise 也会将其作为拒绝理由包装成新的 rejected Promise。

</aside>

**同步/异步执行的二元性**：

```jsx
try {
  throw new Error("同步错误");
} catch (e) {
  console.log(e); // 可以捕获
}

try {
  Promise.reject(new Error("异步错误"));
} catch (e) {
  // 捕获不到！因为拒绝是异步的
}
```

### 11.2.3 期约的实例方法

#### `Promise.prototype.then()`

`then()` 是为期约添加处理程序的主要方法，接收最多两个参数：`onFulfilled` 和 `onRejected`：

```jsx
let p = new Promise((resolve, reject) => resolve("foo"));

p.then(
  (value) => console.log("兑现:", value),   // 兑现: foo
  (reason) => console.log("拒绝:", reason)
);
```

<aside>
💡

`then()` 返回一个**新的期约实例**，这使得**链式调用**成为可能。返回值会被 `Promise.resolve()` 包装。

</aside>

#### `Promise.prototype.catch()`

`catch()` 是 `then(null, onRejected)` 的**语法糖**，专门用于处理拒绝：

```jsx
let p = Promise.reject("error");

p.catch((reason) => console.log(reason)); // "error"

// 等价于
p.then(null, (reason) => console.log(reason));
```

#### `Promise.prototype.finally()`

`finally()` 无论期约兑现还是拒绝都会执行，不接收任何参数，通常用于**清理操作**：

```jsx
let p = Promise.resolve("value");

p.finally(() => console.log("清理操作"))
 .then((value) => console.log(value));
// "清理操作"
// "value"
```

<aside>
⚠️

`finally()` 大多数情况下表现为父期约的**透传**——它不会修改期约的解决值或拒绝原因，除非它返回一个 pending 的期约或抛出错误。

</aside>

#### 非重入期约方法

当期约进入落定（settled）状态时，与该状态相关的处理程序仅仅会被**排期**（入微任务队列），而非立即执行：

```jsx
let p = Promise.resolve();

p.then(() => console.log("onResolved handler"));

console.log("then() returns");
// "then() returns"
// "onResolved handler"
```

#### 邻近处理程序的执行顺序

如果给期约添加了多个处理程序，会按照**添加顺序**依次执行：

```jsx
let p = Promise.resolve();

p.then(() => console.log("1"));
p.then(() => console.log("2"));
p.then(() => console.log("3"));
// 1, 2, 3
```

### 11.2.4 期约连锁与期约合成

#### 期约连锁（Promise Chaining）

每个 `then()`、`catch()`、`finally()` 都返回新的期约，因此可以串联成**期约链**：

```jsx
let p = new Promise((resolve, reject) => {
  console.log("first");
  resolve();
});

p.then(() => console.log("second"))
 .then(() => console.log("third"))
 .then(() => console.log("fourth"));
// first → second → third → fourth
```

**串行化异步任务**：让每个 then 返回一个新的 Promise：

```jsx
function delayedResolve(str) {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log(str);
      resolve();
    }, 1000);
  });
}

delayedResolve("第一步")
  .then(() => delayedResolve("第二步"))
  .then(() => delayedResolve("第三步"));
// 每隔1秒依次输出
```

#### 期约合成（Promise Composition）

**`Promise.all()`**——全部兑现才兑现，任一拒绝即拒绝：

```jsx
let p = Promise.all([
  Promise.resolve(1),
  Promise.resolve(2),
  Promise.resolve(3)
]);

p.then((values) => console.log(values)); // [1, 2, 3]
```

- 合成的期约在**所有**包含期约都兑现后才兑现，解决值是所有期约解决值的**数组**（按迭代器顺序）
- 如果有一个包含的期约拒绝，则合成的期约也拒绝，拒绝理由是**第一个**拒绝的期约的理由

**`Promise.race()`**——"赛跑"，采用第一个落定的期约的状态：

```jsx
let p = Promise.race([
  new Promise((resolve) => setTimeout(resolve, 3000, "slow")),
  new Promise((resolve) => setTimeout(resolve, 1000, "fast"))
]);

p.then((value) => console.log(value)); // "fast"
```

- 无论是兑现还是拒绝，最先落定的结果就是 `Promise.race()` 的结果
- 其余期约会**静默处理**，不影响最终结果

### 11.2.5 期约扩展

书中提到两个 ECMAScript 未实现但常见的期约扩展概念：

**期约取消（Promise Cancellation）**：原生 Promise 一旦创建就无法从外部取消。社区有 `CancelToken` 等取消令牌的方案：

```jsx
class CancelToken {
  constructor(cancelFn) {
    this.promise = new Promise((resolve) => {
      cancelFn(() => {
        this.canceled = true;
        resolve();
      });
    });
  }
}
```

**进度通知（Progress Notification）**：原生 Promise 不支持进度通知。可以通过扩展 Promise 类来实现：

```jsx
class TrackablePromise extends Promise {
  constructor(executor) {
    const notifyHandlers = [];
    super((resolve, reject) => {
      return executor(resolve, reject, (status) => {
        notifyHandlers.forEach((handler) => handler(status));
      });
    });
    this.notifyHandlers = notifyHandlers;
  }

  notify(notifyHandler) {
    this.notifyHandlers.push(notifyHandler);
    return this;
  }
}
```

---

## 11.3 异步函数（async/await）

异步函数（`async/await`）是 ES8（ES2017）引入的特性，是 Promise 的**语法糖**，让异步代码看起来像同步代码。

### 11.3.1 async 关键字

`async` 用于声明异步函数，可用于函数声明、函数表达式、箭头函数和方法：

```jsx
async function foo() {}
let bar = async function () {};
let baz = async () => {};

class MyClass {
  async method() {}
}
```

<aside>
💡

`async` 函数的返回值会被自动包装为 `Promise.resolve()`。即使返回的是原始值，也会被包装成期约对象。

</aside>

```jsx
async function foo() {
  return "hello";
}

foo().then(console.log); // "hello"
```

在 `async` 函数中抛出错误，会返回**拒绝的期约**：

```jsx
async function foo() {
  throw new Error("出错了");
}

foo().catch(console.log); // Error: 出错了
```

### 11.3.2 await 关键字

`await` 用于暂停异步函数的执行，等待期约解决后恢复执行：

```jsx
async function foo() {
  let result = await Promise.resolve("完成");
  console.log(result);
}

foo(); // "完成"
```

`await` 的行为：

- 如果 `await` 后面是一个**期约**，则暂停执行，等待其落定，然后取出解决值
- 如果 `await` 后面是一个**非期约值**，则将其当作已解决的期约值处理
- 如果 `await` 后面的期约**拒绝**，则会把拒绝理由抛出，可用 `try/catch` 捕获

```jsx
async function foo() {
  try {
    await Promise.reject("出错了");
  } catch (e) {
    console.log(e); // "出错了"
  }
}
```

<aside>
⚠️

`await` 只能在 `async` 函数内部使用。在顶层代码或普通函数中使用会抛出 `SyntaxError`。（注：现代 JS 模块支持顶层 `await`）

</aside>

### 11.3.3 await 的限制

1. **不能在顶层上下文中使用**（除 ES Module 外）
2. **异步函数的特质不会扩展到嵌套函数**——在 `async` 函数内定义的普通函数中不能使用 `await`

```jsx
async function foo() {
  // ❌ 错误！await在普通函数中
  const arr = [1, 2, 3];
  arr.forEach(function (item) {
    await doSomething(item); // SyntaxError
  });
}

async function bar() {
  // ✅ 正确！使用for...of
  const arr = [1, 2, 3];
  for (const item of arr) {
    await doSomething(item);
  }
}
```

### 11.3.4 停止和恢复执行

`await` 的本质是让出 JavaScript 运行时的执行线程。理解执行顺序非常重要：

```jsx
async function foo() {
  console.log(2);
  await null;
  console.log(4);
}

console.log(1);
foo();
console.log(3);
// 输出顺序：1, 2, 3, 4
```

执行流程：

1. 打印 `1`
2. 调用 `foo()`，打印 `2`
3. 遇到 `await`，**暂停** `foo()` 的执行，向微任务队列添加一个任务
4. `foo()` 退出，打印 `3`
5. 同步代码执行完毕，从微任务队列取出任务，恢复 `foo()` 的执行，打印 `4`

### 11.3.5 异步函数策略

#### 实现 sleep()

```jsx
async function sleep(delay) {
  return new Promise((resolve) => setTimeout(resolve, delay));
}

async function foo() {
  console.log("开始");
  await sleep(1500);
  console.log("1.5秒后");
}
```

#### 利用平行执行

当多个异步操作彼此**不依赖**时，应让它们**同时开始**：

```jsx
// ❌ 串行执行，效率低
async function slow() {
  await fetch("/api/a");
  await fetch("/api/b");
  await fetch("/api/c");
}

// ✅ 并行执行，效率高
async function fast() {
  const [a, b, c] = await Promise.all([
    fetch("/api/a"),
    fetch("/api/b"),
    fetch("/api/c")
  ]);
}
```

#### 串行执行期约

当异步任务有先后**依赖**时，使用 `for` 循环串行执行：

```jsx
async function serialExecution(tasks) {
  const results = [];
  for (const task of tasks) {
    results.push(await task());
  }
  return results;
}
```

#### 栈追踪与内存管理

<aside>
⚠️

**性能注意**：JavaScript 引擎会在创建期约时保存栈追踪信息。如果大量期约未被 `await`，可能导致内存占用增加。`async/await` 相比手动 Promise 链在栈追踪方面更友好，因为引擎能更好地标记和清理。

</aside>

---

## 本章小结

| 概念 | 说明 | 关键点 |
| --- | --- | --- |
| **回调函数** | 早期异步编程模式 | 嵌套深、错误处理难、耦合高 |
| **Promise** | 异步操作的"占位符"对象 | 三种状态（pending/fulfilled/rejected）、不可逆、链式调用 |
| **then/catch/finally** | 期约实例方法 | 都返回新期约；finally 用于清理 |
| **Promise.all()** | 全部兑现才兑现 | 任一拒绝即拒绝，结果为数组 |
| **Promise.race()** | 采用最先落定的结果 | 第一个落定即为最终结果 |
| **async/await** | Promise 的语法糖 | 同步写法、try/catch 错误处理、await 暂停执行 |