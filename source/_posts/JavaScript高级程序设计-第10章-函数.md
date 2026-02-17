---
title: 第10章 函数
date: 2026-02-17 15:02:03
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
函数是 ECMAScript 中最有意思的部分之一，这主要是因为**函数实际上是对象**。每个函数都是 `Function` 类型的实例，而 `Function` 也有属性和方法，跟其他引用类型一样。函数名就是指向函数对象的指针。

---

## 10.1 箭头函数

ES6 新增了使用 **胖箭头（`=>`）** 语法定义函数表达式的能力：

```jsx
// 以下两种写法等价
let double = function(x) { return 2 * x; };
let double = (x) => { return 2 * x; };

// 只有一个参数时可以省略括号
let double = x => { return 2 * x; };

// 函数体只有一行代码时可以省略大括号，并隐式返回该行的值
let double = x => 2 * x;
```

**箭头函数的限制：**

- 不能使用 `arguments`、`super`、`new.target`
- 不能用作构造函数（不能使用 `new`）
- 没有 `prototype` 属性
- **不绑定自己的 `this`**，会捕获其所在上下文的 `this` 值

---

## 10.2 函数名

函数名就是指向函数的指针，所以一个函数可以有多个名称：

```jsx
function sum(num1, num2) {
  return num1 + num2;
}

let anotherSum = sum;
console.log(anotherSum(10, 10)); // 20

sum = null;
console.log(anotherSum(10, 10)); // 20（仍然可以调用）
```

ES6 的所有函数对象都会暴露一个只读的 **`name`** 属性：

```jsx
function foo() {}
let bar = function() {};
let baz = () => {};

console.log(foo.name);             // "foo"
console.log(bar.name);             // "bar"
console.log(baz.name);             // "baz"
console.log((() => {}).name);      // ""（空字符串）
console.log((new Function()).name); // "anonymous"
```

如果函数是 `get`/`set` 函数或使用了 `bind()`，标识符前面会加上前缀：

```jsx
console.log(foo.bind(null).name);  // "bound foo"

let obj = {
  get name() { return 'bar'; }
};
// "get name"
console.log(Object.getOwnPropertyDescriptor(obj, 'name').get.name);
```

---

## 10.3 理解参数

ECMAScript 函数的参数在内部表现为一个**类数组对象 `arguments`**。函数不关心传入了多少个参数，也不关心参数的数据类型：

```jsx
function sayHi() {
  console.log("Hello " + arguments[0] + ", " + arguments[1]);
}
sayHi("World", "!"); // "Hello World, !"
```

**`arguments` 对象的特点：**

- 可以通过 `arguments.length` 检查传入参数的个数
- `arguments` 的值始终与对应的命名参数**同步**（非严格模式下）
- 严格模式下，`arguments` 与命名参数**不会同步**，且不能对 `arguments` 赋值
- **箭头函数中没有 `arguments` 对象**，但可以在箭头函数中访问外层非箭头函数的 `arguments`

---

## 10.4 没有重载

ECMAScript 函数**没有签名**，因为参数是由零个或多个值的数组表示的。没有函数签名，就不可能有真正的重载：

```jsx
function addSomeNumber(num) {
  return num + 100;
}

function addSomeNumber(num) {
  return num + 200;
}

console.log(addSomeNumber(100)); // 300（后定义的覆盖了先定义的）
```

> 可以通过检查参数的类型和数量，然后执行不同的逻辑来**模拟**重载。
> 

---

## 10.5 默认参数值

ES6 支持**显式定义默认参数值**：

```jsx
function makeKing(name = 'Henry') {
  return `King ${name} VIII`;
}

console.log(makeKing());          // "King Henry VIII"
console.log(makeKing('Louis'));   // "King Louis VIII"
console.log(makeKing(undefined)); // "King Henry VIII"
```

**关键特性：**

- `arguments` 对象始终以调用时传入的值为准，**不反映默认值**
- 默认值可以是**任意表达式**，且在函数调用时（而非定义时）求值
- 后定义的参数可以引用先定义的参数

```jsx
function makeKing(name = 'Henry', numerals = name) {
  return `King ${name} ${numerals}`;
}
console.log(makeKing()); // "King Henry Henry"
```

**默认参数作用域与暂时性死区（TDZ）：**

参数按顺序初始化，后定义的参数可以引用先定义的，但不能反过来引用：

