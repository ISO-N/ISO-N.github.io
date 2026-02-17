---
title: 第9章 代理与反射
date: 2026-02-17 15:02:02
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
ECMAScript 6 新增的**代理（Proxy）**和**反射（Reflect）**为开发者提供了**拦截**并向基本操作嵌入额外行为的能力。具体来说，可以给目标对象定义一个关联的代理对象，而这个代理对象可以作为抽象的目标对象来使用。在对目标对象的各种操作影响目标对象之前，可以在代理对象中对这些操作加以控制。

## 9.1 代理基础

### 9.1.1 创建空代理

最简单的代理是**空代理**——除了作为一个抽象的目标对象，什么也不做。在代理对象上执行的所有操作都会无障碍地传播到目标对象。

代理使用 `Proxy` 构造函数创建，接收两个**必需**参数：**目标对象（target）**和**处理程序对象（handler）**。

```jsx
const target = { id: 'target' };
const handler = {};

const proxy = new Proxy(target, handler);

// id 属性会访问同一个值
console.log(target.id);  // target
console.log(proxy.id);   // target

// 给目标属性赋值会反映在两个对象上
target.id = 'foo';
console.log(target.id);  // foo
console.log(proxy.id);   // foo

// 给代理属性赋值会反映在两个对象上
proxy.id = 'bar';
console.log(target.id);  // bar
console.log(proxy.id);   // bar

// Proxy.prototype 是 undefined，因此不能使用 instanceof
// TypeError: Function has non-object prototype 'undefined' in instanceof check
// console.log(target instanceof Proxy);

// 严格相等可以区分代理和目标
console.log(target === proxy);  // false
```

### 9.1.2 定义捕获器（trap）

使用代理的主要目的是可以定义**捕获器（trap）**。捕获器是在处理程序对象中定义的"基本操作的拦截器"。每个处理程序对象可以包含零个或多个捕获器，每个捕获器都对应一种基本操作。

```jsx
const target = { foo: 'bar' };

const handler = {
  // 捕获器在处理程序对象中以方法名为键
  get() {
    return 'handler override';
  }
};

const proxy = new Proxy(target, handler);

console.log(target.foo);                // bar
console.log(proxy.foo);                 // handler override
console.log(proxy['foo']);              // handler override
console.log(Object.create(proxy).foo); // handler override
```

### 9.1.3 捕获器参数与反射 API

所有捕获器都可以访问相应的参数，基于这些参数可以重建被捕获方法的原始行为。比如 `get()` 捕获器会接收到 **目标对象**、**要查询的属性** 和 **代理对象** 三个参数：

```jsx
const target = { foo: 'bar' };

const handler = {
  get(trapTarget, property, receiver) {
    console.log(trapTarget === target);
    console.log(property);
    console.log(receiver === proxy);
    return trapTarget[property];
  }
};

const proxy = new Proxy(target, handler);
proxy.foo;
// true
// foo
// true
```

开发者并不需要手动重建原始行为，而是可以通过调用全局 **`Reflect`** 对象上的同名方法来轻松重建：

```jsx
const target = { foo: 'bar' };

const handler = {
  get() {
    return Reflect.get(...arguments);
  }
};

const proxy = new Proxy(target, handler);
console.log(proxy.foo);  // bar
```

甚至可以更简洁：

```jsx
const handler = {
  get: Reflect.get
};
```

如果想创建一个**可以捕获所有方法，然后将每个方法转发给对应反射 API** 的空代理，不需要定义处理程序对象，直接用 `Reflect` 即可：

```jsx
const proxy = new Proxy(target, Reflect);
```

### 9.1.4 捕获器不变式（trap invariant）

使用捕获器几乎可以改变所有基本方法的行为，但也不是没有限制。每个捕获的方法都知道目标对象上下文、捕获函数签名，而捕获处理程序的行为必须遵循**"捕获器不变式"**。

比如，如果目标对象有一个不可配置且不可写的数据属性，那么捕获器在返回一个与该属性不同的值时，会抛出 `TypeError`：

```jsx
const target = {};
Object.defineProperty(target, 'foo', {
  configurable: false,
  writable: false,
  value: 'bar'
});

const handler = {
  get() {
    return 'qux';
  }
};

const proxy = new Proxy(target, handler);
console.log(proxy.foo);
// TypeError
```

