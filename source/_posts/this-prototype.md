---
title: this-prototype
---

# this
- this 关键字是 JavaScript 中自己复杂的机制之一。它是一个特别的关键字，被自动定义在所有函数的作用域中
- this 提供了一种更优雅的方式隐式 **传递** 一个对象引用。可以将 API 设计得更加简洁便奇易于复用

## 使用 this 机制
```
function identify () {
    return this.name.toUpperCase()
}

function speak () {
    var greeting = "hello , I'm " + identify.call(this)
    return greeting
}`

var me = {
    name: 'Kyle'
}

var you = {
    name: 'Reader'
}

identify.call(me)
identify.call(you)

speak.call(me)
speak.call(you)
```

## 显式传入上下文对象
```
function identify (context) {
    return context.name.toUpperCase()
}

fucntion speak (context) {
    var greeting = "hellom I'm " + identify(context)
    return greeting
}

identify(you)
speak(me)
```

## 误解

### 1. this 指向自身

#### 函数 foo 被调用的次数？
count 属性和预期不一样， foo 函数调用 this.count 会在无意中创建了一个全局变量 count 值为 NaN
```
function foo (num) {
    console.log('foo: ' + num)
    this.count++
}

foo.count = 0

for(var i = 0; i < 10; i++) {
    if(i > 5) {
        foo(i)
    }
}
foo.count // 0
```
#### 强制 this 指向 foo 函数对象
```
function foo (num) {
    console.log('foo: ' + num)
    this.count++
}

foo.count = 0

for(var i = 0; i < 10; i++) {
    if(i > 5) {
        foo.call(foo, i)
    }
}
```

### 2. this 指向函数的作用域
在某种情况下它即使正确，但是在其他情况下它却是错误。前提明确的是 this 在任何情况下都不指向函数的词法作用域
```
function foo () {
    var a = 2
    this.bar()
}

function bar () {
    return this.a
}

foo() // ReferenceError: a is not defined
```
## this 到底是？
this 是在运行时进行绑定，this 的上下文取决于函数调用时各种条件。this 的绑定和函数声明的位置无任何关系。只取决于函数的调用方式

当函数被调用时
- 活动记录（执行上下文）
    - 调用（调用栈）
    - 函数的调用方式
    - 传入的参数

this 就是这个活动记录的一个属性（执行上下文）

## 调用位置

- 问： this 到底引用的是什么？
- 答：在理解 this 绑定过程之前，首先理解函数被调用位置（而不是函数声明的位置）

### 调用栈和调用位置
调用栈好比函数调用链环环相扣

```
function baz () {
    // 调用栈 全局 -> baz
    console.log('baz')
    bar()   // bar 调用位置
}
function bar () {
    // 调用栈 baz -> bar
    console.log('bar')
    foo()   // foo 调用位置
}
function foo () {
    // 调用栈 baz -> bar -> foo
    console.log('foo')
}

baz()   // baz 调用位置
```

## this 绑定规则
- 默认绑定
- 隐式绑定
- 显示绑定
- new 绑定
- 优先级
- 箭头函数

### 默认绑定
最常用的函数调用类型，独立函数调用。当其他规则无法应用时的默认规则

- 问：this 指向什么？
- 答：foo 调用位置在 window 全局且没有不带任何修饰的函数引用进行调用，所以只能使用默认规则 this 指向 window 全局对象，但严格模式（strict mode）则抛出和 TypeErro 错误， this is undefined

```
function foo () {
    return this.a
}
foo()
```

### 隐式绑定
调用的位置是否存在上下文对象，或者是否被某个对象拥有/包含

- 问：代码示例2 为什么输出 "huang"
- 答：对象属性引用链只有在上一层或最后一层的调用位置中起作用

```
// 代码示例1
function foo () {
    return this.a
}
var obj = {
    name: 'enqiang',
    foo: foo
}

obj.foo()
```

```
// 代码示例2
function foo () {

}
var lastObject = {
    name: 'huang',
    foo: foo
}

var firstObject = {
    name: 'enqiang',
    lastObject: lastObject
}