```jsx
// 报错：ReferenceError
function makeKing(name = numerals, numerals = 'VIII') {
  return `King ${name} ${numerals}`;
}
```

---

## 10.6 参数扩展与收集

### 10.6.1 扩展参数（Spread）

使用**扩展操作符 `...`** 可以将可迭代对象拆分为单独的参数：

```jsx
function getSum() {
  let sum = 0;
  for (let i = 0; i < arguments.length; i++) {
    sum += arguments[i];
  }
  return sum;
}

let values = [1, 2, 3, 4];
console.log(getSum(...values));    // 10
console.log(getSum(-1, ...values)); // 9
console.log(getSum(...values, 5)); // 15
```

### 10.6.2 收集参数（Rest）

使用**剩余参数 `...`** 可以将不定数量的参数收集到一个数组中：

```jsx
function getSum(...values) {
  return values.reduce((prev, cur) => prev + cur, 0);
}

console.log(getSum(1, 2, 3)); // 6
```

- 剩余参数只能作为**最后一个**形参
- 箭头函数也支持剩余参数

---

## 10.7 函数声明与函数表达式

JavaScript 引擎对函数声明和函数表达式的处理方式不同：

- **函数声明**会被**提升**（function declaration hoisting），可以在声明之前调用
- **函数表达式**不会提升，必须先定义后使用

```jsx
// ✅ 正常工作（函数声明提升）
console.log(sum(10, 10)); // 20
function sum(a, b) {
  return a + b;
}

// ❌ 报错（函数表达式不提升）
console.log(sum(10, 10)); // TypeError
var sum = function(a, b) {
  return a + b;
};
```

---

## 10.8 函数作为值

函数名在 ECMAScript 中就是变量，所以函数可以**作为参数传递给另一个函数**，也可以**作为函数的返回值**：

```jsx
function callSomeFunction(someFunction, someArgument) {
  return someFunction(someArgument);
}

function add10(num) {
  return num + 10;
}

console.log(callSomeFunction(add10, 10)); // 20
```

从函数中返回一个函数是非常有用的技术，例如可以用于**按某个属性排序**：

```jsx
function createComparisonFunction(propertyName) {
  return function(obj1, obj2) {
    let val1 = obj1[propertyName];
    let val2 = obj2[propertyName];
    return val1 < val2 ? -1 : val1 > val2 ? 1 : 0;
  };
}

let data = [
  { name: "Zachary", age: 28 },
  { name: "Nicholas", age: 29 }
];

data.sort(createComparisonFunction("name"));
console.log(data[0].name); // "Nicholas"
```

---

## 10.9 函数内部

### 10.9.1 arguments

`arguments` 对象有一个 **`callee`** 属性，指向 `arguments` 对象所在的函数。这在**递归**中非常有用：

```jsx
// 经典阶乘函数（解耦函数名）
function factorial(num) {
  if (num <= 1) return 1;
  return num * arguments.callee(num - 1);
}
```

> ⚠️ 严格模式下访问 `arguments.callee` 会报错。
> 

### 10.9.2 this

`this` 在标准函数和箭头函数中的行为不同：

- **标准函数**中，`this` 引用的是调用函数的**上下文对象**（在全局中调用就是 `window` / `global`）
- **箭头函数**中，`this` 引用的是**定义箭头函数的上下文**

```jsx
window.color = 'red';
let o = { color: 'blue' };

function sayColor() {
  console.log(this.color);
}

sayColor();     // "red"（this === window）
o.sayColor = sayColor;
o.sayColor();   // "blue"（this === o）
```

```jsx
window.color = 'red';
let o = { color: 'blue' };

let sayColor = () => console.log(this.color);

sayColor();     // "red"
o.sayColor = sayColor;
o.sayColor();   // "red"（箭头函数的 this 不变）
```

> 在事件回调或定时回调中，箭头函数的 `this` 非常有用，因为它保留了外层上下文。
> 

### 10.9.3 caller

`caller` 属性引用的是**调用当前函数的函数**：

```jsx
function outer() {
  inner();
}

function inner() {
  console.log(inner.caller); // 输出 outer 的源代码
}

outer();
```

> ⚠️ 严格模式下访问 `caller` 会报错。
> 

### 10.9.4 [new.target](http://new.target)

