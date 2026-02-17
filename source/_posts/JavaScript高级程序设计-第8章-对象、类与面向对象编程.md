---
title: 第8章 对象、类与面向对象编程
date: 2026-02-17 15:02:02
categories:
- [编程技术, JavaScript]
tags:
- JavaScript高级程序设计
---
## 8.1 理解对象

### 8.1.1 属性的类型

ECMA-262 定义了两种属性：**数据属性**和**访问器属性**。

### 数据属性

数据属性有 4 个特性（attribute）：

- `[[Configurable]]`：是否可以通过 `delete` 删除并重新定义、是否可以修改特性、是否可以改为访问器属性。默认 `true`
- `[[Enumerable]]`：是否可以通过 `for-in` 循环返回。默认 `true`
- `[[Writable]]`：属性的值是否可以被修改。默认 `true`
- `[[Value]]`：属性实际的值。默认 `undefined`

要修改属性的默认特性，需使用 `Object.defineProperty()`：

```jsx
let person = {};
Object.defineProperty(person, "name", {
  writable: false,
  value: "Nicholas"
});
console.log(person.name); // "Nicholas"
person.name = "Greg";
console.log(person.name); // "Nicholas"（严格模式下会报错）
```

<aside>
⚠️

一旦把 `configurable` 设置为 `false`，就不能再变回 `true`，并且除 `writable` 外的其他特性都不可修改。

</aside>

### 访问器属性

访问器属性不包含数据值，包含 `getter` 和 `setter` 函数（不是必需的）。有 4 个特性：

- `[[Configurable]]`：同数据属性。默认 `true`
- `[[Enumerable]]`：同数据属性。默认 `true`
- `[[Get]]`：获取函数，读取属性时调用。默认 `undefined`
- `[[Set]]`：设置函数，写入属性时调用。默认 `undefined`

```jsx
let book = {
  year_: 2017,   // 下划线常表示该属性不希望在外部被访问
  edition: 1
};
Object.defineProperty(book, "year", {
  get() { return this.year_; },
  set(newValue) {
    if (newValue > 2017) {
      this.year_ = newValue;
      this.edition += newValue - 2017;
    }
  }
});
book.year = 2018;
console.log(book.edition); // 2
```

### 8.1.2 定义多个属性

`Object.defineProperties()` 可以一次性定义多个属性：

```jsx
let book = {};
Object.defineProperties(book, {
  year_: { value: 2017 },
  edition: { value: 1 },
  year: {
    get() { return this.year_; },
    set(newValue) {
      if (newValue > 2017) {
        this.year_ = newValue;
        this.edition += newValue - 2017;
      }
    }
  }
});
```

### 8.1.3 读取属性的特性

使用 `Object.getOwnPropertyDescriptor()` 获取指定属性的属性描述符：

```jsx
let descriptor = Object.getOwnPropertyDescriptor(book, "year_");
console.log(descriptor.value);        // 2017
console.log(descriptor.configurable); // false（defineProperties默认）
```

ES2017 新增 `Object.getOwnPropertyDescriptors()`，获取**所有**自有属性的描述符。

### 8.1.4 合并对象

`Object.assign()` 将源对象的**可枚举自有属性**（含 Symbol）浅复制到目标对象：

```jsx
let dest = { id: "dest" };
let result = Object.assign(dest, { a: "foo" }, { b: "bar" });
console.log(result); // { id: "dest", a: "foo", b: "bar" }
console.log(dest === result); // true
```

<aside>
💡

`Object.assign()` 实际上是**浅复制**，对每个源属性执行的是 `=` 赋值。如果赋值期间出错，操作会中止并退出，但已完成的赋值不会回滚（**尽 "最大努力" 方法**）。

</aside>

### 8.1.5 对象标识及相等判定

`Object.is()` 解决了 `===` 的一些边界情况：

```jsx
// === 的问题
console.log(+0 === -0);   // true
console.log(NaN === NaN); // false

// Object.is() 的修正
console.log(Object.is(+0, -0));   // false
console.log(Object.is(NaN, NaN)); // true
```

### 8.1.6 增强的对象语法（ES6）

- 属性值简写
    
    ```jsx
    let name = "Matt";
    let person = { name }; // 等价于 { name: name }
    ```
    
- 可计算属性
    
    ```jsx
    const nameKey = "name";
    let person = { [nameKey]: "Matt" };
    ```
    
- 简写方法名
    
    ```jsx
    let person = {
      sayName(name) {
        console.log(`My name is ${name}`);
      }
    };
    ```
    

### 8.1.7 对象解构

