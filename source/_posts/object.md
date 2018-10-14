---
title: object
---

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