firstObject.lastObject.foo()
```


### 显示绑定

#### 显示绑定
`foo.call(this, arg1, arg2)` 和 `foo.apply(this, args)` Javascript 宿主环境提供的特殊函数， `call` 和 `apply` 区别体现在参数传入上

#### 硬绑定
`foo.bind(this)` 执行并回返回一个硬编码函数，它会把你指向的参数设置为 this 的上下文并并调用原始函数

#### API 调用的 上下文
通常 JavaScript 和宿主环境中许多新的内置函数，都会提供一个可选参数，通常称为 上下文（context），作用于 bind 一样，确保回调函数使用指定 this

### new 绑定
JavaScript 中的 构造函数 只是被 new 操作符调用的普通函数而已，并不会属于某个类，也不会实例化一个类。事实上它们甚至不能说是一个种特殊函数类型，只是被 new 操作符调用的普通函数

- 问：使用 new 操作符调用函数会做些什么操作
- 答：  1. 创建一个全新对象 2. 新对象会被链接到 Prototype 3.新对象会绑定到函数调用 this 4. 函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象


### 优先级
（new 绑定 + 硬绑定） > 显示绑定 > 隐式绑定 > 默认绑定

 - 问：函数是否被 new 调用？
 - 答： this 被绑定到新创建的对象
 

 - 问：函数是否通过 call， apply （显示绑定）或者硬绑定调用？
 - 答： this 被绑定到指定的对象
 

 - 问：函数是否在某个上下文对象中调用（隐式调用）？
 - 答： this 绑定到上下文对象
 

 - 问：如果都不是呢？
 - 答：使用默认绑定，懒惰模式绑定到全局 Window ，严格模式绑定到 undefined


找到函数的直接调用位置
1. 是否由 new 调用？ 绑定到新创建的对象
2. 是否由 call 、apply 、bind 调用？绑定到指定对象
3. 是否由上下文对象调用？绑定到上下文独享
4. 默认：是否在严格模式下，严格模式下绑定到 undefined ，懒惰模式绑定到全局对象 

#### 绑定特殊例外
在某些场景下 this 你认为应用其他绑定规则，实际上应用可能是默认绑定规则。比如把 `null` `undefined` 作为 this 的绑定对象传入 `call` `apply` `bind` 这些绑定会忽略并会使用默认绑定规则 

- 问：什么情况你会传入 `null` 呢？
- 答：一种常见的做法使用 `apply` 来展示一个数组，并当作参数传入一个函数。`bind` 可以对参数进行柯里化（预先设置一些参数）

```
function foo(a, b){
    return `a:${a}, b:${b}`
}
// 将数组展开成 参数
foo.apply(null, [2, 3]) a:2, b:3

var bar = foo.bind(null, 2)
bar(3) a:2, b:3
```

#### 更安全的this
`var ø = Object.create(null)` 由这个对象完全是一个空对象，不会对全局对象产生任何影响

- 问： `Object.create(null)` 和字面量 `{}` 有什么区别？
- 答： 两者都是创建一个空对象，但是 `Object.create(null)` 并不会创建 `Object.prototype` 这个委托，所以它比字面量 `{}` 更空

### 间接引用
有时可能无意间创建一个函数的 间接引用 在这种情况下，调用该函数会应用默认绑定规则


注意：对于默认绑定来说，决定 this 绑定对象并不是调用位置是否处于严格模式，而是函数体是否处于严格模式。如果函数体处于严格模式，this 绑定到 undefined，否则默认绑定规则 this 绑定到全局对象


赋值表达式`p.foo = o.foo` 的返回值是目标函数的引用，因此调用位置的是 `foo()` 而不是 `p.foo()` `o.foo()` 
```
function foo () {
    return this.a
}
var a = 2
var o = { a: 3, foo: foo }
var p = { a: 4 }

o.foo()  // 3
(p.foo = o.foo)() // 2
```

### 软绑定
硬绑定方式把 this 强制绑定到指定的对象（除了使用 new 时），防止函数调用应用默认绑定规则。使用硬绑定降低函数的灵活性，而无法使用隐式或显式绑定来修改 this

```
if(!Function.prototype.softBind) {
    Function.prototype.softBind = function (obj) {
        var fn = this
        // 捕获所有 curried 参数
        var curried = [].slice.call(arguments, 1)
        var bound = function () {
            return fn.apply(!this || this === (window || global) ? obj : this,
                curried.concat.apply(curried, arguments)
            )
        }
        bound.prototype = Object.create(fn.prototype)
        return bound
    }
}