```jsx
let person = { name: "Matt", age: 27 };
let { name: personName, age: personAge } = person;
console.log(personName); // "Matt"
console.log(personAge);  // 27
```

- 嵌套解构与部分解构
    
    ```jsx
    let person = { name: "Matt", age: 27, job: { title: "Engineer" } };
    let { job: { title } } = person;
    console.log(title); // "Engineer"
    ```
    
- 参数上下文匹配
    
    ```jsx
    function printPerson({ name, age }) {
      console.log(name, age);
    }
    printPerson({ name: "Matt", age: 27 });
    ```
    

---

## 8.2 创建对象

### 8.2.1 概述

虽然 `Object` 构造函数和对象字面量可以创建单个对象，但用同一个接口创建多个对象时会产生大量**重复代码**。

### 8.2.2 工厂模式

```jsx
function createPerson(name, age, job) {
  let o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function() { console.log(this.name); };
  return o;
}
let person1 = createPerson("Nicholas", 29, "SE");
```

<aside>
⚠️

工厂模式解决了创建多个相似对象的问题，但**没有解决对象标识问题**（即新创建的对象是什么类型）。

</aside>

### 8.2.3 构造函数模式

```jsx
function Person(name, age, job) {
  this.name = name;
  this.age = age;
  this.job = job;
  this.sayName = function() { console.log(this.name); };
}
let person1 = new Person("Nicholas", 29, "SE");
let person2 = new Person("Greg", 27, "Doctor");
```

使用 `new` 操作符调用构造函数会执行以下操作：

1. 在内存中创建一个新对象
2. 新对象内部的 `[[Prototype]]` 被赋值为构造函数的 `prototype` 属性
3. 构造函数内部的 `this` 被赋值为这个新对象
4. 执行构造函数内部的代码
5. 如果构造函数返回非空对象，则返回该对象；否则返回刚创建的新对象

```jsx
console.log(person1.constructor === Person); // true
console.log(person1 instanceof Person);      // true
```

<aside>
⚠️

**构造函数的问题**：其定义的方法会在每个实例上都创建一遍，导致功能相同的函数在不同实例上不相等（`person1.sayName !== person2.sayName`）。

</aside>

### 8.2.4 原型模式

每个函数都会创建一个 `prototype` 属性，它是一个对象，包含由特定引用类型的实例**共享的属性和方法**。

```jsx
function Person() {}
Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.sayName = function() { console.log(this.name); };

let person1 = new Person();
let person2 = new Person();
console.log(person1.sayName === person2.sayName); // true ✅
```

### 理解原型

三个关键关系：

- `Person.prototype.constructor === Person`
- `person1.__proto__ === Person.prototype`
- `Object.getPrototypeOf(person1) === Person.prototype`

```jsx
// 检测原型关系
console.log(Person.prototype.isPrototypeOf(person1)); // true
// 获取原型
console.log(Object.getPrototypeOf(person1) === Person.prototype); // true
// 设置原型（性能差，不推荐）
Object.setPrototypeOf(obj, proto);
// 推荐：使用 Object.create() 创建具有指定原型的新对象
let biped = { numLegs: 2 };
let person = Object.create(biped);
```

### 原型层级与属性遮蔽（Shadowing）

当在实例上添加与原型同名的属性时，实例属性会**遮蔽**原型属性：

```jsx
person1.name = "Greg";
console.log(person1.name); // "Greg" —— 来自实例
console.log(person2.name); // "Nicholas" —— 来自原型

delete person1.name;
console.log(person1.name); // "Nicholas" —— 恢复为原型上的值
```

- `hasOwnProperty()`：检测属性是否在**实例**上
- `in` 操作符：检测属性是否在实例**或**原型上
- `Object.keys()`：返回实例上所有**可枚举**的自有属性
- `Object.getOwnPropertyNames()`：返回所有自有属性（含不可枚举）
- `Object.getOwnPropertySymbols()`：返回所有符号属性

### 对象迭代（ES2017+）

- `Object.values()`：返回对象值的数组
- `Object.entries()`：返回键/值对的数组

```jsx
const o = { foo: "bar", baz: 1, qux: {} };
console.log(Object.values(o)); // ["bar", 1, {}]
console.log(Object.entries(o)); // [["foo","bar"],["baz",1],["qux",{}]]
```

<aside>
💡

这两个方法执行的是**浅复制**，符号属性会被忽略。

</aside>

---

## 8.3 继承

### 8.3.1 原型链

原型链是 ECMAScript 的主要继承方式。基本思想：让一个引用类型的原型等于另一个引用类型的实例。

