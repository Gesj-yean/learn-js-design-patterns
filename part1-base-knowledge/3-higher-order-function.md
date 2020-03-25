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

那么使用函数柯里化可以做什么呢？

1. 参数复用

```base
function curryingCheck(reg) { // 将第一个参数reg进行复用，别的地方就可以直接调用hasNumber验证结果
    return function(txt) {
        return reg.test(txt)
    }
}

var hasNumber = curryingCheck(/\d+/g) // 调用起来更加只用传第二个参数，更加方便
var hasLetter = curryingCheck(/[a-z]+/g)

hasNumber('test1')      // true  
hasNumber('testtest')   // false
hasLetter('21212')      // false
```

- 反柯里化 uncurrying

反柯里化的作用在与扩大函数的适用性，使本来作为特定对象所拥有的功能的函数可以被任意对象所用。call和apply是最常见的应用场景之一。

- 函数节流 throttle

应用场景：

1. window.onresize事件。当给Window对象绑定了resize事件，浏览器窗口大小拖动时触发的频率非常高如果事件中有跟dom节点有关的操作，可能会非常消耗性能造成卡顿现象。
2. mousemove事件。移动端上下滑动屏幕时触发事件频率高，影响性能。
3. 上传进度。

因此我们会想到让原本要执行的事件fn延迟每隔一段时间500ms执行，如果在这个时间中有触发就忽略掉。简单版throttle函数实现：

```base
const throttle = (fn, delay=500) => { // 接收需要延迟执行的函数 延迟执行的时间间隔
    let _self = fn
    let timer = null
    let firstTime = true
    return (...args) => {
        _me = this
        if(firstTime) {               // 如果是第一次调用，直接执行
            _self.apply(_me, args)
            firstTime = false
        }
        if(timer) {                   // 如果定时器还在，上次延迟执行还没完成
            return false
        }
        timer = setTimeout(() => {
            clearTimeout(timer)       // 每次执行函数时需要先清除定时器
            timer = null
            _self.apply(_me, args)
        }, delay)
    }
}

window.onresize = throttle(()=> {
    console.log(1)
}, 1000)
```

- 函数防抖 debounce

应用场景：

1. 搜索框输入。不需要实时请求服务器获取数据，只需要在用户输入完成后的规定时间没有再次输入的话再去向服务器请求即可。
2. 验证用户名、手机号、邮箱等输入。

```base
const debounce = (fn, delay=500) => { // 接收需要延迟执行的函数 延迟执行的时间间隔
    let timer = null
    return (...args) => {
        _self = this
        if(timer) {
            clearTimeout(timer)
        }
        timer = setTimeout(() => {
           fn.apply(_self, args)
        }, delay)
    }
}

window.onresize = debounce(()=> {
    console.log(1)
}, 1000)
```

- 分时函数

分时函数也是js性能优化的一种方式。当我们需要向页面中添加大量的DOM节点，可能会导致浏览器卡顿甚至假死。

```base
let ary = [];

for ( let i = 1; i <= 1000; i++ ){      // 假设ary装载了1000个好友的数据
    ary.push( i );
};

const renderFriendList = function( data ){
    for ( let i = 0, l = data.length; i < l; i++ ){
        const div = document.createElement( 'div' );
        div.innerHTML = i;
        document.body.appendChild( div );
    }
};

renderFriendList( ary)
```

这个问题的解决方案之一是使用timeChunk函数，它让创建节点的工作分批进行。比如1s创建1000个节点改为每200ms创建8个节点。
timeChunk函数接收3个参数：节点数据；创建节点的逻辑函数；每批创建的节点数量。

```base
const timeChunk = function (ary, fn, count) {
    let t = null
    const len = ary,length
    const start = () => {
        for(let i = 0; i< Math.min(count || 1, ary.length); i++){
            const obj = ary.shift()
            fn(obj)
        }
    }

    return () => {
        t = setInterval(() => {
            if(ary.length === 0) {
                return clearInterval(t)
            }
            start()
        }, 200)
    }
}
```

试验一下：

```base
let ary = [];  
for ( let i = 1; i <= 1000; i++ ){
    ary.push( i );  
};  
const renderFriendList = timeChunk( ary, (n) => {
    var div = document.createElement( 'div' );
    div.innerHTML = n;
    document.body.appendChild( div );  
}, 8 );
```

- 惰性加载函数

在web开发中，因为浏览器之间的差异，一些嗅探工作不可避免。比如IE提供attachEvent接口，firefox提供了addEventLisenter。
常见的写法是每次用if( window.addEventListener )或 if( window.attachEvent)去判断，但是每次都要判断就很冗余，这时就可以采用惰性载入函数方案。
主要思想是第一次进入函数条件分支后在内部重写这个函数，重写后的函数就是我们期望的函数，下一次进入函数时，就不再存在条件分支语句。

```base
<html>
    <body>
        <div id="div1">点我绑定事件</div>
        <script>
            const addEvent = ( el, type, fn ) => {
                if ( window.addEventListener ){
                    addEvent = ( el, type, fn ) => {
                        console.log(11)
                        el.addEventListener( type, fn, false );
                    }
                } else if ( window.attachEvent ){
                    addEvent = ( el, type, fn ) => {
                        console.log(22)
                        el.attachEvent( 'on' + type, fn );
                    }
                }
                addEvent( el, type, fn );
            }
            var div = document.getElementById( 'div1' );
            addEvent( div, 'click', function(){
                alert (1);
            });
            addEvent( div, 'click', function(){
                alert (2);
            });
        </script>
    </body>
</html>
```

## 总结

- 明白高阶函数：函数可以作为参数被传递；函数可以作为返回值被输出。

- 提升js性能优化方面有关的函数：函数柯里化currying、函数节流throttle、函数防抖debounce、分时函数timeTrunk、惰性加载函数。