ES6 新增了检测函数是否通过 `new` 关键字调用的 **`new.target`** 属性：

```jsx
function King() {
  if (!new.target) {
    throw 'King must be instantiated using "new"';
  }
  console.log('King instantiated using "new"');
}

new King();  // ✅ "King instantiated using 'new'"
King();      // ❌ Error: King must be instantiated using "new"
```

---

## 10.10 函数属性与方法

每个函数都有两个属性：**`length`** 和 **`prototype`**。

- **`length`**：保存函数定义的**命名参数个数**
- **`prototype`**：保存引用类型所有实例方法的地方（不可枚举）

### apply()、call() 和 bind()

这三个方法用于设置函数体内 `this` 的值：

```jsx
function sum(num1, num2) {
  return num1 + num2;
}

// apply()：参数以数组形式传入
function callSum1(num1, num2) {
  return sum.apply(this, [num1, num2]);
}

// call()：参数逐个传入
function callSum2(num1, num2) {
  return sum.call(this, num1, num2);
}

console.log(callSum1(10, 10)); // 20
console.log(callSum2(10, 10)); // 20
```

**`bind()`** 会创建一个**新函数**，其 `this` 值被绑定到传入的对象：

```jsx
window.color = 'red';
let o = { color: 'blue' };

function sayColor() {
  console.log(this.color);
}

let objectSayColor = sayColor.bind(o);
objectSayColor(); // "blue"
```

> `apply()` 和 `call()` 的真正强大之处在于**扩充函数运行的作用域**，对象不需要与方法有任何耦合关系。
> 

---

## 10.11 函数表达式

函数表达式看起来就像常规的变量赋值，创建的函数叫做**匿名函数**（也叫 *Lambda 函数*）：

```jsx
let functionName = function(arg0, arg1) {
  // 函数体
};
```

函数表达式与任何表达式一样，使用前必须赋值，否则会报错。

---

## 10.12 递归

递归函数通常是一个函数通过名称**调用自身**的情况。推荐使用**命名函数表达式**来编写递归，避免函数名被修改后出错：

```jsx
const factorial = (function f(num) {
  if (num <= 1) return 1;
  return num * f(num - 1);
});
```

---

## 10.13 尾调用优化

ES6 规范新增了一项内存管理优化机制——**尾调用优化**（Tail Call Optimization）。当外部函数的返回值是一个内部函数的**直接调用**时，引擎可以**复用栈帧**，从而减少内存开销。

**满足尾调用优化的条件：**

- [x]  代码在**严格模式**下执行
- [x]  外部函数的返回值是对**尾调用函数的调用**
- [x]  尾调用函数返回后**不需要执行额外的逻辑**
- [x]  尾调用函数**不是引用外部函数作用域中自由变量的闭包**

```jsx
"use strict";

// ❌ 不是尾调用：返回后还有加法操作
function fib(n) {
  if (n < 2) return n;
  return fib(n - 1) + fib(n - 2);
}

// ✅ 尾调用优化版本
function fib(n, a = 0, b = 1) {
  if (n === 0) return a;
  return fib(n - 1, b, a + b);
}
```

---

## 10.14 闭包

**闭包**指的是那些引用了另一个函数作用域中变量的函数，通常是在**嵌套函数**中实现的。

```jsx
function createComparisonFunction(propertyName) {
  return function(obj1, obj2) {
    // 这里引用了外部函数的变量 propertyName —— 形成闭包
    let val1 = obj1[propertyName];
    let val2 = obj2[propertyName];
    return val1 < val2 ? -1 : val1 > val2 ? 1 : 0;
  };
}
```

### 闭包与作用域链

每个函数执行时都会创建一个**执行上下文**，上下文中有一个**作用域链**：

1. 函数被调用时，创建执行上下文和作用域链
2. 用 `arguments` 和其他命名参数来初始化函数的**活动对象**
3. 外部函数的活动对象是内部函数作用域链上的**第二个对象**

> 闭包会保留对外部函数活动对象的引用，因此**外部函数执行完毕后其活动对象不会被销毁**，直到闭包被销毁。
> 

### 10.14.1 this 对象

闭包中的 `this` 可能会让人意外——**每个函数在被调用时都会自动获取 `this` 和 `arguments`**，内部函数永远不会直接访问外部函数的 `this`：

