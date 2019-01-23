
模块地址:
> https://github.com/taylorhakes/promise-polyfill

参考地址:
> https://segmentfault.com/a/1190000014368256

# 关于PromiseA+的几点有趣的事实

1. Promise的构造函数的resolve钩子不能返回自身.
```javascript
const pro = new Promise((resolve)=>{
    // 由于这里的代码是异步执行的原因
    // 所以向resolve传递的值不是undefined
    resolve(pro); // error
});
```
2. 调用then即返回一个新的Promise
```javascript
const pro = new Promise(()=>{});

console.log(pro === pro.then()); // false
```
3. Promise构造函数的回调中可以不调用任何钩子函数
但是你不可以不传入这个回调函数.
```javascript
new Promise(()=>{

}).then((result)=>{
    console.log(result === undefined) // true
});
```
4. A+规范只定义了Promise和then
准确的来说PromiseA+规范只定义了Promise和then本身的行为和状态.
没有描述catch以及Promise.all等常见方法.

# 分析

在`promise-polyfill`中Promise的回调函数执行和`then`部分填入的回调是分开处理的.

根据第二点的约束一旦调用了then即返回一个新的Promise,所以`promise-polyfill`中在一个Promise实例的内部会提供一个列表来保存这个Promise实例then方法产生的所有Promise.

## 思考

Promise内部逻辑很复杂,但是大致可以分为三个独立的逻辑部分.

1. 函数构造部分
2. then部分
3. 回调钩子和抛出异常的处理部分

这三个部分中最关键的核心在**回调钩子和抛出异常的处理部分**上.

## 构造函数部分

这部分主要完成两个步骤:
1. Promise内部状态初始化(主要完成了以下的几个功能)
   1. Promise内部的状态
   2. 是否绑定then
   3. resolve的返回值(默认undefined)
   4. 一个队列来保存所有的内容
2. 立即执行回调函数

初始化:
```javascript
function Promise(fn) {
    // 根据A+规范传入的内容必须是函数
    if (!(this instanceof Promise))
    throw new TypeError('Promises must be constructed via new');
    if (typeof fn !== 'function') throw new TypeError('not a function');

    // 创建内部状态
    this._state = 0;
    this._handled = false;
    this._value = undefined;
    this._deferreds = [];

    // 立即执行这个回调函数
    doResolve(fn, this);
}
```

立即执行函数(有删减):
```javascript
function doResolve(fn, self) {
    // 防止多次调用
    var done = false;
    try {
    fn(
        function(value) {
        if (done) return;
        done = true;
        // 处理resolve函数钩子
        },
        function(reason) {
        if (done) return;
        done = true;
        // 处理reject函数钩子
        }
    );
    } catch (ex) {
    if (done) return;
    done = true;
    // 执行错误
    // 当作reject处理
    }
}
```

到目前为止内容都是同步的,但是不要忘记了即使我们Promise中的内容的同步的但是结果返回是异步的.

示例:
```javascript
new Promise(resolve => {
    console.log('sync')
    resolve('async');
}).then(result => console.log(result));
console.log('sync');
```
输出:
```
sync
sync
async
```

关于`promise-polyfill`如何完成这个功能的请看第三部分,到目前为止这就是Promise构造函数完成的任务.

## then部分

then的规则看起来有很多实际上它工作起来非常简单.

1. 创建一个对象然后将一个新Promise和then挂载上去
2. 将这个包裹添加到内部的列表中然后返回新的Promise

```javascript
Promise.prototype.then = function(onFulfilled, onRejected) {

    // 创建一个空的Promise
    // prom === new Promise(()=>{})
    var prom = new this.constructor(noop);

    // 将三者进行绑定然后填入到内部的列表中
    handle(this, new Handler(onFulfilled, onRejected, prom));

    // 返回这个新的Promise
    return prom;
};
```

这样一来即使返回了一个新的Promise,原来的Promise也可以拿到新Promise的引用.

到目前为止这就是then完成的所有功能.

## 回调钩子和抛出异常的处理部分

念起来比较拗口,实际上理解起来也比较麻烦,还是按照不同情况来说明.

首先我们回顾一下,如果调用了resolve或者reject钩子以及在我们提供的回调函数中抛出异常会出现什么情况.

```javascript
new Promise((resolve,reject)=>{

    resolve()
    reject()
    throw new Error();

});
```

这三个一旦其中有一个触发则会让Promise去寻找对应的处理钩子.

根据第一部分的代码我们知道实际上Promise**会同步执行**我们传入的函数钩子,这意味着:
```javascript
new Promise((resolve)={
    resolve('hello world');
}).then((result)=>{
    console.log(result);
});
```
回调函数会先于then提供的钩子执行.

但是在使用者的角度来看即使我们提供的是同步代码但是执行依然是异步的,Promise在内部是如下处理的:

1. 如果传入的回调是同步执行
   1. 获取返回结果保存在Promise内部
   2. 根据回调的执行情况来修改内部的状态
2. 如果回调代码是异步执行则then方法会早于回调函数执行
   1. 则将then包装好的内容添加到内部的队列中

