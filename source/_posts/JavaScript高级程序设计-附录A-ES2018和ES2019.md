---
title: 附录A ES2018和ES2019
date: 2026-02-17 15:02:21
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## A.1 异步迭代

### A.1.1 创建并使用异步迭代器

- 异步迭代器与同步迭代器类似，但 `next()` 方法返回的是 **Promise**
- 异步迭代器实现 `Symbol.asyncIterator` 方法
- 使用 `for-await-of` 循环来消费异步迭代器

```jsx
class AsyncCounter {
  constructor(limit) {
    this.limit = limit;
  }

  [Symbol.asyncIterator]() {
    let count = 0;
    const limit = this.limit;
    return {
      async next() {
        if (count < limit) {
          return { done: false, value: count++ };
        }
        return { done: true, value: undefined };
      }
    };
  }
}

(async () => {
  for await (const num of new AsyncCounter(5)) {
    console.log(num); // 0, 1, 2, 3, 4
  }
})();
```

### A.1.2 理解异步迭代器队列

- 异步迭代器会按顺序处理每个 `next()` 调用
- 如果前一个 Promise 还未解决，后续调用会排队等待
- 每次调用 `next()` 都会返回一个新的 Promise，保证**顺序消费**

### A.1.3 处理异步迭代器的 reject

- `for-await-of` 循环中如果某个 Promise 被拒绝，循环会**终止**并抛出错误
- 可以使用 `try/catch` 来捕获被拒绝的 Promise

```jsx
async function* generateErrors() {
  yield 1;
  throw new Error('出错了');
  yield 2; // 不会执行
}

(async () => {
  try {
    for await (const val of generateErrors()) {
      console.log(val);
    }
  } catch (e) {
    console.log('捕获错误:', e.message);
  }
})();
```

### A.1.4 使用 `next()` 手动异步迭代

```jsx
const asyncIterable = {
  [Symbol.asyncIterator]() {
    let i = 0;
    return {
      next() {
        if (i < 3) {
          return Promise.resolve({ value: i++, done: false });
        }
        return Promise.resolve({ value: undefined, done: true });
      }
    };
  }
};

const asyncIterator = asyncIterable[Symbol.asyncIterator]();
asyncIterator.next().then(console.log); // { value: 0, done: false }
asyncIterator.next().then(console.log); // { value: 1, done: false }
asyncIterator.next().then(console.log); // { value: 2, done: false }
asyncIterator.next().then(console.log); // { value: undefined, done: true }
```

---

## A.2 对象字面量的剩余操作符和扩展操作符

### A.2.1 对象的剩余操作符（Rest）

- 用于**解构赋值**中，将剩余属性收集到一个新对象

```jsx
const person = { name: 'Matt', age: 27, job: 'Engineer' };
const { name, ...rest } = person;
console.log(name); // 'Matt'
console.log(rest); // { age: 27, job: 'Engineer' }
```

<aside>
⚠️

剩余操作符必须放在解构模式的**最后一个位置**，否则会报语法错误。

</aside>

### A.2.2 对象的扩展操作符（Spread）

- 用于将一个对象的**所有可枚举自有属性**浅拷贝到新对象中

```jsx
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 };
console.log(obj2); // { a: 1, b: 2, c: 3 }
```

- 扩展操作符执行的是**浅拷贝**（引用类型的值仍然是同一个引用）
- 后面的属性会**覆盖**前面的同名属性

```jsx
const merged = { ...{ a: 1 }, ...{ a: 2 } };
console.log(merged.a); // 2
```

---

## A.3 Promise.prototype.finally()

- `finally()` 处理程序在 Promise 转换为 **resolved** 或 **rejected** 状态时都会执行
- 主要用于清理操作（如隐藏加载指示器），避免在 `then()` 和 `catch()` 中重复同样的逻辑
- `finally()` **不接收任何参数**，因为无法区分 Promise 是解决还是拒绝
- `finally()` 返回的 Promise 默认会**传递原来的值或拒绝理由**

```jsx
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(err => console.error(err))
  .finally(() => {
    console.log('请求完成，无论成功或失败');
    // 隐藏加载动画等清理工作
  });
```

---

## A.4 正则表达式相关特性

### A.4.1 dotAll 标志（`s` 标志）

- 默认情况下，正则中的 `.` 不匹配换行符（`\n`、`\r`、行分隔符、段分隔符）
- 使用 `s`（dotAll）标志后，`.` 可以匹配**任意字符**，包括换行符

```jsx
const text = 'foo\nbar';
console.log(/foo.bar/.test(text));  // false
console.log(/foo.bar/s.test(text)); // true
```

### A.4.2 后行断言（Lookbehind Assertions）

- **正向后行断言** `(?<=...)` — 匹配前面是指定模式的位置
- **负向后行断言** `(?<!...)` — 匹配前面不是指定模式的位置

