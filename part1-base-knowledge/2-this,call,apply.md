# this、call、apply

## 2.1 this

this的指向大致可以分为以下四种，但请记住一句话**谁调用它this就指向谁**：

- 在**对象的方法**中调用this，此时这里的this等于该对象。

- 在**普通函数**使用this，此时this等于window。

- **构造函数**调用指向返回对象。

- **call，apply**的第一个参数。

### 2.1.1首先来看对象方法中调用this

```base
const obj = {
    name: 'name1',
    getName() {
        console.log(this.name)
    }
}
const obj2 = Object.create(obj)
obj2.name= 'name2'
obj.getName() // name1
obj2.getName() // name2
```

此时的``this``就是指向调用它的对象。``obj.getName()``是``obj``调用的``getName``方法，所以``this = obj``。而``obj2.getName()``中调用的虽然是``obj``的方法，但它指向的是``obj2``，所以``this.name = obj2.name``。

### 2.1.2再来看在函数中使用this

简单的例子：直接定义一个全局的方法``getName``，并调用它。由于谁调用它``this``就指向谁，所以此时``this``等于``window``。会打印出全局的``name: theName``。

```base
window.name = 'theName'
const getName = function () {
    console.log(this.name)
}
getName() // 'theName'
```

注意，这里是在非严格模式下才会指向，如果使用了``'use strict'``，那么会报错。
再看一个定义在obj对象内的方法``getName``，同时再定义一个``getName``使其等于``obj``的``getName``（const getName = obj.getName注意没有括号）。此时调用的对象分别是``obj``，``window``时，``this``就分别指向``obj``和``window``了。

```base
window.name = 'theName'
const obj = {
    name: 'objName',
    getName() {
        console.log(this.name)
    }
}
const getName = obj.getName
console.log(obj.getName()) // 'objName'
console.log(getName()) // 'theName'
```

综上，请记住，**谁调用了this，this就指向谁**。

### 2.1.3 构造器

首先要知道，大部分的js函数都可以当做构造器使用。我们可以直接把一个普通函数当做构造器使用。那么构造器与普通函数有什么区别呢？调用的方式不同。使用new运算符调用构造器，该构造器会返回一个对象。构造器里的this就指向这个对象，所以``obj.name = this.name``。

```base
const constructor = function () {
    this.name = 'seven'
}
const obj = new constructor()
console.log(obj) // { name: 'seven' }
console.log(obj.name) // 'seven'
```

但如果构造器显示的返回一个object，那么返回的就是显式定义的这个对象：

```base
const constructor = function () {
    this.name = 'seven'
    return {
        name: 'annni'
    }
}
const obj = new constructor()
console.log(obj) // { name: 'annni' }
console.log(obj.name) // 'annni'
```

### 2.1.4 call，apply来指定上下文的'this'

```base
const replay = function(name, job) {
    const replays = this.name + ' is a ' + this.job
    console.log(replays)
}
const person = {
    name: 'A',
    job: 'student'
}
replay()
replay.call(person) // A is a student
replay.apply(person) // A is a student
```

## 2.2 call和apply

它们是给function的原型定义的两个方法。``Function.prototype.call``和``Function.prototype.apply``方法，作用一模一样，仅在传入参数的形式不同。

- function.call(thisArg, arg1, arg2, ...) 第一个参数是函数体内this的指向，后面的是转个传入的参数。

- function.apply(thisArg, [argsArray]) 第一个参数是函数体内this的指向，后面的是数组或类似数组对象。

- 如果传入的第一个参数是``null``，那么函数体内的this会指向window。但如果是在严格模式下，函数体内的this还是为null。

- 有时候使用call和apply的目的不在于this的指向，而在于借用某个对象的方法：

```base
Math.max.apply(null,[1,2,3,4,5]) // 输出5
```

### 2.2.1 实际用途

#### 改变或修正this的指向

```base
var obj1 = { name: 'sven'}
var obj2 = { name: 'anne' }
window.name = 'window'
var getName = function(){
    alert ( this.name )
}
getName();    // 输出: window
getName.call( obj1 );    // 输出: sven
getName.call( obj2 );    // 输出: anne
```

实际开发中可能会遇到``this``不经意被改变的场景，就可以使用``call``或``apply``去修正。

#### Function.prototype.bind

- ``bind()``方法创建一个新的函数，在 ``bind()``被调用时，这个新函数的``this`` 被指定为 ``bind()`` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。
- 返回一个原函数的拷贝，并拥有指定的 this 值和初始参数。

```base
window.num = 9
const obj = {
    num: 10,
    getNum() {
        return this.num
    }
}
const getNum = obj.getNum
const bindNum = getNum.bind(obj)
obj.getNum() // 10
getNum()     // 9，因为函数是在全局作用域中调用，this指向window
bindNum()    // 10， this 被指定为 bind() 的第一个参数obj，使用()去执行
```

- 如何实现一个bind函数？

简化版：

```base
Function.prototype._bind = function(context) {
    const self = this
    return function() {
        return self.apply(context, arguments)
    }
}
const obj = {
    num: 123
}
const func = function() {
    console.log(this.num) // 123
}._bind(obj)
func()
```

```base
Function.prototype._bind = function (context) {
  let self = this;
  let firstArg = [].slice.call(arguments, 1);
  return function () {
    let secArg = [].slice.call(arguments);
    let finishArg = firstArg.concat(secArg);
    return self.apply(context,finishArg);
  }
}
const obj = {
    num: 123
}
const func = function(a,b,c,d) {
    console.log(this.num) // 123
    console.log(a,b,c,d) // 1 2 3 4
}._bind(obj, 1, 2)
func(3,4)
```

可能以上的例子并不好明白，需要可以先看一下**函数柯里化currying**。

- 函数柯里化的概念

参考： [详解JS函数柯里化](https://www.jianshu.com/p/2975c25e4d71)

```base
// 普通的add函数
function add(x, y) {
    return x + y
}

// Currying后
function curryingAdd(x) {
    return function (y) {
        return x + y
    }
}

add(1, 2)           // 3
curryingAdd(1)(2)   // 3
```

## 总结

- 对于this的指向，谁调用它就指向谁（对象中、函数中、构造器、call和apply）。

- 使用call和apply修正this的指向。

- 利用call和apply实现一个bind函数，理解函数柯里化currying可能会帮助理解如何去实现一个bind函数。