`promise-polyfill`内部处理resolve钩子的情况(有删减):
```javascript
// 我们调用resolve时候同步触发
// self就是Promise实例本身
// newValue就是使用者调用resolve传入的值
function resolve(self, newValue) {
  try {
    self._state = 1;
    self._value = newValue;
    finale(self);
  } catch (e) {
    reject(self, e);
  }
}
```

`promise-polyfill`内部处理reject钩子的情况:
```javascript
// 我们调用reject时候同步触发
// self就是Promise实例本身
// newValue就是使用者调用reject传入的值
function reject(self, newValue) {
  self._state = 2;
  self._value = newValue;
  finale(self);
}
```

可以看到`promise-polyfill`在内部处理的回调钩子的时候修改了自己的内部状态.

而最终调用的`finale`函数内部会延时触发:
```javascript
function finale(self) {
    // 执行错误且没有提供catch异步抛出异常(没有提供错误钩子)
    if (self._state === 2 && self._deferreds.length === 0) {
        Promise._immediateFn(function() {
            if (!self._handled) {
            Promise._unhandledRejectionFn(self._value);
            }
        });
    }

    // 执行then向内部添加的Promise
    // 如果我们的回调函数是同步执行则_deferreds还没有then添加的内容
    // 如果我们的回调函数是异步执行则此时的_deferreds中已经有了内容
    for (var i = 0, len = self._deferreds.length; i < len; i++) {
        handle(self, self._deferreds[i]);
    }
    self._deferreds = null;
}
```

如果你还记得之前then的源码你会发现无论是同步触发钩子,
还是调用then添加对应的处理函数都会调用一个函数(有删减),
这个函数用户触发then提供的钩子.
```javascript
// self即Promise本身
// deferred 调用then方法后内部创建的对象
function handle(self, deferred) {

    // 关键,如果回调函数异步执行则状态还未改变
    // 将deferred添加到内部的数组中
    // 实际上这个分支就是给then准备的
    if (self._state === 0) {
        self._deferreds.push(deferred);
        return;
    }

    // 下面的分支给回调函数触发

    // 表示已经添加了then回调函数进行处理
    self._handled = true;
    
    // 异步处理
    Promise._immediateFn(function() {
        var cb = self._state === 1 ? deferred.onFulfilled : deferred.onRejected;
        if (cb === null) {
            (self._state === 1 ? resolve : reject)(deferred.promise, self._value);
            return;
        }
        var ret;
        try {
            ret = cb(self._value);
        } catch (e) {
            reject(deferred.promise, e);
            return;
        }
        resolve(deferred.promise, ret);
    });
}
```

不同的是resolve和reject以及回调中的错误触发这个handle函数是延时执行的(伪代码):
```javascript
setTimeout(() => {
  handle();
}, 0);
```
之所以需要延时执行是为了获取then方法提供的处理钩子,下面这个例子可以看到then方法要早于我们的回调中的调用(伪代码):
```javascript
function handle(){

}

new Promise((resolve)=>{

    setTimeout(()=>{
        resolve();
        handle();
    },0);

}).then(()=>{
    handle();    
});
```

# 值得回味的部分

实际上这里的代码十分值得我们的学习,他巧妙的控制了对象以及函数之间的内存平衡而且最大限度的复用了函数.

## 1. 公用的无状态的函数

在`promise-polyfill`中有很多复用的函数,他们依赖传递上下文的方式:
```javascript
function(self,value){

}
```
从而减少了对象上的开销,但是增加了闭包开销.  
不过考虑到JIT的优化,对于我这个只知道皮毛的人来说,这其中的奥秘还需要更长时间才能解开.

## 2. 低开销的对象

实际上这点和上点是平衡的,因为复用了函数所以建立的方法是低开销的.

仔细观察一下这个Promise实例上的方法,他们内部实际上都利用到了外部的无状态函数,而不是内部的方法.

当然在日常的编码中这么做未免有点过火,但是在Prototype上定义的方法是公用的,所以平常我们没有必要定义大量的外部函数
来如此节约性能.

## 3. 短路运算符

短路运算符我经常使用但是还没有作者玩的这么骚气:
```javascript
Promise._immediateFn =
  (typeof setImmediate === 'function' &&
    function(fn) {
      setImmediate(fn);
    }) ||
  function(fn) {
    setTimeoutFunc(fn, 0);
  };

var cb = self._state === 1 ? deferred.onFulfilled : deferred.onRejected;
if (cb === null) {
    (self._state === 1 ? resolve : reject)(deferred.promise, self._value);
    return;
}
```
## 4. 及时回收的内存

在finale中作者在执行完成then提供的Promise后会及时的清空引用.

```javascript
  for (var i = 0, len = self._deferreds.length; i < len; i++) {
    handle(self, self._deferreds[i]);
  }
  self._deferreds = null;
```

## 5. 高性能异步

这里使用了setImmediate而不是setTimeout(当然是在支持setImmediate环境下).

> https://www.aliyun.com/jiaocheng/1068535.html

