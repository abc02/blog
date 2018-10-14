---
title: this
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

