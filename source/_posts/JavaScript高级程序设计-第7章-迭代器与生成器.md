---
title: 第7章 迭代器与生成器
date: 2026-02-17 15:02:01
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 7.1 理解迭代

**迭代**（iteration）是指按照顺序反复多次执行一段程序，通常会有明确的终止条件。

在 JavaScript 早期，执行迭代必须使用循环或其他辅助结构。随着代码量增加，这种方式变得越来越混乱。很多语言通过原生语言结构来解决这个问题，开发者无须事先知道如何迭代就能实现迭代操作——**迭代器模式**就是这样的结构。

### 早期迭代的问题

以 `for` 循环遍历数组为例：

```jsx
let collection = ['foo', 'bar', 'baz'];
for (let index = 0; index < collection.length; ++index) {
  console.log(collection[index]);
}
```

这种方式存在的问题：

- **迭代之前需要事先知道如何使用数据结构**（数组需要通过 `[]` 操作符、`length` 属性取值）
- **遍历顺序并不是数据结构固有的**（通过递增索引来访问数据是特定于数组的方式，并不适用于所有数据结构）

> ES5 新增了 `Array.prototype.forEach()` 方法，向通用迭代需求迈进了一步，但仍然不够理想——无法标识迭代何时终止，回调结构也比较笨拙。
> 

---

## 7.2 迭代器模式

**迭代器模式**描述了一个方案：把有些结构称为"**可迭代对象**"（iterable），因为它们实现了正式的 `Iterable` 接口，并且可以通过"**迭代器**"（`Iterator`）来消费。

### 7.2.1 可迭代协议（Iterable Protocol）

实现 `Iterable` 接口要求同时具备两种能力：

1. 支持迭代的**自我识别**能力
2. 创建实现 `Iterator` 接口的对象的能力

这意味着必须暴露一个属性作为"**默认迭代器**"，而且这个属性必须使用特殊的 `Symbol.iterator` 作为键。

**内置可迭代类型**（默认实现了 `Iterable` 接口）：

- `String`
- `Array`
- `Map`
- `Set`
- `arguments` 对象
- `NodeList` 等 DOM 集合类型
- `TypedArray`

```jsx
let str = 'abc';
let arr = ['a', 'b', 'c'];
let map = new Map().set('a', 1).set('b', 2).set('c', 3);
let set = new Set().add('a').add('b').add('c');

// 检查是否存在默认迭代器
console.log(str[Symbol.iterator]); // f values() { [native code] }
console.log(arr[Symbol.iterator]); // f values() { [native code] }
console.log(map[Symbol.iterator]); // f entries() { [native code] }
console.log(set[Symbol.iterator]); // f values() { [native code] }
```

**接收可迭代对象的原生语言特性**包括：

- `for...of` 循环
- 数组解构
- 扩展操作符 `...`
- `Array.from()`
- 创建 `Set` / `Map`
- `Promise.all()` / `Promise.race()`
- `yield*` 操作符

### 7.2.2 迭代器协议（Iterator Protocol）

迭代器是一种**一次性使用**的对象，用于迭代与其关联的可迭代对象。迭代器使用 `next()` 方法在可迭代对象中遍历数据。

每次成功调用 `next()`，都会返回一个 `IteratorResult` 对象，包含两个属性：

- **`done`**：布尔值，表示是否还可以再次调用 `next()` 取得下一个值（`false` 表示未耗尽，`true` 表示已耗尽）
- **`value`**：可迭代对象的下一个值（`done` 为 `true` 时为 `undefined`）

```jsx
let arr = ['foo', 'bar'];
let iter = arr[Symbol.iterator]();

console.log(iter.next()); // { done: false, value: 'foo' }
console.log(iter.next()); // { done: false, value: 'bar' }
console.log(iter.next()); // { done: true, value: undefined }
```

<aside>
💡

每个迭代器都表示对可迭代对象的**一次性有序遍历**。不同迭代器的实例相互之间没有联系，只会独立地遍历可迭代对象。

</aside>

### 7.2.3 自定义迭代器

任何实现了 `Iterator` 接口的对象都可以作为迭代器使用：