function foo() {
    return `name: ${this.name}`
}

var obj = { name: 'obj' },
    obj2 = { name: 'obj2' },
    obj3 = { name: 'obj3' }

var fooOBJ = foo.softBind(obj)
fooOBJ() // name: obj

obj2.foo = foo.softBind(obj)
obj2.foo()  // name: ojb2

fooOBJ.call(obj3) //  name: obj3

setTimeout(obj2.foo, 10)  // name: obj 
```

### this 词法
ES6 箭头函数不使用 function 关键词定义，箭头函数不使用 this 的绑定规则，而是规矩外层作用域来决定 this


最常用于回调函数，例如事件处理器，定时器
```
function foo () {
    setTimeout(() => {
        return this.a
    })
}

var obj = { a: 2 }

foo.call(obj) // 2
```

# 对象

### 语法
字面量形式 和 构造形式
```
var obj =  {
    key:  value
}

var obj = new Object()
obj.key = value
```


### 类型 
- string
- number
- boolean
- null
- undefined
- symbol
- object

### 内置对象
- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

### 键值对
对象的内容（value）由一些存储在特定命名位置的值（key）组成的称为属性

#### value
内容（value）值实际上被存储在对象内部，引擎内部，这些值存储方式多种多样，一般不会存储在对象容器内部


#### key
属性（key）存储在对象容器的是这些属性的名称，指针（引用），指向这些值真正的存储位置

属性名永远都是字符串，如果使用 string （字面量）以外的其他值作为属性名，那么它将会被转换一个字符串

```
var obj = { a: 2 } // 在内存开辟一块空间（堆内存）

obj.a // 指针指向地址（栈内存）

obj['a']
```

- `.`操作符称为 属性访问
- `[]`操作符称为 键访问

区别`.`操作符要求属性名满足标识符的命名规范，而`[..]`语法可以接受任意 UTF-8/Unicode 字符串作为属性名
`[last-name]`必须使用`[]` 语法访问， `last-name` 并不是有效的标识符属性名


### 可计算属性名
ES6增加来可计算属性，可以在文字形式中使用 [prefix + name] 包裹一个表达式来作属性名。常用的常见可能是 ES6 的符号（Symbol）?

### 属性与方法
访问的对象属性是一个函数，使用不一样的叫法以作区分。由于函数很容易被认为是某个对象，属于对象的函数通常称为 方法， 因此把 属性访问 叫成 方法访问 也就不奇怪了

JavaScript 语法规范也作出了同样区别

从技术角度 函数永远不会 属于 一个对象，所以把对象内部引用的函数称为 方法 似乎不妥

有些函数具有 this 引用，有时这些 this 确实会指向调用位置的对象引用，但是这种用法没有本质并没有把一个函数变成一个 方法。 因为 this 是在运行时根据调用的位置动态绑定的，所有函数和对象的关系只能是 间接关系

最保险的说法可能是， 函数 和 方法在 JavaScript 中是可以互换的
```
functiion foo () {
    return 'foo'
}

var someFoo = foo

var myObject = {
    someFoo: foo
}

foo

someFoo 

myObject.someFoo
```

即使你在对象使用字面量形式声明一个函数表达式，这个函数也不 属于 这个对象，它们 只是对于相同的函数对象的多个引用
```
var myObject = {
    foo: function () {
        return 'foo'
    }
}

var someFoo = myObject.foo

someFoo

