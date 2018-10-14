---
title: prototype
---

# 原型

### `[[Prototype]]`
JavaScript 中的对象有一个特殊 `[[prototype]]` 内置属性，其实就是对其他对象的引用。几乎所有对象在创建时 `[[prototype]]` 属性都会被赋予一个非空的值。


```
var myObject = { 
    a: 2
}
myObject.a // 2
```
note：当你试图引用对象的属性时会触发默认 `[[Get]]` 操作，第一步是检查对象本身是否拥有这个属性，如果有那就使用它


```
var anotherObject = {
    a: 2
}

var myObject = Object.create(anotherObject)

myObject.a // 2
```
note：但如果 `[[Get]]` 操作无法在对象属性找到需要的属性，那就会继续访问对象的 `[[Prototype]]` 链。

```
var anotherObject = {
    a: 2
}

var myObject = Object.create(aotherObject)

for(var key in myObject) {
    console.log(key, myObject[key])
}
// a: 2

('a' in myObject) // true
```
note： `for .. in` 遍历对象时原理和查找 `[[Prototype]]` 链类似，任何可以通过原型链访问到的属性都被枚举。

note2： `in` 操作符来检查属性在对象中是否存在时，同样会查找对象的整条原型链。


##### Object.prototype
所有普通 `[[prototype]]` 链最终都会指向内置的 `Object.prototuype` 。由于所有 "普通" 对象都 "源于" `Object.prototype` 对象，所它包含 Javascript 中许多通用的功能。

##### 属性设置和屏蔽
给对象设置属性并不仅仅是添加一个新属性或者修改已有的属性值。

- 当属性不是直接存在于对象中

`[[prototype]]` 链就会被遍历，类似 `[[Get]]` 操作。如果原型上找不到需要的属性，那么属性会被直接添加至对象中

- 当属性即存在于对象中，也存在于原型链上层

那么属性会屏蔽，原型链上层的所有对应的属性。因为对象的属性总是会选择原型链中最底层的属性。

- 当属性不存在直接对象中，而是原型链上层中

note：原型链上层存在属性且普通数据访问属性（writable: true），那么直接在对象中添加一个新的属性（屏蔽属性）

note2：原型链上层存在属性但（writable: false），那么无法修改已有属性或在 直接对象中创建屏蔽属性并且静默失败。如果在严格模式下将会抛出错误

note3：原型链上层存在属性且拥有 setter 那么一定会调用 setter ， 属性不会添加至 对象中，也不会重新定义这个属性的 setter

### "类"
Javascript 才是真正应该被称为 面对对象 的语言， 因为它是少有的可以不通过类，直接创建对象的语言。在 Javascript 中，类无法描述对象的行为，对象直接定义自己的行为。Javascript 中只有对象。

##### "类" 函数
在 JavaScript 中有一种奇怪的行为一直被无耻的滥用，就是模仿类。


```
function Foo() {
    // ..
}

Foo.prototype // {}
```
note：所有的函数默认都会拥有一个名为 `prototype` 的公有且不可枚举的属性，它会指向另一个对象。这个对象通常被称为 `Foo` 的原型，通过 `Foo.prototype` 属性引用来访问它。 

```
function Foo() {
    // ..
}
var a = new Foo()

Object.getPrototypeOf(a) ==== Foo.prototype // true
```
note：在这个对象被用 `new Foo()` 时创建时，最后会被关联到这个 `Foo.prototype` 对象上
 
##### "构造函数"
```
function Foo() {
    // ..
}

Foo.prototype.constructor === Foo //true

var a = new Foo()
a.constructor === Foo // true
```
note：`Foo.prototype` 默认第一行声明中有一个公开并且不可枚举的属性 `constructor` ，该属性引用的是对象关联的函数。此外看到通过 "构造函数" 调用 `new Foo()` 创建的对象也有一个 `constructor` 属性指向 "创建这个对象的函数"。

note2：看似 `a.constructor === Foo` a确实有一个指向 `Foo.constructor` 属性，但是事实不是这样，这是一个误解，实际上 `constructor` 属性引用同样被委托给 `Foo.prototype` 而 `Foo.prototype.constructor` 默认指向 `Foo`


```
function NothingSpecial() {
    console.log('Don't mind me!')
}

var a = new NothingSpecial() // Don't mind me!

a // {}
```
note：函数不是构造函数，但是当且仅当使用 new 时，函数调用会变成 "构造函数调用" 。使用 new 调用时，它就会构造一个对象并赋值给 "它"。

##### 技术?
```
function Foo(name) {
    this.name = name
}

Foo.prototype.myName = function () {
    return this.name
}

var a = new Foo('a')
var b = new Foo('b')

a.myName() // a
b.myName() // b
```
note：在创建过程中，a b 的内部 `[[prototype]]` 都会被关联至 `Foo.prototype` 上。当a b 无法找到 myName 时， 通过委托在 `Foo.prototype` 上查找。

```
function Foo() {}

Foo.prototype = {}

var a = new Foo()

a.constructor === Foo // false
a.constructor === Object // true
```

note： `Foo.prototytpe.constructor` 属性只是 `Foo` 函数现在声明时的默认属性，如果再创建一个新对象并代替函数默认引用 `prototype` 对象引用。那么新对象并不会自动获取 `constructor` 属性。