```jsx
class Counter {
  constructor(limit) {
    this.limit = limit;
  }

  [Symbol.iterator]() {
    let count = 1, limit = this.limit;
    return {
      next() {
        if (count <= limit) {
          return { done: false, value: count++ };
        } else {
          return { done: true, value: undefined };
        }
      }
    };
  }
}

let counter = new Counter(3);
for (let i of counter) { console.log(i); }
// 1
// 2
// 3
```

### 7.2.4 提前终止迭代器

可选的 `return()` 方法用于指定在迭代器提前关闭时执行的逻辑。可能的情况包括：

- `for...of` 循环通过 `break`、`continue`、`return` 或 `throw` 提前退出
- 解构操作并未消费所有值

```jsx
class Counter {
  constructor(limit) {
    this.limit = limit;
  }

  [Symbol.iterator]() {
    let count = 1, limit = this.limit;
    return {
      next() {
        if (count <= limit) {
          return { done: false, value: count++ };
        } else {
          return { done: true, value: undefined };
        }
      },
      return() {
        console.log('Exiting early');
        return { done: true };
      }
    };
  }
}

let counter = new Counter(5);
for (let i of counter) {
  if (i > 2) break;
  console.log(i);
}
// 1
// 2
// Exiting early
```

<aside>
⚠️

如果迭代器没有关闭，则还可以从上次离开的地方继续迭代。比如数组的迭代器就是不能关闭的。

</aside>

---

## 7.3 生成器

**生成器**是 ES6 新增的一个极为灵活的结构，拥有在一个函数块内**暂停和恢复代码执行**的能力。

### 7.3.1 生成器基础

生成器的形式是一个函数，函数名称前面加一个星号 `*` 表示它是一个生成器。只要是可以定义函数的地方，就可以定义生成器。

```jsx
// 生成器函数声明
function* generatorFn() {}

// 生成器函数表达式
let generatorFn = function* () {};

// 作为对象字面量方法的生成器函数
let foo = {
  * generatorFn() {}
};

// 作为类实例方法的生成器函数
class Foo {
  * generatorFn() {}
}

// 作为类静态方法的生成器函数
class Bar {
  static * generatorFn() {}
}
```

<aside>
⚠️

**箭头函数不能用来定义生成器函数。**

</aside>

调用生成器函数会产生一个**生成器对象**（Generator Object）。生成器对象一开始处于**暂停执行**（suspended）的状态，与迭代器相似，也实现了 `Iterator` 接口，具有 `next()` 方法。

```jsx
function* generatorFn() {}

const g = generatorFn();

console.log(g);       // generatorFn {<suspended>}
console.log(g.next);  // f next() { [native code] }
```

### 7.3.2 通过 yield 中断执行

`yield` 关键字可以让生成器停止和开始执行，也是生成器最有用的地方。

生成器函数在遇到 `yield` 关键字之前会正常执行。遇到这个关键字后，执行会停止，函数作用域的状态会被保留。停止执行的生成器函数只能通过在生成器对象上调用 `next()` 方法来恢复执行。

```jsx
function* generatorFn() {
  yield 'foo';
  yield 'bar';
  return 'baz';
}

let generatorObject = generatorFn();

console.log(generatorObject.next()); // { done: false, value: 'foo' }
console.log(generatorObject.next()); // { done: false, value: 'bar' }
console.log(generatorObject.next()); // { done: true,  value: 'baz' }
```

<aside>
💡

**`yield` 关键字只能在生成器函数内部使用**，用在其他地方会抛出错误。类似于函数的 `return` 关键字，`yield` 关键字必须直接位于生成器函数定义中，出现在嵌套的非生成器函数中会抛出语法错误。

</aside>

#### 1. 生成器对象作为可迭代对象

因为生成器对象实现了 `Iterable` 接口，所以可以用在任何消费可迭代对象的地方：

```jsx
function* generatorFn() {
  yield 1;
  yield 2;
  yield 3;
}

for (const x of generatorFn()) {
  console.log(x);
}
// 1
// 2
// 3
```

#### 2. 使用 yield 实现输入和输出

`yield` 关键字还可以作为函数的**中间参数**使用。上一次让生成器函数暂停的 `yield` 关键字会接收到传给 `next()` 方法的第一个值：

```jsx
function* generatorFn(initial) {
  console.log(initial);
  console.log(yield);
  console.log(yield);
}

let generatorObject = generatorFn('foo');

generatorObject.next('bar');  // foo  （第一次调用 next() 传入的值不会被使用）
generatorObject.next('baz');  // baz
generatorObject.next('qux');  // qux
```