myObject.foo
```

### 数组
数组也是对象，虽然数组期望通过 索引 访问值的储存的位置，但你仍然可以给数组添加属性

完全可以将数组当作 键/值 对象使用，并不添加任何数值索引，但并不推荐这么做，所以最好只用对象存储 键/值 对，只用数组来存储数值 下标/值 对

### 浅拷贝深拷贝
这个问题都没有明确的答案


这种方法需要保证对象是 JSON 安全的，所以只适用于部分情况
```
var newObject = JSON.parse( JSON.stringify(somObject))
```

相比深拷贝，浅拷贝非常易懂并且问题要少得多，所以ES6定义 `Object.assign` 方法来实现浅复制，第一个参数是目标对象，之后跟参可以一个或多个源对象，它会遍历一个或多个源对象的所有 可枚举（enumerable） 的自有键（owned key）并把它们复制（使用 = 操作符赋值）到目标对象，最后返回目标对象
```
var newObject = Object.assign({}, myObject)
```
 
### 属性描述符
ES5之前，JavaScript 语言本身并没有提供可以直接检测属性特性的方法，比如判断属性是否只读？    


ES5开始，所有属性都具备来属性描述符（数据描述符）。它保存可不仅仅是一个数据值 2 ， 它还包含另外三个特性， writable（可写）、enumerable（可枚举）、configurable（可配置）
```
var myObject = {
    a: 2
}

Object.getOwnPropertyDesriptor(myObject, 'a')
```


在创建普通属性时属性描述符会使用默认值， 使用`Object.defineProperty()`方法来添加一个新属性或修改一个已有属性（如果它是configurable）并对特性进行设置
```
{
     value: 2,
     writable: true,
     enumerable: true,
     configurable: true,
}
```

- writable

决定是否可以修改属性的值。

`writable: false` 对于属性修改静默失败，但是在严格模式下抛出 TypeError

- configurable

只有属性是可配置的，就可以使用`defineProperty()`方法来修改属性描述符。

`configurable: false` 不管是否处于严格模式下都抛出 TypeError。单向操作，无法撤销

note: `configurable: false` 还是可以修改 `writable` 属性 `true` -> `false` , 但无法 `false` -> `true`。还会禁止删除该属性

- enumerable

控制属性是否会出现在对象的属性枚举中。例如 `for .. in` 循环

`enumerable: false` 该属性就不会出现枚举中。但仍可以正常访问它


##### 不变性

当希望属性或对象是不可改变的，ES5中可以通过很多中方法来实现。

note: 所有方法创建的都是浅不变性，它们只会影响目标对象和它的直接属性。但目标对象引用其他对象，其他对象的内容不受影响。仍是可变

- 对象常量

结合`writable: false` `configurablue: false` 就可以创建个真正的常量属性

- 禁止扩展

`Object.preventExtensions()` 方法禁止对象添加新属性且保留已有属性

note： 创建新属性会静默失败，在严格模式下抛出 TypeError

- 密封

`Obejct.seal()`方法会创建个 密封 的对象，但实际上该方法会在现有对象调用 `Object.preventExtensions()` 方法并把所有现有属性标记为 `configurable: false`

note: 密封 后 不仅不能添加新属性，也不能重新配置或修改任何现有属性（但可以修改属性的值）

- 冻结

`Object.freeze()` 方法会创建个 冻结 对象，但实际上该方法会在现有对象调用 `Object.seal()` 方法并把所有 数据访问 属性标记为 `writable: false` 

note：无法修改它们的值。该方法是可以应用在对象上级别最高的不可变性，它会禁止对于对象本身及其任意直接属性修改（但该对象引用的其他对象是不受影响）

##### [[Get]]
```
var myObject = {
    a: 2
}

