# 面向对象的JavaScript

## 1.1 动态类型语言和鸭子类型

- 编程语言按照数据类型大体被分成两类：**静态类型语言&动态类型语言**。js属于动态类型语言。为啥？因为静态语言需要编译时已经被确定数据类型，而动态类型语言是被赋值后才成为某种类型。
- 举个栗子！变量`a`可以等于Number/String/Boolean/null/undefined/Object/Array种类的值，由自己决定，所以js属于动态语言类型。
- **鸭子类型**的通俗说法是：如果他走起路来像鸭子，叫起来像鸭子，那么它就是鸭子。（即使它是一只鸡，但拥有了这两个属性，我就认为它是鸭子，哈哈。）
- 在动态类型语言的面向对象设计中，**“鸭子类型”思想非常滴重要**。比如你知道啥是【面向接口编程】吗？就是只要实现接口的要求（走路像鸭子，叫起来像鸭子），其他的我都不管。
- 顺便补充【面向对象编程】，我有一个鸭子类，其它的鸭子只要继承我，扩展我就可以了。比如js的`axios`就是一个类。如果你创建一个类继承`axios`类。就是面向对象了。

## 1.2 多态polymorphism

- 当我们给不同对象发出同一命令，不同对象分别产生执行结果就是【多态】。把**不变的部分隔离出来，把可变的部分封装起来**，这给予了我们扩展程序的能力。

```base
function commandAnimal(animal) {
    animal.sound()
}
class Duck {
    sound() {
        console.log('嘎嘎嘎')
    }
}
class Chicken {
    sound() {
        console.log('叽叽叽')
    }
}
let duck = new Duck()
let chicken = new Chicken()
commandAnimal(duck) // 嘎嘎嘎
commandAnimal(chicken) // 叽叽叽
```

## 1.3 封装

- 封装意味着我们不在乎内部如何实现，而只关心对外的接口有没有变化。

## 1.4 原型模式和基于原型继承的 JavaScript 对象系统

- 第一个设计模式【原型模式】，如果我们**需要一个和某个对象一摸一样的对象**就是用原型模式。
- 使用`const B = Object.create(A)`克隆出来的对象，他的`__proto__`等于被克隆对象的`prototype`。意味着直接console.log(B)打印出的是{}，因为它**克隆的是原型上的属性**。
- 使用`Object.assign()`克隆的是可枚举属性。

```base
const A = {
    name: 'human',
    showName() {
        console.log(this.name)
    }
}
const B = Object.create(A)
const C = Object.assign({}, A)
console.log(B) // {}
B.name = 'abc'
B.showName() // abc
console.log(C) // {name: "human", showName: ƒ}
```

- `Object`是`Animal`的原型，而`Animal`是`Duck`的原型，它们之间形成了一条**原型链**。具体是我们想要`Duck`的某个方法，会先在`Duck`的`prototype`中找，没找到的话，去`Duck`的`__proto__`找。而`Duck`的`__proto__ = Animal`的`prototype`。以此向上找，直到`Object`的`prototype`停止，返回`undefined`。

- 为啥是undefined不是null？`null` 表示一个值被定义了，定义为“空值”；`undefined` 表示根本不存在定义。

- ES6中带来了新的`Class`语法，但其背后的原理还是**通过原型机制创建对象**：

```base
class Animal {
    constructor(name) {
        this.name = name
    }
    getName() {
        return this.name
    }
}

class Dog extends Animal {
    constructor(name) {
        super(name)
    }
    speak() {
        return "woof"
    }
}
const dog = new Dog("dd")
console.log(dog.getName() + ":" + dog.speak()) // dd:woof

```

## 总结

- “鸭子类型”可以帮助我们理解什么是**面向接口编程**。

- 【多态】的实际含义是同一操作于不同对象上，可以产生不同的执行结果。

- `Object.create()`就是原型模式的天然实现。

- `Class`的机制就是原型模式的实现的。因为Class是非常常用到的，可见原型模式十分重要和强大。

- 常见的**原型链问题**可以由原型模式引发思考。
