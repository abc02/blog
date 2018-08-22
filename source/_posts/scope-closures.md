---
title: 作用域
---
# 作用域
## 传统编译语言
### 编译流程三个步骤
- 分词/词法分析
- 解析/语法分析
- 代码生产
### 原理
- 字符串分解成有意义的代码块(词法单元)
- 词法单元流（数组）转换成一个由元素逐级嵌套所组成的代表程序语法结构的树(抽象语法树 Abstract Syntax Tree, AST)
- AST转换为可执行代码的过程称为代码生产，这个过程与语言、目标平台等息息相关
### 范例`var a = 2`AST
- 顶级节点VariableDeclaration
    - 子节点Identifier  
        - 子节点AssignmentExpression
        - 子节点NumericLiteral
## JavaScript
 "动态" 或 "解释执行" 语言
- JavaScript引擎
- 编译器
- 作用域
### 引擎
程序编译 执行过程
### 编译器
语法分析 代码生产
### 作用域
收集并维护由所有声明的标识符（变量、函数）组成的一系列查询，实施一套非常严格的规则，确定当前的代码对这些标识符的访权限
- 词法作用域
- 作用域气泡
- 作用域嵌套
- LHS
- RHS
- ReferenceError
- TypeError
- 函数作用域
- 块作用域
#### 词法作用域
- 词法作用域就是定义在词法阶段的作用域
- 词法欺骗
    - eval
    - with
#### 作用域气泡
- 作用域气泡的结构和互相之间的位置关系给引擎提供来足够的位置信息，引擎用这些信息来查找标识符位置
- 作用域查找从运行时所处的最内部作用域开始，逐级向上进行，直到遇见第一个匹配标识符为止
- 多层的嵌套作用域中可以定义同名的标识符。叫作 "遮蔽效应"
#### LHS(Left-Hand Side)
查找标识符的容器本身
#### RHS(Right-hand Side)
获取标识符的源值
#### ReferenceError
作用域判别失败
- RHS 所有作用域查询不到所需的标识符
- LHS 严格模式禁止自动或隐式创建全局变量 or 宽松/懒惰模式全在全局作用域中创建标识符
#### TypeError
作用域判别成功，但操作是非法或不合理
- RHS 对非函数类型的标识符进行函数调用 or 引用null或undefined类型的值中的属性
#### 函数作用域
- 隐藏内部实现
    - 最小特权原则中引出/最小授权/最小暴露原则
    - 避免同名标识符之间的冲突
- 全局命名空间
- 模块管理
##### 函数声明
##### 函数表达式
- 匿名函数表达式
    - 在栈追踪中不会显示有意义的函数名
    - 无法引用自身
    - 省略了对于代码可读性/可理解性很重要的函数名，一个描述性的名称可以让代码不言自明
- 具名函数表达式
    - 有效解决以上问题
- 立即执行函数表达式（IIFE）
    ###### 范例
    - `(function foo() {})()`
    - `(function foo() {}())`
#### 块作用域
- with
- try/catch catch分句会创建一个块作用域
- let关键字可以将变量绑定到所在的任意作用域中（通常是 `{...}`内部）
    ###### 范例
    let为其声明的标识符隐式地劫持了所在的块作用域
     ```ES6
    var foo = true
    if (foo) {
        { // <- 显式的块作用域
            let bar = 2
            console.log(bar)
        }
    }
     ```
- cosnt常量
#### 提升
- 函数优先
- 函数声明会被提升
- 函数表达式不会被提升
- 变量提升
#### 范例
编译器将程序语法分析分解成词法单元，然后将词法单元解析成一个树结构。但是当编译器开始进行代码生成时，它对程序处理方式会和预期的有所不同。编译器在编译过程生成了代码，引擎执行它时，会通过查找标示符来判断它是否已声明过。查找的过程由作用域进行协助