myObject.a // 2
```
`myObject.a` 属性访问，但这条语句不仅仅是在 `myObject` 中查找名字为 `a` 的属性

note： 实际上实现来 `[[Get]]` 操作。对象默认的内置 `[[Get]]` 操作首先在对象中查找是否有名称相同的属性，找到就返回该属性的值。如果没有找到名称相同的属性，按照 `[[Get]]` 算法的定义会执行另外一中重要的行为（遍历可能存在的 `[[Prototype]]` 链）。最后都没有找到名称相同的属性， `[[Get]]` 操作会返回值 `undefiend`

##### [[Put]]

既然有可以获取属性值的 `[[Get]]` 的操作，对应有 `[[Put]]` 操作。

note: 触发`[[put]]` 时实际的行为取决许多因素。
1. 是否已经存在这个属性
    - 不存在 触发`[[Put]]` 来设置或创建这个属性
    - 存在
        - 属性是否是访问描述符
            - 是并且存在 `setter` 将调用 `setter`
        - 属性的数据描述 `writable`是否 `false` ?
            - 是静默失败，严格模式下抛出 TypeError
        - 如果以上都不是，将该值设置为属性的值

##### Gette和Setter
对象默认的 `[[Put]]` 和 `[[Get]]` 操作分别可以控制属性的设置和获取

getter 和 setter： 

- 默认操作且是隐藏函数
- getter 会在获取属性值调用。
- setter 会在设置属性值调用。
- 当给属性定义 getter 、 setter、 两者都有时。该属性被定义为 访问描述符 （和 数据描述符 相对）。
- 对于 访问描述符 会忽略 value 和 writable 
- 只关注 set 和 get （configurable 和 enumerable）特性

note： 只能应用在单个属性上，无法应用在整个对象上


对象字面量 `get a() {}`， 还是 `defineProperty()` 中显示定义。两者都会在对象创建一个不包含值的属性。对于这个属性的访问会自动调用隐藏函数，它的返回值会被当作属性访问的返回值。
```
var myObject = {
    get a() {
        return a;
    }
}

Object.defineProperty(myObject, "b" , {
    get() {
        return this.a * 2
    },
    enumerable: true
})

myObject.a // 2
myObject.b // 4
```

只定义 a 的 getter ， 所以对 a 的值进行设置时 set 操作会忽略赋值操作，不会抛出错误，而且即便有合法 setter， 由于只自定义 getter 返回值 2，所以 set 操作会被忽略没有意义的
```
var myObject = {
    get a () {
        return a
    }
}

myObject.a = 3 
myObject.a  // 2
```

为了让属性合理，应当定义 setter ， 覆盖单个属性默认操作，通过 getter 和 setter 是成对出现。否则会产生意料之外的行为
```
var myObject = {
    get a() {
        return this._a_
    }
    set a(val) {
        this._a_ = val * 2
    }
}
myObject.a = 2
myObject.a // 4
```


##### 存在性
例如 myObject.a 属性访问返回值可能是 undefined  ，但该值可能存储为 undefined， 也可能因为属性不存在则返回 undefined


`in` 操作符会检查属性是否在对象及 `[[prototype]]` 原型中，相比 `hasOwnProperty()` 方法只会检查属性是否在对象中，不会检查 `[[prototype]]` 链

note1： 所有对象可以通过 `Object.prototype` 委托来访问 `hasOwnProperty()` 方法，但如果对象没有链接到 `Obejct.prototype` （通过 `Object.create(null)` 来创建）的情况下，访问 `hanOwnProperty()` 方式会失败。


note2： `Object.prototype.hasOwnProperty.call(myObject, 'a')` 通过一种显示绑定借用方法进行判断。
```
var myObject = {
    a: 2
}

("a" in myObject) // true
("b" in myObject) // false

myObject.hasOwnProperty("a") // true
myObject.hasOwnProperty("b") // false
```

##### 枚举
可枚举 相当于 是否可以出现在对象属性的遍历中

```
var myObject = {}

Object.defineProperty(myObject, 'a', {
    value: 2,
    enumerable: true
})

ObjectdefineProperty(myObject, 'b', {
    value: 3,
    enumerable: false
})

myObject.b // 3
('b' in myObject) // true
myObject.hasOwnProperty('b') // true


for(var key in myObject) {
    console.log(key, myObject[key]) // a 2
}


myObject.propertyIsEnumerable('a') // true
myObject.propertyIsEnumerable('b') // false