### 9.1.5 可撤销代理

有时候可能需要中断代理对象与目标对象之间的联系。使用 `new Proxy()` 创建的普通代理的联系在代理对象的生命周期内一直持续存在。

`Proxy.revocable()` 方法支持撤销代理对象与目标对象的关联。撤销代理的操作是**不可逆**的。撤销函数（`revoke()`）是**幂等**的，调用多少次结果都一样。撤销后再调用代理会抛出 `TypeError`。

```jsx
const target = { foo: 'bar' };
const handler = {
  get() {
    return 'intercepted';
  }
};

const { proxy, revoke } = Proxy.revocable(target, handler);

console.log(proxy.foo);  // intercepted
console.log(target.foo); // bar

revoke();

console.log(proxy.foo);  // TypeError
```

### 9.1.6 实用反射 API

**Reflect API 与 Object API 的区别：**

1. **反射 API 并不限于捕获处理程序**
2. 大多数反射 API 方法在 `Object` 类型上有对应的方法

通常，`Object` 上的方法适用于通用程序，而反射方法适用于**细粒度的对象控制与操作**。

**状态标记（status flags）：**

很多反射方法返回布尔值（称为"状态标记"），表示意图执行的操作是否成功。这比抛异常更优雅：

```jsx
// 初始代码（可能抛异常）
const o = {};
try {
  Object.defineProperty(o, 'foo', 'bar');
  console.log('success');
} catch (e) {
  console.log('failure');
}

// 重构后使用 Reflect（返回布尔值）
const o = {};
if (Reflect.defineProperty(o, 'foo', { value: 'bar' })) {
  console.log('success');
} else {
  console.log('failure');
}
```

以下反射方法都会提供状态标记：

- `Reflect.defineProperty()`
- `Reflect.preventExtensions()`
- `Reflect.setPrototypeOf()`
- `Reflect.set()`
- `Reflect.deleteProperty()`

**用一等函数替代操作符：**

- `Reflect.get()` — 替代对象属性访问操作符
- `Reflect.set()` — 替代 `=` 赋值操作符
- `Reflect.has()` — 替代 `in` 操作符或 `with()`
- `Reflect.deleteProperty()` — 替代 `delete` 操作符
- `Reflect.construct()` — 替代 `new` 操作符

### 9.1.7 代理另一个代理

代理可以拦截反射 API 的操作，而这意味着完全可以创建一个代理，通过它去代理另一个代理，从而在一个目标对象之上构建**多层拦截网**：

```jsx
const target = { foo: 'bar' };

const firstProxy = new Proxy(target, {
  get() {
    console.log('first proxy');
    return Reflect.get(...arguments);
  }
});

const secondProxy = new Proxy(firstProxy, {
  get() {
    console.log('second proxy');
    return Reflect.get(...arguments);
  }
});

console.log(secondProxy.foo);
// second proxy
// first proxy
// bar
```

### 9.1.8 代理的问题与不足

1. **代理中的 `this`**：代理会改变 `this` 的指向——方法中的 `this` 通常会指向代理对象而非目标对象。若目标对象依赖自身标识（如 `WeakMap`），可能会出现问题。
2. **代理与内部槽位**：某些 ECMAScript 内置类型（如 `Date`）可能依赖代理无法控制的机制——**内部槽位（internal slots）**。比如 `Date` 类型的方法会依赖 `this` 值上的 `[[NumberDate]]` 内部槽位，代理对象上不存在此槽位，调用相关方法会抛出 `TypeError`。

---

## 9.2 代理捕获器与反射方法

代理可以捕获 **13 种**不同的基本操作，每种操作都有对应的反射 API 方法。

### 9.2.1 get()

- **捕获器**：`get(target, property, receiver)`
- **对应反射 API**：`Reflect.get()`
- **拦截的操作**：`proxy.property` / `proxy[property]` / `Object.create(proxy)[property]`
- **捕获器不变式**：
    - 若目标属性不可写且不可配置，处理程序返回值必须与目标属性值匹配
    - 若目标属性不可配置且 `[[Get]]` 为 `undefined`，处理程序返回值也必须是 `undefined`

### 9.2.2 set()