note2： `a.construcotr` 其实并没有这个属性，那么它会委托  `[[prototype]]` 链上的 `Foo.prototype` 查找这个属性，这个对象也没有那继续委托链顶端的 `Obejct.prototype` ，那么这个对象 `constructor` 属性指向内置 `Object` 函数。


```
function Foo() {}

Foo.prototype = {}

Object.defineProperty(Foo.prototype, 'constructor', {
    enumerable: false,
    writable: true,
    configurable: true,
    value: Foo // 指向 Foo
})

var a = new Foo()
a.constructor === Foo // true
```
note： `constructor` 属性指向 `Foo` 看作 `a` 对象由 `Foo` "构造" 的误解。这是一种虚假的安全感。`a.constructor` 只是通过默认 `[[Get]]` 算法查找 `[[prototype]]` 链委托指向 `Foo` ，和 "构造" 毫无关系。

### 原型 继承
- Foo.prototype
    - a1
    - a2
    - Bar.prototype
        - b1
        - b2


##### 原型风格
```
function Foo(name) {
    this.name = name
}
Foo.prototype.myName = function () {
    return this.name
}

function Bar(name, age) {
    Foo.call(this, name)
    this.age = age
}
// 创建一个新对象Bar.prototytpe 并关联Foo.prototype
Bar.prototype = Object.create(Foo.prototype)
// constructor 手动修复指向 Bar
Bar.prototype.constructor = Bar

Bar.prototype.myAge = function () {
    return this.age
}

var a = new Bar('huang', 24)

a.myName() // huang
a.myAge() // 24
```

### 第一种

```
Bar.prototytpe = Foo.prototype
```

- 优点：简单
- 缺点：子类修改影响父类

note：并不会创建一个关联 `Bar.prototype` 的新对象，它直接是 `Bar.prototype` 引用 `Foo.prototype` 对象

```
Bar.prototype = new Foo()
```

- 优点：简单
- 缺点：父类存在副作用（比如状态），将影响子类的实例

note：构造函数调用，会创建一个新的对象并关联 `Bar.prototype`

### 第二种

- 优点：子类即能继承父类，由基本不影响父类，达到真正意义上的继承
- 缺点：实例对象 和 父类原型对象 相同的时候，可能会出现子类修改父类对象原型中的所有属性被实例共享，共享适合函数，基本类型值。但是对引用类型的值的属性会发生共享。

##### ES3
```
function create(proto) {
    function F() {}
    F.prototype = proto
    return new F()
}
Bar.prototype = create(Foo.prototype)
```

note： 创建一个新 `F` 对象 `F.prototype` 关联 `proto`，通过 "构造函数调用" 并且返回一个对象给 `Bar.prototype`。

##### ES5
```
Bar.prototype = Object.create(Foo.prototype)
```

note：需要创建一个新对象然后把 旧对象 抛弃掉，不能直接修改已有的默认对象。

##### ES6
```
Obejct.setPrototypeof(Bar.prototype, Foo.prototype)
```

note：ES6辅助函数，可以用标准并且可靠的方法修改关联。

```
class Foo () {
    constructor (name, age) {
        this.name
        this.age
    }
}

Foo.prototype.getName = function () {
    return this.name
}
Foo.prototype.getAge = function () {
    return this.age
}

Foo.prototype.obj = {
    lastName: 'enqiang'
}

class Bar extends Foo {
    constructor (...arguments) {
        super(...arguments)
    }
}
```

note：class 语法糖

### 检查 "类" 关系
假设对象 `a` ，寻找对象 `a` 委托的对象？ 在传统的面向类环境中，检测一个实例（JavaScript中的对象）的继承祖先（JavaScript中的委托关联），通常称为 内省（反射）

##### 第一种站在 "类" 的角度来判断

```
function Foo() {}

var a = new Foo()

a instanceof Foo // true
```

note：`instanceof` 操作符的左操作数是一个普通的对象，右操作数是一个函数。在 `a` 的整条 `[[prototype]]` 链中是否有指向 `Foo.prototype` 的对象？


##### 第二种判断 `[[prototype]]`  反射的方法

```
Foo.prototype.isPrototypeOf(a) // true
```

note： 实际上并不关系 `Foo`， 我们只需一个可以用来判断的的对象（例子中是 `Foo.protytpe` ) 就行，`a` 对象的整条 `[[prototype]] ` 链中是否出现过 `Foo.prototype` ?

```
b.isPrototypeOf(c)
```

note：该方法并不需要函数，可以直接判断两个对象 `b` 和 `c` 之间的对象引用的关系。


##### 非标准方法

````
a.__proto__ === Foo.prototype // true
````

note：`__proto__` 属性 ES6并不是标准，但可以引用内部 `[[prototype]]` 对象，甚至你可以通过遍历它来查找原型链。

```
Object.defineProperty(Object.prototype, '__proto__', {
    get: function () {
        return Object.getPrototypeOf(this)
    }
    set: function (proto) {
        Object.setPrototypeOf(proto)
        return proto
    }
})
```

note：`__proto__` 属性实际上并不存在真正使用的对象中，`__proto__` 与其他函数（`toSting()` `isPrototypeOf()`）一样存在与内置 `Object.prototype`（不可枚举）。

##### ES5

```
Object.isPrototypeOf(a)
```

note：判断指定对象是否在本对象的原型链中。

```
Object.getPrototypeOf(a)
```

note：获取指定对象的原型对象

```
Object.setPrototyeOf(a)
```

note：设置指定对象的原型

