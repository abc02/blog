---
title: class
---

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