- **捕获器**：`set(target, property, value, receiver)`
- **对应反射 API**：`Reflect.set()`
- **拦截的操作**：`proxy.property = value` / `proxy[property] = value` / `Object.create(proxy)[property] = value`
- **捕获器不变式**：若目标属性不可写且不可配置，则不能修改目标属性的值

### 9.2.3 has()

- **捕获器**：`has(target, property)`
- **对应反射 API**：`Reflect.has()`
- **拦截的操作**：`property in proxy` / `with(proxy) { property; }`
- **捕获器不变式**：若目标属性存在且不可配置，处理程序必须返回 `true`；若目标属性存在且目标对象不可扩展，处理程序必须返回 `true`

### 9.2.4 defineProperty()

- **捕获器**：`defineProperty(target, property, descriptor)`
- **对应反射 API**：`Reflect.defineProperty()`
- **拦截的操作**：`Object.defineProperty(proxy, property, descriptor)`

### 9.2.5 getOwnPropertyDescriptor()

- **捕获器**：`getOwnPropertyDescriptor(target, property)`
- **对应反射 API**：`Reflect.getOwnPropertyDescriptor()`
- **拦截的操作**：`Object.getOwnPropertyDescriptor(proxy, property)`

### 9.2.6 deleteProperty()

- **捕获器**：`deleteProperty(target, property)`
- **对应反射 API**：`Reflect.deleteProperty()`
- **拦截的操作**：`delete proxy.property` / `delete proxy[property]`

### 9.2.7 ownKeys()

- **捕获器**：`ownKeys(target)`
- **对应反射 API**：`Reflect.ownKeys()`
- **拦截的操作**：`Object.getOwnPropertyNames(proxy)` / `Object.getOwnPropertySymbols(proxy)` / `Object.keys(proxy)`

### 9.2.8 getPrototypeOf()

- **捕获器**：`getPrototypeOf(target)`
- **对应反射 API**：`Reflect.getPrototypeOf()`
- **拦截的操作**：`Object.getPrototypeOf(proxy)`

### 9.2.9 setPrototypeOf()

- **捕获器**：`setPrototypeOf(target, prototype)`
- **对应反射 API**：`Reflect.setPrototypeOf()`
- **拦截的操作**：`Object.setPrototypeOf(proxy, prototype)`

### 9.2.10 isExtensible()

- **捕获器**：`isExtensible(target)`
- **对应反射 API**：`Reflect.isExtensible()`
- **拦截的操作**：`Object.isExtensible(proxy)`

### 9.2.11 preventExtensions()

- **捕获器**：`preventExtensions(target)`
- **对应反射 API**：`Reflect.preventExtensions()`
- **拦截的操作**：`Object.preventExtensions(proxy)`

### 9.2.12 apply()

- **捕获器**：`apply(target, thisArg, argumentsList)`
- **对应反射 API**：`Reflect.apply()`
- **拦截的操作**：`proxy(...args)` / `Function.prototype.apply(thisArg, args)` / `Function.prototype.call(thisArg, ...args)`

### 9.2.13 construct()

- **捕获器**：`construct(target, argumentsList, newTarget)`
- **对应反射 API**：`Reflect.construct()`
- **拦截的操作**：`new proxy(...args)`
- **捕获器不变式**：`construct()` 必须返回一个对象

---

## 9.3 代理模式

使用代理可以实现一些有用的编程模式。

### 9.3.1 跟踪属性访问

通过捕获 `get`、`set` 和 `has` 等操作，可以知道对象属性什么时候被访问、被查询。把实现相应捕获器的某个对象代理放到应用中，可以**监控**这个对象何时在何处被访问过：

```jsx
const user = { name: 'Jake' };

const proxy = new Proxy(user, {
  get(target, property, receiver) {
    console.log(`Getting ${property}`);
    return Reflect.get(...arguments);
  },
  set(target, property, value, receiver) {
    console.log(`Setting ${property}=${value}`);
    return Reflect.set(...arguments);
  }
});

proxy.name;       // Getting name
proxy.age = 27;   // Setting age=27
```

### 9.3.2 隐藏属性

代理的内部实现对外部代码是不可见的，因此可以隐藏目标对象上的特定属性：

