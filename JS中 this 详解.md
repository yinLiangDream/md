---
title: JS中 this 详解 
tags:  JavaScript,this
grammar_cjkRuby: true
---


`this` 是 `js` 中的一个关键字。

在了解 `this` 之前，先了解一下 `js` 中的执行环境。执行环境是 `js` 中最为重要的一个概念， `js` 中的执行环境主要有两种：全局执行环境和函数执行环境。执行环境（Execution Context ）简称 `EC` ，可以将其看作一个对象，它由变量对象、`this`、作用域链组成。由此引出 `this` 。

在全局执行环境下， `this` 指向 `window` 对象；在函数执行环境下，`this`指向调用该函数的对象。

全局执行环境：

在全局执行环境下， `this` 等价于 `window` 。
```javascript
// 全局环境执行
console.log(this); // Window
console.log(window === this); // true
```

```javascript
// 示例
function test() {
    var a = 1;
    console.log(this.a); // undefined
    console.log(this); // Window
}
test();
```

上代码中，定义了一个`test`函数并调用它，`test()`等价于`window.test()`，即此时调用该函数的对象是`window`对象，故在执行函数的过程中，函数内部`this`指向`window`对象，由于`window`对象中并未定义变量`a`，故打印`this.a`结果为`undefined`。此时若将代码稍微改变一下，结果即如下所示：

```javascript
var a = 1;
function test() {
    console.log(this.a); // 1
    console.log(this); // Window
}
test();
```

函数执行环境：

1. 作为对象方法的调用

```javascript
var o = {
    user: "tom",
    fn: function() {
        console.log(this.user); // tom
    }
}
```
上代码中，由于调用`fn`函数的是`o`对象，故在执行函数的过程中`this`指向`o`对象。`this`的指向在函数创建的时候是无法确定的，在调用的时候才能确定，总是指向调用函数的那个对象。

2. 作为构造函数调用

```javascript
function Fn() {
    this.user = "tom";
}
var a = new Fn();
console.log(a.user); //tom
```
上代码中，创建了一个新对象，并将构造函数的作用域赋值给了新对象，因此`this`就指向这个新对象。

3. `call` and `apply`

`call()`和`apply()`是函数对象的方法，用于改变`this`的指向。

```javascript
function testFunc(val){
    this.a = val;
    this.b = 'bb';
}
function execFunc() {
    var a = 'exec aa';
    var b = 'exec bb';
    console.log(this.a, this.b);
}
var testInstance = new testFunc('aa');
execFunc.call(testInstance); // aa bb
execFunc.apply(testInstance); // aa bb
```

`call` `apply`的第一个参数是`func`函数运行时的`this`值。上图中通过`call` `apply`函数将`execFunc`的`this`值指向`testInstance`对象。