```jsx
window.identity = 'The Window';

let object = {
  identity: 'My Object',
  getIdentityFunc() {
    return function() {
      return this.identity;
    };
  }
};

console.log(object.getIdentityFunc()()); // "The Window"
```

**解决方案**：在外部函数中将 `this` 保存到变量中：

```jsx
let object = {
  identity: 'My Object',
  getIdentityFunc() {
    let that = this;
    return function() {
      return that.identity;
    };
  }
};

console.log(object.getIdentityFunc()()); // "My Object"
```

### 10.14.2 内存泄漏

闭包可能导致**内存泄漏**，特别是在引用 HTML 元素时：

```jsx
function assignHandler() {
  let element = document.getElementById('someElement');

  // 闭包引用了 element，导致其内存无法回收
  element.onclick = () => console.log(element.id);
}

// 改进：消除循环引用
function assignHandler() {
  let element = document.getElementById('someElement');
  let id = element.id;

  element.onclick = () => console.log(id);
  element = null; // 解除引用
}
```

---

## 10.15 立即调用的函数表达式（IIFE）

**IIFE**（Immediately Invoked Function Expression）是一种立即定义并执行的函数：

```jsx
(function() {
  // 块级作用域
  let count = 0;
  console.log(count); // 0
})();

// count 在外部不可访问
```

IIFE 的常见用途包括：

- 在 ES6 之前**模拟块级作用域**
- 锁定变量值（如在循环中创建闭包）

```jsx
// ES5 时代经典的 IIFE 用法
for (var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(() => console.log(j), j * 1000);
  })(i);
}

// ES6 使用 let 即可
for (let i = 0; i < 5; i++) {
  setTimeout(() => console.log(i), i * 1000);
}
```

> ES6 之后，IIFE 的使用场景大大减少，因为 `let` 和 `const` 提供了块级作用域。
> 

---

## 10.16 私有变量

严格来讲，JavaScript 没有私有成员的概念，但函数中定义的变量在函数外部无法访问。**私有变量**包括函数参数、局部变量以及函数内部定义的其他函数。

利用闭包可以创建**特权方法**（privileged method）来访问私有变量：

```jsx
function Person(name) {
  // 特权方法
  this.getName = function() {
    return name;
  };
  this.setName = function(value) {
    name = value;
  };
}

let person = new Person('Nicholas');
console.log(person.getName()); // "Nicholas"
person.setName('Greg');
console.log(person.getName()); // "Greg"
```

### 10.16.1 静态私有变量

通过在**私有作用域**中定义私有变量和函数，可以创建所有实例共享的静态私有变量：

```jsx
(function() {
  let privateVariable = 10;

  function privateFunction() {
    return false;
  }

  // 构造函数（注意没有 let/var/const，挂到全局）
  MyObject = function() {};

  MyObject.prototype.publicMethod = function() {
    privateVariable++;
    return privateFunction();
  };
})();
```

### 10.16.2 模块模式

**模块模式**在单例对象的基础上加以扩展，通过作用域链实现对私有变量的访问：

```jsx
let singleton = function() {
  let privateVariable = 10;

  function privateFunction() {
    return false;
  }

  return {
    publicProperty: true,
    publicMethod() {
      privateVariable++;
      return privateFunction();
    }
  };
}();
```

### 10.16.3 模块增强模式

适用于单例对象**必须是某个特定类型的实例**的场景：

```jsx
let singleton = function() {
  let privateVariable = 10;

  let object = new CustomType();

  object.publicProperty = true;
  object.publicMethod = function() {
    privateVariable++;
    return privateVariable;
  };

  return object;
}();
```

---

## 小结

<aside>
💡

**函数**是 JavaScript 编程中最有用也最通用的工具。本章关键要点：

- 函数表达式与函数声明不同：函数声明会提升，函数表达式不会
- ES6 新增了**箭头函数**，语法简洁但有很多限制（无 `this` 绑定、不能做构造函数等）
- **默认参数值**、**扩展操作符**和**剩余参数**让函数定义和调用更灵活
- **闭包**的核心在于作用域链：内部函数可以访问外部函数的变量，即使外部函数已经返回
- **IIFE** 可以模拟块级作用域（ES6 之前），锁定变量值
- **尾调用优化**可以在严格模式下优化递归函数的内存使用
- 利用闭包可以实现**私有变量**与**特权方法**
</aside>