#### 3. 产生可迭代对象

可以使用星号增强 `yield` 的行为，让它能够迭代一个可迭代对象，从而一次产出一个值：

```jsx
function* generatorFn() {
  yield* [1, 2, 3];
}

for (const x of generatorFn()) {
  console.log(x);
}
// 1
// 2
// 3
```

`yield*` 的值是关联迭代器返回 `done: true` 时的 `value` 属性：

```jsx
function* innerGeneratorFn() {
  yield 'foo';
  return 'bar';
}

function* outerGeneratorFn() {
  console.log('iter value:', yield* innerGeneratorFn());
}

for (const x of outerGeneratorFn()) {
  console.log('value:', x);
}
// value: foo
// iter value: bar
```

#### 4. 使用 yield* 实现递归算法

`yield*` 最有用的地方是实现递归操作，此时生成器可以产生自身：

```jsx
function* nTimes(n) {
  if (n > 0) {
    yield* nTimes(n - 1);
    yield n - 1;
  }
}

for (const x of nTimes(3)) {
  console.log(x);
}
// 0
// 1
// 2
```

### 7.3.3 生成器作为默认迭代器

因为生成器对象实现了 `Iterable` 接口，而且生成器函数和默认迭代器被调用后都产生迭代器，所以生成器格外适合作为默认迭代器：

```jsx
class Foo {
  constructor() {
    this.values = [1, 2, 3];
  }

  * [Symbol.iterator]() {
    yield* this.values;
  }
}

const f = new Foo();
for (const x of f) {
  console.log(x);
}
// 1
// 2
// 3
```

### 7.3.4 提前终止生成器

与迭代器类似，生成器也支持"可关闭"的概念。生成器对象除了 `next()` 外还有两个方法：

#### 1. return()

`return()` 方法会强制生成器进入关闭状态。提供给 `return()` 方法的值就是终止迭代器对象的值：

```jsx
function* generatorFn() {
  yield* [1, 2, 3];
}

const g = generatorFn();

console.log(g.next());     // { done: false, value: 1 }
console.log(g.return(4));  // { done: true,  value: 4 }
console.log(g.next());     // { done: true,  value: undefined }
```

#### 2. throw()

`throw()` 方法会在暂停的时候将一个提供的错误注入到生成器对象中。如果错误未被处理，生成器就会关闭：

```jsx
function* generatorFn() {
  for (const x of [1, 2, 3]) {
    yield x;
  }
}

const g = generatorFn();
console.log(g); // generatorFn {<suspended>}

try {
  g.throw('foo');
} catch (e) {
  console.log(e); // foo
}

console.log(g); // generatorFn {<closed>}
```

如果生成器函数**内部处理了**这个错误，那么生成器就不会关闭，还可以恢复执行：

```jsx
function* generatorFn() {
  for (const x of [1, 2, 3]) {
    try {
      yield x;
    } catch (e) {}
  }
}

const g = generatorFn();

console.log(g.next());    // { done: false, value: 1 }
g.throw('foo');
console.log(g.next());    // { done: false, value: 3 }
```

<aside>
⚠️

如果生成器对象还没有开始执行（即还没有调用过 `next()`），那么调用 `throw()` 抛出的错误**不会在函数内部被捕获**，因为这相当于在函数块外部抛出了错误。

</aside>

---

## 7.4 小结

<aside>
📌

- **迭代器**是一个可以由任意对象实现的接口，支持连续获取值。任何实现了 `Iterable` 接口的对象都有一个 `Symbol.iterator` 属性，该属性引用默认迭代器。默认迭代器就像一个迭代器工厂，调用后产出一个实现了 `Iterator` 接口的对象。
- **迭代器协议**定义了 `next()` 方法，每次调用该方法都会返回一个 `IteratorResult` 对象，其中包含迭代器返回的下一个值。
- **生成器**是一种特殊函数，调用后返回一个生成器对象。生成器对象实现了 `Iterable` 接口，因此可用在任何消费可迭代对象的地方。
- 生成器的独特之处在于支持 **`yield`** 关键字，能够暂停执行生成器函数。使用 `yield` 关键字还可以通过 `next()` 方法接收输入和产出输出。
- **`yield*`** 可以将跟在它后面的可迭代对象序列化为一连串值，实现递归与组合。
</aside>