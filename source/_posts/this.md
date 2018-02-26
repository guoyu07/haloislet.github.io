---
title: 理性分析 JavaScript 中的 this
date: 2018-2-26
updated: 2018-2-26
tags: 
- JavaScript
categories: 
- 理性分析 JavaScript
---

> 在每一个方法中，关键字 this 表示隐式参数。 
> —— 《Java 核心技术 卷Ⅰ》

<!--more-->
<br>

## this 是什么？

<hr>

了解 python 的同学可能会知道，python 构造函数中总是会出现 self 参数。这个参数用来表示创建的实例对象。



```python
class Student(object):
    def __init__(self, name, score):
        self.name = name
        self.score = score
```
在 JavaScript 和 Java 中这个参数被隐藏了。我们不必在参数列表中显式声明这个参数，就可以在函数中使用这个参数。这个参数就是 this 。

```javascript
function Student(name, score){
	this.name = name
	this.score = score
}
var studentA = new Student('a', 100)
console.log(studentA.name, studentA.score) // a 100
```

### 隐式参数
援引 《Java 核心技术 卷Ⅰ》 中的一句话：在每一个方法中，关键字 this 表示隐式参数。 所谓的隐式参数，就是没有在参数列表中显式声明的参数。隐式参数和参数列表中定义的显式参数统称为形式参数。与形式参数相对应的是实际参数。

### 形式参数和实际参数
形式参数，简称形参。形参就是在定义函数的时候使用的参数，用来接收调用该函数时传递的参数。如上述代码中的 name ，score 参数都是形参。

实际参数，简称实参，实参就是调用该函数时传递的参数。如上述代码中的 'a' ， 100 都是实参。

### 为什么 this 的值是在调用时确定的？
《 你不知道的JavaScript（上卷）》中提了一个问题，问：为什么采用词法作用域的 JavaScript 中的 this 的值是在调用时确定的？

在理解了形参和实参之后，我们便能很好地理解这个问题了。

因为 this 是一个形参，形参的值是由实参决定的。而传参这个操作时在调用时发生的，所以 this 的值是在调用时确定的。

<br>

## this 的值

<hr>

既然 this 的值是由实参的值决定的，那么这个实参的值到底是什么呢？

参考 《Java 核心技术 卷Ⅰ》 中的一句话：隐式参数的值是出现在函数名之前的对象。当作为构造函数时，this 用来表示创建的实例对象。来看两个例子：
```javascript
function bar () {
  console.log(this.name)
}
var foo = {
  name: 'foo',
  bar: bar
}
foo.bar() // foo
```
this 指向函数名（bar）之前的 foo 对象

```javascript
function Student(name, score){
	this.name = name
	this.score = score
}
var studentA = new Student('a', 100)
console.log(studentA.name, studentA.score) // a 100
```
this 指向创建的实例对象 studentA 

### call apply bind
JavaScript 也提供了几个函数去改变 this 的值。这几个函数都会返回一个原函数的拷贝，并在这个拷贝上传递 this 的值。所以从结果上看，我们可以看到原有的 this 会被覆盖。

```javascript
function bar () {
  console.log(this.name)
}
var foo = {
  name: 'foo',
  bar: bar
}
var obj = {
  name: 'obj',
}
foo.bar.call(obj) // obj
```
this 指向新的对象 obj 。

### 为什么 this 指向了全局对象？
《 你不知道的JavaScript（上卷）》中描述了一种现象：this 丢失了原来的绑定对象，指向了全局对象。书中称为隐式丢失。来看示例：
```javascript
function foo() { 
	console.log( this.a )
}
var obj = { 
	a: 2,
	foo: foo 
}
var bar = obj.foo // 赋值操作
var a = "oops, global" 
bar() // "oops, global"
```
JavaScript 只有值传递，没有引用传递。在赋值操作的时候，其实是将一个引用的拷贝赋值给另外一个变量。`var bar = obj.foo` 在这个语句中，没有传参操作，所以 this 的值是由 bar 函数在调用时传递的那个实参决定的。这个实参如未显式指定，那么便是指向全局对象。所以上述代码中的 this 指向了全局对象。

同样的，我们在函数传参的过程中，经常发现隐式丢失问题，原因也是中间发生了一次赋值操作。代码示例如下：
```javascript
var name = 'global'
function bar () {
  console.log(this.name)
}
var foo = {
  name: 'foo',
  bar: bar
}
function callFunc(func){
  func()
}
callFunc(foo.bar) // global
```
在传参的过程中，发生了`func = foo.bar`的赋值操作，导致最后 this 的值指向了全局对象。