```jsx
function SuperType() {
  this.property = true;
}
SuperType.prototype.getSuperValue = function() {
  return this.property;
};

function SubType() {
  this.subproperty = false;
}
SubType.prototype = new SuperType(); // 关键：原型被重写

SubType.prototype.getSubValue = function() {
  return this.subproperty;
};

let instance = new SubType();
console.log(instance.getSuperValue()); // true
```

<aside>
⚠️

**原型链的两个问题：**

1. 原型中包含的引用值会在所有实例间共享
2. 子类型在实例化时不能给父类型的构造函数传参
</aside>

### 8.3.2 盗用构造函数（经典继承）

在子类构造函数中调用父类构造函数：

```jsx
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
function SubType(name) {
  SuperType.call(this, name); // 继承 SuperType
}
let instance1 = new SubType("Nicholas");
instance1.colors.push("black");
console.log(instance1.colors); // ["red","blue","green","black"]

let instance2 = new SubType("Greg");
console.log(instance2.colors); // ["red","blue","green"] ✅ 不共享
```

<aside>
⚠️

**盗用构造函数的问题**：方法必须在构造函数中定义，无法重用；子类也不能访问父类原型上定义的方法。

</aside>

### 8.3.3 组合继承（伪经典继承）⭐

综合了原型链和盗用构造函数的优点，是 JavaScript 中**使用最多的继承模式**：

```jsx
function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  console.log(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);   // 第二次调用 SuperType()
  this.age = age;
}
SubType.prototype = new SuperType();  // 第一次调用 SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
  console.log(this.age);
};

let instance = new SubType("Nicholas", 29);
instance.colors.push("black");
instance.sayName(); // "Nicholas"
instance.sayAge();  // 29
```

### 8.3.4 原型式继承

```jsx
// ES5 规范化为 Object.create()
let person = { name: "Nicholas", friends: ["Shelby", "Court"] };
let anotherPerson = Object.create(person, {
  name: { value: "Greg" }
});
```

### 8.3.5 寄生式继承

创建一个仅用于封装继承过程的函数：

```jsx
function createAnother(original) {
  let clone = Object.create(original);
  clone.sayHi = function() { console.log("hi"); };
  return clone;
}
```

### 8.3.6 寄生式组合继承 ⭐⭐

**最佳的引用类型继承范式**，解决了组合继承调用两次父构造函数的效率问题：

```jsx
function inheritPrototype(subType, superType) {
  let prototype = Object.create(superType.prototype); // 创建对象
  prototype.constructor = subType;                     // 增强对象
  subType.prototype = prototype;                       // 赋值对象
}

function SuperType(name) {
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  console.log(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);
  this.age = age;
}
inheritPrototype(SubType, SuperType);
SubType.prototype.sayAge = function() {
  console.log(this.age);
};
```

<aside>
✅

寄生式组合继承只调用了**一次** `SuperType` 构造函数，避免了在 `SubType.prototype` 上创建不必要的多余属性，原型链保持不变，`instanceof` 和 `isPrototypeOf()` 正常工作。

</aside>

---

## 8.4 类（ES6）

ES6 的 `class` 关键字是基于原型继承的**语法糖**，背后仍然使用的是原型和构造函数的概念。

### 8.4.1 类定义

```jsx
// 类声明
class Person {}

// 类表达式
const Animal = class {};
```

<aside>
💡

**与函数声明的重要区别**：函数声明可以提升（hoisting），但**类声明不能提升**。

</aside>

类可以包含以下内容：

- 构造函数方法（`constructor`）
- 实例方法
- 获取函数（`get`）和设置函数（`set`）
- 静态类方法（`static`）

```jsx
class Person {
  constructor(name) {
    // 添加到 this 的内容会存在于不同的实例上
    this.name = name;
    this.locate = () => console.log("instance");
  }
  // 定义在类的原型对象上
  locate() {
    console.log("prototype");
  }
  // 定义在类本身上
  static locate() {
    console.log("class", this);
  }
}
```

### 8.4.2 类构造函数

- `constructor` 关键字
    - 使用 `new` 调用类时会执行 `constructor` 函数
    - 不定义 `constructor` 相当于将其定义为空函数
    - 类实例化时传入的参数会用作 `constructor` 的参数
    - 如果 `constructor` 返回的不是 `this` 对象而是其他对象，`instanceof` 检测将不符合预期
- 类的本质
    
    ```jsx
    class Person {}
    console.log(typeof Person); // "function"
    console.log(Person.prototype.constructor === Person); // true
    ```
    
    类本身就是一个特殊的函数，具有 `prototype` 属性，而原型也有 `constructor` 属性指回类自身。
    

### 8.4.3 实例、原型和类成员

