---
title: 理性分析 JavaScript 中的闭包
date: 2018-2-27
updated: 2018-2-27
tags: 
- JavaScript
categories: 
- 理性分析 JavaScript
---

> 闭包是指有权访问另一个函数作用域中的变量的函数。创建闭包的常见方式，就是在一个函数内部创建另一个函数。
> ——[《JavaScript高级程序设计》](https://book.douban.com/subject/10546125/)
> 
>闭包是一个函数和声明该函数的词法环境的组合。从理论角度来说，所有函数都是闭包。
>——[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)
> 
> 当函数可以记住并访问所在的词法作用域时，就产生了闭包。
>——[《你不知道的JavaScript（上卷）》](https://book.douban.com/subject/26351021/)

<!--more-->

<br>

### 闭包的作用

<hr>

正常来说，一个函数中的局部变量是私有的。函数外无法访问到该函数中的局部变量。

有时候，我们希望能够从函数外访问到函数内的局部变量。借助闭包，我们便可以实现这个目标。

在 JavaScript 中，内部函数可以访问外部函数中的变量。而函数是 JavaScript 中的一等公民，可以作为返回值返回。通过引用返回的内部函数，我们便可以访问到外部函数中的局部变量。这就构成了一个闭包。

示例：
```javascript
function foo (){
  var num = 0
  function bar (){
    console.log(num++)
  }
  return bar;
}

var func = foo()
func() // 0
func() // 1
```

<br>

### 闭包的特点

<hr>

闭包的特点和闭包的行为息息相关。

根据垃圾回收机制，JavaScript 垃圾回收器将定期清除不可获得的对象。
 
我们通过持有内部函数的引用来访问到外部函数中的局部变量。使得外部函数对象终保存在内存中，这样我们就可以一直访问到外部函数中的局部变量。同时，如果滥用闭包，会导致内存消耗过大。

<br>

### 总结

<hr>

文章开头列举了几个闭包的定义，可以辅助我们理解闭包。从实际出发，我们可以把闭包理解为解决访问局部变量的一种方案。

闭包提供了 JavaScript 从函数外访问局部变量的能力，使得局部变量可以常驻内存。滥用闭包，会导致内存消耗过大。

#### 相关知识点
- 内存回收机制
- 作用域

#### 参考资料
- [学习Javascript闭包（Closure）—— 阮一峰](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
- [MDN Closures](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)
- [《JavaScript高级程序设计》](https://book.douban.com/subject/10546125/)
- [《你不知道的JavaScript（上卷）》](https://book.douban.com/subject/26351021/)