Object.keys(myObject) // ['a']
Object.getOwnPropertyNames(myObject) //['a', 'b']
```

- `myObjcet.propertyIsEnumerable('a')`

判断指定属性是否枚举

- `Object.keys(myObject)`

返回一个数组，包含所有可枚举属性

- `Object.getOwnPropertyNames(myObject)`

返回一个数组，包含所有可枚举和不可枚举属性


### 遍历
回调函数中特殊的返回值和普通 `for` 循环中 `break` 语句类似，会提前终止遍历

- `for ...` 

- `for .. in`

可以遍历对象的可枚举属性列表包括 `[[prototype]]` 链。

note：无法直接获取属性值的，实际上遍历的对象中的所有可枚举属性，需要手动获取属性值

note2：遍历数组下标时采用的是数顺序，但是遍历对象属性时的顺序不确定，因此不要相信任何观察到的顺序，它们时不可靠。


*ES5新增迭代器*

- `for .. of`

能直接遍历数组值，而不是数组下标

note：如果对象本身定义来迭代器 `@@iterator` 的话也可以遍历对象

- `forEach()`

遍历数组所有值并忽略回调函数返回值

- `every()`

一直运行至回调函数返回 false

- `someO()`

一直运行值回调函数返回 true

# 混合对象  "类"
面向类的设计模式

- 实例化（instantiation）
- 继承（inheritance）
- 多态（polymorphism）

### 类
类/继承 描述类一种代码的组织结构形式，一种在软件对真实世界中问题领域的建模方法

面对对象变成强调的是数据和操作数据的行为本质上是互相关联（不同数据有不同行为），因此好的设计就是把数据以及和它相关的行为打包（封装），这在正式的计算机科学中有时称为数据结构。

- 栗子1

任意长度字符串通常被称为字符串，字符就是数据，但我们不关心数据是什么，而是可以对数据做什么？，所以应用在这数据的行为都被设计成 String 类的方法

所有字符串都是 String 类的一个实例，包含字符数据和我们可以应用在数据上的函数方法


- 栗子2

现实世界中 汽车 可以被看作 交通工具 的一种特例，后者是更广泛的类。

在软件中定义一个 Vehicle 类 Car 类来对这种关系进行建模，Vehicle 中定义来所有类型的交通工具都包含的行为，定义 Cart 时，只要声明它继承 Vehicle 这个基础类的定义就行，Cart 的定义就是对通用 Vehicle 定义的特殊化

note：Vehicle 和 Car 会定义相同的方法，但实例中的数据可能不同。例如车牌号。

- "类" 设计模式

可能没有把 类 作为设计模式来看待，但讨论最多得是面向对象设计模式 *迭代器模式* *观察者模式* *工厂模式* *单例模式* 等等。从这个角度来说？我们似乎是在（低级）面向对象类的基础上实现了所有（高级）设计模式？似乎面向对象是优秀代码的基础。
    
    面向过程化编程

    这种代码只包含过程（函数）调用，没有高层的抽象。

    如有有函数式编程的经验？会知道 类 也是一种非常常用的一种设计模式，不是必须的编程基础，而是一种可选的代码抽象。

有些语言（比如Java）并不会给你选择的机会，类并不是可选的 万物皆是类。

其他语言（比如C++/PHP）会提供过程化和面向类这两种语法，可选其一种或混合两种风格

- JavaScript中的 "类"

JavaScript 中实际上 没有 类，而是增加了一些近似 类 的语法元素（比如new instanceof class）。尽管 JavaScript 提供近似 类 的语法糖和（广泛使用）的 类 库来掩盖这个现实。JavaScript 的机制似乎一直在阻止你使用 类 设计模式，但是你迟早会面对它，其他语言中的 类 和 JavaScript 中的 类 并不一样。

note： 在软件设计中类是一种可选的模式，需自己决定是否在 JavaScript 中使用它。但许多开发者都非常喜欢面向类的软件设计

### 类的机制

许多面向类的语言中，"标准库" 会提供 Stack 类，它是一种 栈 数据结构（支持压入，弹出，等）。Stack 类内部会有些变量来存储数据，同时会提供些公有的可访问的行为，从而让你的代码可以和（隐藏）数据进行交互。

- 建造

建筑蓝图 -> 类

建筑物 -> 实例

- 构造函数

类实例是有一个特殊的类方法构造，这个方法名通常和 类 名相同。被称为构造函数。类构造函数属于 类 ? 构造函数大多需要用 new 调用。这样语言引擎擦才知道你想要构造一个新的类实例

### 类的继承

先定义一个类，在定义一个继承前者的类。后者通常称为 子类，前者称为 父类 。通常定义好一个子类，相对父类来说它是一个独立并且完全不同的类，子类会包含父类行为的原始副本，但也可以重写所有继承的行为和定义新行为。

note：父类和子类并不是实例

```
class Vehicle {
    engines = 1

    ignition() {
        output('Turning on my engine.')
    }

