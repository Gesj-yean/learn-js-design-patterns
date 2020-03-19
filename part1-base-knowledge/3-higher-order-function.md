# 高阶函数

## 3.2 高阶函数

高阶函数需要满足两个条件中一项：函数可以作为参数被传递；函数可以作为返回值被输出。

### 3.2.1 函数作为参数被传递

- 回调函数

函数A被作为实参传入另一函数B，并在B内调用A，用来完成A中的某些任务。A被称为回调函数。

```base
const A = function(res) {
    console.log(res)
}
const B = function(callback) {
    let res = 'res'
    callback(res)
}
B(A) // res
```

回调函数不仅用于异步操作中，当一个函数不适合执行一些请求时，可以把这些请求封装成一个函数再坐为参数传递给另一个参数，委托另一个函数执行。

举一个实际应用：
在小程序的页面中我们需要调用getLatest去赋值请求的结果：

```base
getLatest((res) => {
    this.setData({
        data: res
    })
})
```

实际上，getLatest做了更多的操作（请求数据）：

```base
getLatest(sCallback) {
    this.request({
        url: '/classic/latest',
        success:((res) => {
            sCallback(res) // 利用回调函数sCallback传递结果
            // 其他业务操作
        })
    })
}
```

- Array.prototype.sort

```base
[1,4,3].sort((a,b) => {
    return a-b
})

// [1, 3, 4]
```

### 3.2.2 函数作为返回值输出

- 判断数据的类型

```base
var isType = function( type ){
    return function( obj ){
        return Object.prototype.toString.call( obj ) === '[object '+ type +']';
    }
};
var isString = isType( 'String' );
var isArray = isType( 'Array' );
var isNumber = isType( 'Number' );
console.log( isArray( [ 1, 2, 3 ] ) );     // 输出：true，等价于isType('Array')([ 1, 2, 3 ])
```

### 高阶函数其他应用

- 柯里化currying

currying又称部分求值，将一个接受多个参数的函数转化为接受单一参数的函数的技术。

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
curryingAdd(1)(2)   // 3 先用一个函数接收x然后返回一个函数去处理y参数
```

参考：[详解JS函数柯里化](https://www.jianshu.com/p/2975c25e4d71)

- 反柯里化 uncurrying

- 函数节流

```base
const throttle = (fn, delay=500) => {
    let flag = true
    return (...args) => {
        if(!flag) return
        flag = false
        setTimeout(() => {
            fn.apply(this, args)
        }, delay)
    }
}
```

- 函数防抖

```base
const debounce = (fn, delay) => {
    let timer = null
    return (...args) => {
        clearInterval(timer)
        timer = setInterval(() =>{
           fn.apply(this, args)
        }, delay)
    }
}
```

- 分时函数

- 惰性加载函数

（更新中）
