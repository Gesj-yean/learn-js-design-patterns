# 闭包

## 3.1 闭包

先记住：**闭包可以让你从内部函数访问外部函数作用域**。首先来了解和闭包有关的两个变量：**变量的作用域**和**变量的生存周期**。

- 变量的作用域

声明一个变量时，如果没有带var，这个变量就会成为全局变量。如果在函数内部使用var声明变量，这时的变量是局部变量（只有在函数内部才能访问这个变量，外部访问不到）。
变量的搜索是由内到外直达全局变量，而非由外到内。

```base
var a = 1
var fun1 = function() {
    var b = 2
    var fun2 = function() {
        var c = 3
        console.log(a) // 1
        console.log(b) // 2
    }
    fun2()
    console.log(c) // 报错
}
```

- 变量的生存周期

全局变量的生存周期是永久的，除非主动销毁。一个闭包的例子：

```base
var fun1 = function() {
    var num = 1
    return function() {
        num++
        console.log(num)
    }
}
const f = fun1() // f是执行fun1创建的匿名函数的引用
f() // 2
f() // 3
f() // 4
```

- 闭包经典应用

这段代码点击每个div打印的都是5。当事件被触发时，for循环早已结束，此时的i为5（i++了5遍）。所以在onclick事件函数中顺着作用域链从内到外查找时，查找到的值总是5。

```base
<html>
<body>
    <div>1</div>
    <div>2</div>
    <div>3</div>
    <div>4</div>
    <div>5</div>
</body>
</html>
<script>
    var nodes = document.getElemnetByTagName('div)
    for(var i = 0; i < nodes.length; i++){
        nodes[i].onClick = function() {
            console.log(i)
        }
    }
</script>
```

解决办法是在闭包的帮助下，把每次循环的i值封闭起来。

```base
for(var i = 0; i < nodes.length; i++) {
    (function(i){
        nodes[i].onClick = function() {
            console.log(i)
        }
    })(i)
}
```

- 闭包的更多作用

用闭包封装私有变量和方法：有利于限制对代码的访问，避免非核心方法弄乱代码的公共接口部分。

```base
var makeCounter = function() {
  var privateCounter = 0;  // 私有变量
  function changeBy(val) { // 私有函数
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }  
};

var Counter1 = makeCounter();
var Counter2 = makeCounter();
console.log(Counter1.value()); /* logs 0 */
Counter1.increment();
Counter1.increment();
console.log(Counter1.value()); /* logs 2 */
Counter1.decrement();
console.log(Counter1.value()); /* logs 1 */
console.log(Counter2.value()); /* logs 0 */
```

## 总结

- 闭包就是指有权访问另一个函数作用域中的变量的函数。

- 它由两部分构成：函数，以及创建该函数的环境。环境由闭包创建时在作用域中的任何局部变量组成。

- 外部函数调用之后其变量对象本应该被销毁，但闭包的存在使我们仍然可以访问外部函数的变量对象。

- 释放对闭包的引用可以手动把这些变量设为null。如：Counter1 = null。

> 在知乎上看到一个回答：
> 通俗地讲就是别人家有某个东西，你想拿到但是因为权限不够（不打死你才怪），但是你可以跟家里的孩子套近乎，通过他拿到！这个家就是局部作用域，外部无法访问内部变量，孩子是返回对象，对家里的东西有访问权限，借助返回对象间接访问内部变量！