    drive() {
        ignition()
        output('Steering and moving forward!')
    }
}

class Car inherits Vehielc {
    wheels = 4

    drive() {
        inherited: drive()
        output('Rolling on all', wheels, 'wheels!')
    }
}

class SpeedBoat intherits Vehicle {
    engines = 2

    ignition() {
        output('Turning on my', engines, 'engines.')
    }

    pilot() {
        inherited: drive()
        output('Speeding through the water with ease!')
    }
}
```


- 多态

多态（在继承链的不同层次名称相同但是功能不同的函数）看起来似乎是从子类引用父类，但是本质上引用的其实是复制的结果

note: JavaScript 并不会（像类）自动创建对象的副本。

- 多重继承

有些面向类的语言允许继承多个 父类，多重继承意味所有父类的定义都就会被复制到子类中。看似非常有用的功能，把许多功能组合在一起，机制却带来很多复杂的问题。

note：JavaScript 本身不提供 多重继承 功能，但开发者相处了模拟类的复制行为，这个方法就是 *混入*

### 混入
由于 JavaScript 不会自动现实类的复制行为，所以需要我们手动实现复制能功能。

note: 混入模式（无论显式还是隐式）可以用来模拟类的复制行为，但通常会产生丑陋并且脆弱的语法。

- 显示混入

```
function minx(sourceObject, targetObject) {
    for(var key in sourceObject) {
        if(!(key in targetObejct) {
            targetObject[key] = sourceObject[key]
        }
    }

    return targetObject
}

var Vehicle = {
    engines: 1,
    ignition: function () {
        return 'Turning on my engine.'
    }

    drive: function () {
        this.ignition()
        return 'Steering and moving forward!'
    }
}

var Car = mixin(Vehicle, {
    wheels: 4,

    drive: function () {
        Vehicle.dirce.call(this)
        return 'Rolling on all' + this.wheels + 'wheels!'
    }
})
```

note：`OtherObject.methodName.call(this)` 显式伪多态，但会让代码加难懂并且难以维护。实际上无法完全模拟类的复制行为，因为对象（函数）只能复制引用，无法复制被引用的对象或者函数本身。

- 混合复制

```
function mixin(sourceObject, targetObject) {
    for(var key in sourceObject) {
        targetObject[key] = sourceObject[key]
    }
    
    return targetObject
}

var Vehicle = {
    // ..
}

// 首先创建一个空对象并把源对象的内容复制进去
var Car = mixin(Vehicle, {})


// 把新内容复制到Car
mixin({
    wheels: 4,

    drive: function () {
        // ..
    }
}, Car)
```

note：这两种方法都可以把不重叠的内容从源对象复制至目标对象。但引用类型对象仍可以 影响 对方。

- 寄生继承

```
// 类
function Vehicle() {
    this.engines = 1;
}

Vehicle.prototype.ignition = function () {
    return `Turning on my engine.`
}

Vehicle.prototype.drive = function () {
    this.ignition()
    return 'Stering and moving forward!'
}

// 寄生类
function Car() {
    // 实例类
    var cart = new Vehicle()
    // 定制
    car.wheels = 4
    // 引用  
    var vehicleDrive = car.drive
    
    // 重写
    car.drive = function () {
        vehicleDrice.call(this)
        return 'Rolling on all' + this.wheels + 'wheels!'
    }
    
    return car    
}

var myCar = new Car()

myCar.drice()
```

note：复制父类对象的定义，混入子类对象的定义（如需保留到父类的特殊引类），任何用这个复合对象构建实例

- 隐式混入

```
var Something = {
    cool: function () {
        this.greeting = 'hellom world'
        this.count = this.count ? this.count + 1 : 1
    }
}

Something.cool()
Something.greeting // hello, world
Something.count // 1

var Another = {
    cool: function() {
        // 隐式把 Something 混入 Another
        Something.cool.call(this)
    }
}

Another.cool()
Another.greeting // hello world
Another.count // 1
```

note： 通过构造函数调用或方法调用中使用 `Something.cool.call(this)`，通过 this 绑定并在 混入对象 的上下文中调用了它。

# 原型
JavaScript 中的对象有一个特殊


# 行为委托

