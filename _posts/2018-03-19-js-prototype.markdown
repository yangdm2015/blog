---
layout:     post
title:      "追根溯源话原型"
subtitle:   "到底是先有对象还是先有原型？"
date:       2018-03-19
author:     "Shany"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - JavaScript
    - 原型
---

> This document is not completed and will be updated anytime.


# 目录

1. [问题起源](#问题起源)
2. [可以接受的解释](#可以接受的解释)
3. [发散](#发散)


# 问题起源



> js是一门十天内被设计出来的语言.它的原型机制一直很让人费解。本文根据一个既是对象，由不是对象的东西说起，给出了一种解释。



众所周知，在js中，要判断一个对象是什么类型 ，在同一个上下文中，可以使用instanceof函数
```js
let a = {name:'a'}
a instanceof Object // true
```
在形如 A instanceof B  的表达式中 instanceof函数实际上是看A的原型链上是否能追溯到<b>B的原型<b>

```js
let a = {name:'a'}
a.__proto__===Object.prototype  // true
let fn = function(name){ this.name = name}
fn.__proto__===Function.prototype  // true
```
因此，我们平常说，a是一个Object，实际上是说

> a的__proto__属性指向了Object的prototype属性。

```js
let fn = function(name){ this.name = name}
let o = new fn('Jerry')
o.__proto__===fn.prototype  // true
fn.prototype.__proto__ === Object.prototype  // true
o.__proto__.__proto__ === Object.prototype // true
```
经过上面一番操作，我们可以确定地说，o是一个fn，同时，o也是一个Object。所以，只要是被一个构造函数new出来的对象，都是一个Object，都具有Object的prototype上的方法和属性。

js 中一切皆对象，函数也不例外，我们来看看这句话对不对
```js
fn.__proto__.__proto__===Function.prototype.__proto__ // true
Function.prototype.__proto__ ===Object.prototype // true
fn.__proto__.__proto__ ===Object.prototype // true
```
由此可见，每一个函数也是一个Object，拥有Object的prototype上的方法和属性。
那Object.prototype 是什么呢？

```js
Object.prototype.__proto__ // null
Object.prototype.__proto__ === null // true
typeof Object.prototype.__proto__ // "object"
Object.prototype.__proto__ instanceof Object // false
```
按照上面 a 是一个 Object 的定义，Object.prototype不是任何一个类型
这就尴尬了，js里一切皆对象，但Object.prototype除外？

# 可以接受的解释
Function 是一个 Function ,而且 Object 在Function 的原型链上

```js
Function.__proto__ === Function.prototype // true
Function.prototype.__proto__  === Object.prototype  // true
Function.__proto__.__proto__  === Object.prototype  // true
// 所以以下成立
Function instanceof Object // true
```
与此同时，我们可以发现 Object是一个Function，而Object.__proto__是一个Object

```js
Object.__proto__=== Function.prototype  // true
Object.__proto__.__proto__===Object.prototype  // true
```

我们发现，Object的原型链中有Function，而Function的原型链中有Object。按照js中继承的说法，Object继承自Function，Function继承自Object 
我们可以总结：除Object.prototype外的任意一个对象，它的原型链上都会有Object，所以都可以说它们继承自Object，都具有Object.prototype上定义的方法。

这么说，可以想象，如果我是js语言的设计者，我的设计过程很有可能是这样的：
1. 先定义一个名叫Object.prototype 的东西，在它上面定义一堆所有对象都应该具有的方法，比如hasOwnProperty ，valueof，toString等等。
2.  再由Object.prototype派生出Function.prototype和Function.__proto__  这俩是一个东西，它们上面有一堆所有的函数都具有的方法，比如apply等等。到这里还没Object什么事
3. 下一步，由Function.prototype 派生出Object 这个具体的函数，这样它就会同时有Object.prototype和Function.prototype上的方法。在这之后，所有由Object派生出去的对象，都只会具有Object.prototype上的方法。
4. 再后来 Function.prototype 派生出各种各样的函数，这些函数都只会具有Function.prototype上的方法。但由于Function.prototype是Object.prototype派生出来的，所以所有的函数都同时也会具有Object.prototype上的方法。同时，所有这些派生出的函数实例的prototype属性实际上是由Object.prototype派生出来的，所以也具有Object.prototype上的方法。
5. 再由这些函数实例（fn）new 出来的对象，则实际上是由fn.prototype派生出来的，也只会具有Object.prototype上的方法，所以这些也就是对象。

我这里用了<b>派生</b>这个词，而不是继承，我定义 a 派生出 b的意思是
> b的方法和属性，a天然都有，并且a中的这些方法和属性，初始时，都默认指向b中的这些方法和属性

说得很绕，但意思你们懂得！所以答案是:<strong>先有原型，再有对象！</strong>

# 发散

这个问题就好像是问先有dna还是先有生命？个人感觉，Object.prototype就像是最初的dna一样，dna是蛋白质，它从无到有，偶然具有了可以产生蛋白质的机制，就想第一步由Object.prototype派生出Function.prototype一样。有了这个机制，后来竟然可以复制自己了（Object.__proto__），再后来，这些细胞和细胞内的dna不断演化，才派生出了各种对象。