```jsx
// 正向后行断言：匹配$后面的数字
const price = '$10.53';
console.log(price.match(/(?<=\$)\d+\.\d+/)); // ['10.53']

// 负向后行断言：匹配前面不是$的数字
const str = '€10 $20';
console.log(str.match(/(?<!\$)\d+/)); // ['10']
```

### A.4.3 命名捕获组

- 使用 `(?<name>...)` 为捕获组命名，通过 `groups.name` 访问

```jsx
const re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const match = re.exec('2019-01-02');
console.log(match.groups.year);  // '2019'
console.log(match.groups.month); // '01'
console.log(match.groups.day);   // '02'
```

- 在 `replace()` 中使用 `$<name>` 引用命名捕获组

```jsx
const result = '2019-01-02'.replace(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,
  '$<day>/$<month>/$<year>'
);
console.log(result); // '02/01/2019'
```

### A.4.4 Unicode 属性转义

- 使用 `\p{...}` 和 `\P{...}`（配合 `u` 标志）匹配 Unicode 属性

```jsx
// 匹配所有希腊字母
const greekRe = /\p{Script=Greek}/u;
console.log(greekRe.test('π')); // true
console.log(greekRe.test('a')); // false
```

---

## A.5 数组打平方法

### A.5.1 `Array.prototype.flat()`

- 将嵌套数组**打平**指定深度，默认深度为 **1**

```jsx
const arr = [1, [2, [3, [4]]]];
console.log(arr.flat());          // [1, 2, [3, [4]]]
console.log(arr.flat(2));         // [1, 2, 3, [4]]
console.log(arr.flat(Infinity));  // [1, 2, 3, 4]
```

### A.5.2 `Array.prototype.flatMap()`

- 等价于先执行 `map()`，再执行深度为 1 的 `flat()`

```jsx
const arr = [1, 2, 3];
const result = arr.flatMap(x => [x, x * 2]);
console.log(result); // [1, 2, 2, 4, 3, 6]
```

---

## A.6 `Object.fromEntries()`

- 将**键值对数组**（或任何可迭代的键值对）转换为对象
- 是 `Object.entries()` 的**逆操作**

```jsx
const entries = [['name', 'Matt'], ['age', 27]];
const obj = Object.fromEntries(entries);
console.log(obj); // { name: 'Matt', age: 27 }
```

- 常用于将 `Map` 转换为普通对象

```jsx
const map = new Map([['a', 1], ['b', 2]]);
const obj = Object.fromEntries(map);
console.log(obj); // { a: 1, b: 2 }
```

---

## A.7 字符串修剪方法

- `String.prototype.trimStart()` — 去除字符串**开头**的空白字符
- `String.prototype.trimEnd()` — 去除字符串**末尾**的空白字符
- 它们分别是 `trimLeft()` 和 `trimRight()` 的**标准别名**

```jsx
const str = '  hello  ';
console.log(str.trimStart()); // 'hello  '
console.log(str.trimEnd());   // '  hello'
console.log(str.trim());      // 'hello'
```

---

## A.8 `Symbol.prototype.description`

- 提供一种直接访问 Symbol 描述的只读属性
- 替代之前通过 `toString()` 获取描述的方式

```jsx
const sym = Symbol('mySymbol');
console.log(sym.description); // 'mySymbol'
console.log(Symbol().description); // undefined
```

---

## A.9 可选的 catch 绑定

- ES2019 允许在 `catch` 块中**省略错误参数**
- 当不需要使用 error 对象时非常有用

```jsx
// ES2019 之前
try {
  // ...
} catch (e) {
  // 即使不使用 e 也必须声明
}

// ES2019
try {
  // ...
} catch {
  // 可以省略错误参数
  console.log('出错了');
}
```

---

## A.10 `JSON.stringify()` 的修订

- 修复了 `JSON.stringify()` 对 **lone surrogates**（单独的代理项）的处理
- 之前对于无法配对的 UTF-16 代理项会返回无效的 Unicode 字符串
- ES2019 后，这些代理项会被转义为 `\uXXXX` 的形式，确保输出始终是有效的 UTF-8

---

## A.11 `Array.prototype.sort()` 的稳定性

- ES2019 规范要求 `Array.prototype.sort()` 必须是**稳定排序**
- 即排序后，**相等元素的相对顺序保持不变**
- 大多数现代引擎在此之前就已经实现了稳定排序

---

## A.12 `Function.prototype.toString()` 的修订

- ES2019 要求 `toString()` 返回函数的**精确源代码**，包括空格和注释
- 如果源代码不可用（如原生函数），则返回标准化的占位字符串

```jsx
function /* 注释 */ foo() { /* 内部注释 */ }
console.log(foo.toString());
// "function /* 注释 */ foo() { /* 内部注释 */ }"
```