- 实例成员
    
    在 `constructor` 中通过 `this` 添加的属性，每个实例独有。
    
- 原型方法和访问器
    
    ```jsx
    class Person {
      get name() { return this.name_; }
      set name(newName) { this.name_ = newName; }
      sayName() { console.log(this.name_); }
    }
    ```
    
- 静态方法
    
    ```jsx
    class Person {
      static create() {
        return new Person();
      }
    }
    ```
    
    静态方法中的 `this` 引用类自身，适合作为工厂函数或实例化之外的操作。
    
- 非函数原型和类成员
    
    类定义中不支持直接添加**数据属性**，但可以在类定义外手动添加：
    
    ```jsx
    Person.greeting = "My name is";
    Person.prototype.defaultName = "Unknown";
    ```
    
- 迭代器与生成器方法
    
    ```jsx
    class NTimes {
      constructor(max) { this.max = max; }
      *[Symbol.iterator]() {
        for (let i = 0; i < this.max; i++) yield i;
      }
    }
    for (let x of new NTimes(3)) console.log(x); // 0, 1, 2
    ```
    

### 8.4.4 继承

ES6 类原生支持继承，使用 `extends` 关键字。

- 基本继承
    
    ```jsx
    class Vehicle {
      identifyPrototype(id) { console.log(id, this); }
    }
    class Bus extends Vehicle {}
    let bus = new Bus();
    bus.identifyPrototype("bus"); // "bus", Bus {}
    ```
    
- `super` 关键字
    - 在**构造函数**中调用 `super()` 会调用父类构造函数
    - 在**静态方法**中可以通过 `super` 调用父类上定义的静态方法
    - 派生类如果定义了 `constructor`，则**必须**在其中调用 `super()`，或者返回一个其他对象
    
    ```jsx
    class Vehicle {
      constructor(licensePlate) {
        this.licensePlate = licensePlate;
      }
    }
    class Bus extends Vehicle {
      constructor(licensePlate) {
        super(licensePlate); // 必须在使用 this 之前调用
        this.hasBigWindows = true;
      }
    }
    ```
    
    <aside>
    ⚠️
    
    **`super` 使用注意事项：**
    
    - `super` 只能在派生类的构造函数和静态方法中使用
    - 不能单独引用 `super`，要么调用构造函数 `super()`，要么引用静态方法 `super.method()`
    - 调用 `super()` 会调用父类构造函数，并将返回的实例赋值给 `this`
    - 在调用 `super()` 之前不能在构造函数中引用 `this`
    </aside>
    
- 抽象基类
    
    JS 没有专门的抽象类语法，但可以通过 `new.target` 实现：
    
    ```jsx
    class Vehicle {
      constructor() {
        if (new.target === Vehicle) {
          throw new Error("Vehicle cannot be directly instantiated");
        }
        if (!this.foo) {
          throw new Error("Inheriting class must define foo()");
        }
      }
    }
    ```
    
- 继承内置类型
    
    ```jsx
    class SuperArray extends Array {
      shuffle() {
        for (let i = this.length - 1; i > 0; i--) {
          const j = Math.floor(Math.random() * (i + 1));
          [this[i], this[j]] = [this[j], this[i]];
        }
      }
    }
    let a = new SuperArray(1, 2, 3, 4, 5);
    a.shuffle();
    ```
    
- 类混入（Mixin）
    
    `extends` 后面可以是任意表达式，利用这一点可以实现多继承的效果：
    
    ```jsx
    let SerializableMixin = (Superclass) => class extends Superclass {
      serialize() { return JSON.stringify(this); }
    };
    let AreaMixin = (Superclass) => class extends Superclass {
      getArea() { return this.length * this.width; }
    };
    
    class Square extends AreaMixin(SerializableMixin(Object)) {
      constructor(length) {
        super();
        this.length = length;
        this.width = length;
      }
    }
    let s = new Square(5);
    console.log(s.getArea());   // 25
    console.log(s.serialize()); // {"length":5,"width":5}
    ```
    

---

## 8.5 本章小结

<aside>
📌

**核心要点回顾：**

- 对象属性分为**数据属性**和**访问器属性**，可通过 `Object.defineProperty()` 精确定义
- 创建对象的模式从**工厂模式** → **构造函数模式** → **原型模式**逐步演进
- 继承模式从**原型链** → **盗用构造函数** → **组合继承** → **寄生式组合继承**逐步优化
- ES6 的 `class` 是语法糖，底层仍是原型机制，但提供了更清晰的写法
- `extends` + `super` 让继承变得简洁，还支持继承内置类型和类混入
</aside>