```jsx
const hiddenProperties = ['foo', 'bar'];
const targetObject = { foo: 1, bar: 2, baz: 3 };

const proxy = new Proxy(targetObject, {
  get(target, property) {
    if (hiddenProperties.includes(property)) {
      return undefined;
    }
    return Reflect.get(...arguments);
  },
  has(target, property) {
    if (hiddenProperties.includes(property)) {
      return false;
    }
    return Reflect.has(...arguments);
  }
});

console.log(proxy.foo);       // undefined
console.log(proxy.bar);       // undefined
console.log(proxy.baz);       // 3
console.log('foo' in proxy);  // false
console.log('baz' in proxy);  // true
```

### 9.3.3 属性验证

因为所有赋值操作都会触发 `set()` 捕获器，所以可以根据所赋的值决定是允许还是拒绝赋值：

```jsx
const target = { onlyNumbersGoHere: 0 };

const proxy = new Proxy(target, {
  set(target, property, value) {
    if (typeof value !== 'number') {
      return false;
    }
    return Reflect.set(...arguments);
  }
});

proxy.onlyNumbersGoHere = 1;
console.log(proxy.onlyNumbersGoHere);  // 1

proxy.onlyNumbersGoHere = '2';
console.log(proxy.onlyNumbersGoHere);  // 1（赋值被拒绝）
```

### 9.3.4 函数与构造函数参数验证

跟保护和验证对象属性类似，也可以对**函数和构造函数参数**进行审查。比如让函数只接收某种类型的值：

```jsx
function median(...nums) {
  return nums.sort()[Math.floor(nums.length / 2)];
}

const proxy = new Proxy(median, {
  apply(target, thisArg, argumentsList) {
    for (const arg of argumentsList) {
      if (typeof arg !== 'number') {
        throw new TypeError('Non-number argument provided');
      }
    }
    return Reflect.apply(...arguments);
  }
});

console.log(proxy(4, 7, 1));  // 4
console.log(proxy(4, '7', 1));
// TypeError: Non-number argument provided
```

类似地，可以要求实例化时必须传入参数：

```jsx
class User {
  constructor(id) {
    this.id_ = id;
  }
}

const proxy = new Proxy(User, {
  construct(target, argumentsList, newTarget) {
    if (argumentsList[0] === undefined) {
      throw new Error('User cannot be instantiated without id');
    }
    return Reflect.construct(...arguments);
  }
});

new proxy(1);      // ✅
new proxy();       // Error: User cannot be instantiated without id
```

### 9.3.5 数据绑定与可观察对象

通过代理可以把运行时中原本不相关的部分联系到一起，从而实现各种模式，实现**数据绑定与可观察对象**：

```jsx
const userList = [];

class User {
  constructor(name) {
    this.name_ = name;
  }
}

const proxy = new Proxy(User, {
  construct() {
    const newUser = Reflect.construct(...arguments);
    userList.push(newUser);
    return newUser;
  }
});

new proxy('John');
new proxy('Jacob');

console.log(userList);
// [User { name_: 'John' }, User { name_: 'Jacob' }]
```

还可以把集合绑定到一个事件分派程序，每次插入新实例时都会发送消息：

```jsx
const userList = [];

function emit(newValue) {
  console.log(newValue);
}

const proxy = new Proxy(userList, {
  set(target, property, value, receiver) {
    const result = Reflect.set(...arguments);
    if (result) {
      emit(Reflect.get(target, property, receiver));
    }
    return result;
  }
});

proxy.push('John');   // John
proxy.push('Jacob');  // Jacob
```

---

## 9.4 小结

<aside>
📝

- 代理是 ECMAScript 6 新增的令人兴奋且动态十足的特性。尽管不支持向后兼容，但它开辟了一片前所未有的**元编程和抽象**的新天地。
- 代理是**真实 JavaScript 对象**，通过它可以控制其背后目标对象的行为。
- 代理的行为实际上是**由捕获器定义**的，它们类似于操作系统中的中断。
- 可以定义 **13 种**捕获器。每种捕获器对应一或多种可以直接或间接触发的基本操作。
- 代理模式在实践中有很多用途，如**跟踪属性访问、隐藏属性、阻止修改或删除属性、函数参数验证、构造函数参数验证、数据绑定，以及可观察对象**。
</aside>