# 闭包
JavaScript中闭包无处不在，你只需要能够识别并拥抱它
## 理解和识别闭包
当函数可以记住访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行
### 基于词法作用域的查找规则
函数 bar 具有一个涵盖 foo 作用域的闭包。因为 bar 嵌套在 foo 内部
```
function foo () {
    var a = 2
    function bar () {
        console.log(a) // 2
    }
}

foo()
```
### 闭包
1. 函数 bar 的词法作用域能够访问 foo 的内部作用域，然后将函数 bar 本身当作一个值类型进行传递。在 foo 执行后将 bar 所引用的函数对象本身当作返回值赋值给变量 baz 并调用，实际上只是通过不同的标识符引用调用内部的函数 bar ，拜 bar 所声明的位置所赐， bar 拥有涵盖 foo 内部作用域的闭包，所以该作用域能够保持存活。以供 bar 在之后任何时间进行引用
2. 这个函数在定义时的词法作用域以外的地方被调用。闭包使得函数可以继续访问定义时的词法作用域
3. 无论通过何种手段将内部函数传递到所在的词法作用域以外，他都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包

- 问： 什么是闭包？
- 答：无论在何处执行该函数依然持有对该作用域的引用，而这个引用就叫作闭包
     - 附： 内部函数在定义时的词法作用域以外的地方调用。闭包使得函数可以继续访问定义时的词法作用域

```
function () {
    var a = 2
    function bar () {
        console.log(a)
    }
    return bar
}
var baz = foo()
baz() // 2 闭包效果
```
#### 内存回收
在 foo 执行， 通常会期待 foo 的整个内部作用域都被销毁，引擎有垃圾回收器用来释放不再使用的内存空间
- 问：什么是内存垃圾回收？
- 答：通常函数执行后，内部不会再次使用，引擎垃圾回收器自然会对其进行回收

由于看上去 foo 内容不会再次使用，所以很自然地会考虑对其进行回收，而闭包的 "神奇" 之处正是可以阻止这件事情的发生，实际上内部作用域依然存在，因此没有被回收。
- 问：谁在使用这个内部作用域？
- 答：函数 bar 本身在使用

#### 闭包的应用
- 定时器
- 事件监听器
- Ajax请求
- 跨域窗口通信
- Web Workers
- 任何其他异步（同步）任务

只要使用了**回调函数**实际上就是使用了闭包

#### 模块

##### 词法作用域查找规则
这段代码没有明显的闭包，但是私有数据变量，以及私有的内部函数，内部函数的词法作用域（而这就是闭包？）也就是 foo 的内部作用域
```
 function foo () {
    var something = 'cool'
    var another = [1, 2, 3]

    function doSomething () {
        console.log(somethin)
    }
    
    fucntion doAnorther () {
        console.log(another.join('!'))       
    }
 }

 ```
##### 最常见的实现模块的方式通常称为模块暴露
模块模式需要具备两个必要条件
- 必须有外部的封闭（封装？）函数，该函数至少被调用一次
- 封闭（封装？）函数必须返回至少一个内部函数引用，这样内部函数才能在私有作用域中形成闭包，并且能访问或修改私有状态
```
function CoolModule () {
    var something = 'cool'
    var another = [1, 2, 3]

    function doSomething () {
        console.log(somethin)
    }
    
    fucntion doAnorther () {
        console.log(another.join('!'))
    }

    return {
        doSomething: doSomething,
        doAnorther: doAnorther
    }
}

var foo = CoolModule()
foo.doSomething()
foo.doAnorther()
 ```
##### 模块函数 => IIFE
立即执行具名函数并将返回值赋值给但例的模块实例标识符 foo
```
var foo = (function CoolModule () {
    var something = 'cool'
    var another = [1, 2, 3]

    function doSomething () {
        console.log(somethin)
    }
    
    fucntion doAnorther () {
        console.log(another.join('!'))
    }

    return {
        doSomething: doSomething,
        doAnorther: doAnorther
    }
})()

foo.doSomething()
foo.doAnorther()
 ```

#### 现代的模块机制
- CommonJS
- AMD
- CMD
- UMD
- ES6

#### 模块历史演变
- 直接定义依赖
- 命名空间模式
- 闭包模块化模式
- YUI
- CommonJS 用于 Node.js 服务端
- AMD RequireJS
- CMD SeaJS
- UMD
- ESMAScript2015 => ES6 称为 编译时夹在 或 静态加载