但是如果我们使用 bind 绑定了 this 的值，那么在发生赋值操作时，this 的值将不再改变。来看下面例子。

### 再谈 bind
bind 和 call ，apply 有一点不同的是  call，apply 返回的是调用结果，而 bind 返回的是绑定 this 后的函数对象。那么当绑定 this 后的函数作为实参传入函数时，与未绑定 this 的结果就完全不同了。

来看下面的例子。

```javascript
var name = 'global'
function bar () {
  console.log(this.name)
}
var foo = {
  name: 'foo',
  bar: bar
}
function callFunc(func){
  func()
}
callFunc(foo.bar.bind(foo)) // foo
```
将 bar 函数中的 this 绑定到 foo 再传入 callFunc 函数中，最后打印的结果是 foo 。

实际上， bind 函数内部维护了一个闭包，使得调用始终发生在函数内部，来保证 this 的值不变。来看 MDN 提供的 ployfill
```
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          return fToBind.apply(this instanceof fNOP
                 ? this
                 : oThis,
                 aArgs.concat(Array.prototype.slice.call(arguments)))
        };

    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype
    }
    fBound.prototype = new fNOP()

    return fBound
  }
}
```

```
// return 部分
 return fToBind.apply(this instanceof fNOP
                 ? this
                 : oThis,
                 aArgs.concat(Array.prototype.slice.call(arguments)))
```

在 return 的时候使用了 apply 函数来改变 this ，若未发生 new 操作，那么这个 this 的值将绑定到 bind 函数提供的那个对象。

### new 操作
当发生 new 操作时，this 将绑定到这个实例对象。
从上面这个 ploy fill 可以看出 new 操作中的 this 值会覆盖原有 this 的值。来看例子

```
function bar () {
  this.name = 'bar'
}
var foo = {
  name: 'foo',
}

var a = bar.bind(foo)
a()
console.log(foo.name) // bar
var b = new a()
console.log(b.name) // bar
```
当执行 new 操作之前，a 函数中的 this 指向 foo。当执行 new 操作之后，a 函数中的 this 指向了 b 。

new 操作会返回一个重新绑定 this 后的新对象。所以当发生 new 操作之后，原有的 this 发生了改变。具体步骤如下：
1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行 [[ 原型 ]] 连接。
3. 这个新对象会绑定到函数调用的 this 。
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

### 箭头函数中的 this
箭头函数中的 this 继承了父作用域的 this。

```javascript
var name = 'global'
var foo = {
  name: 'foo',
  bar: () => {
	console.log(this.name)
  }
}

foo.bar() // global
```
箭头函数的父作用域为全局作用域，全局作用域的 this 指向全局对象，所以 this 指向了全局对象。

```javascript
var name = 'global'
var foo = {
  name: 'foo',
  bar: function () {
	setTimeout(() => {
	  console.log(this.name)
	},100)
  }
}

foo.bar() // foo
```
箭头函数的父作用域为 bar 函数，在调用时，父作用域 bar 函数中的 this 指向了 foo 函数，所以箭头函数中的 this 指向了 foo 。

### 严格模式下的 this 
严格模式下禁止 this 指向全局对象。在严格模式下当 this 指向全局对象的时候会变成 undefined 。

<br>

## 总结

<hr>

1. this 指向创建的实例对象或函数名之前的对象。如未指定，便是指向全局对象。
2. 由于 call 、apply 、bind 函数会返回一个原函数的拷贝，并在这个拷贝上传递 this 值。所以当使用 call 、apply 、bind 函数会覆盖原有的 this 值。
3. new 操作可以覆盖 call、apply、bind 绑定的 this 值。

### tips

- 严格模式下禁止 this 指向全局对象。在严格模式下当 this 指向全局对象的时候会变成 undefined 。
- 在发生赋值操作时，由于引用复制， this 的值指向被赋值变量的调用对象。
- ES 6 中新增箭头函数，可以继承父作用域的 this ，可以解决 this 隐式丢失的问题。

### 相关知识点
- 词法作用域和动态作用域
- 闭包
- 作用域和作用域链
- 严格模式
- ES6 新增特性
- 引用传递和值传递

<hr>

欢迎交流指正

