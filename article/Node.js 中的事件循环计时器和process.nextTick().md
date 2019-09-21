# 前言

本篇文章翻译自 Node.js 官网的同名文章也算是经典老物了, 不过官网的文章也随着 Node.js 的演化在修改, 这篇文章最后的编辑时间是 2019年9月10日请注意时效性, 地址在文章的最后有给出.

首次翻译英语水平有限, 错误之处还请多多指教.

**注意**: 这里描述的事件循环的工作机制是 Node11 后的工作机制和之前的版本不同, 详情可以查看[这篇文章](https://zhuanlan.zhihu.com/p/54951550).

# 什么是事件循环

事件循环允许node.js执行非阻塞I/O操作. 虽然 JavaScript 是单线程的, 但是事件循环会尽可能的将操作转移到系统内核中来完成.

现代的操作系统内核都是多线程的, 它们可以在后台处理多种操作. 一旦这些操作完成, 系统内核会通知 Node.js 以便将事件回调放入轮询队列中等待执行. (我们会在随后的内容讨论它们的具体工作细节)

# 解析事件循环

当 Node.js 启动的时候, 他会初始化事件循环, 处理输入的脚本内容 (或者进入 [REPL](https://nodejs.org/api/repl.html#repl_repl)), 脚本可能会调用异步接口, 设置定时器, 或者调用 `process.nextTick()`, 然后开始处理事件循环(eventloop).

下面的简图中展示了事件循环的操作流程:

```
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

> 每一个方框代表了事件循环中不同的阶段(所有阶段执行完成算是一次事件循环).

每一个阶段都有一个由回调组成的 FIFO 队列被用于执行. 虽然不同的队列执行方式不同, 总的来看, 当事件循环进入该阶段后会执行该阶段对应的操作, 然后调用对应的回调直到队列耗尽或者达到了回调执行上限. 在到达上述情况后事件循环进入下一阶段, 然后继续这样的流程.

由于处理单个操作可能会产生新的操作以及在轮询阶段产生的新事件会被内核排队, 在轮询事件(poll events)的过程中轮询事件会被排队. 因此, 执行一个长耗时的回调会超出在轮询阶段设定的定时器的阈值.

> *Windows and the Unix/Linux* 平台略有差别, 但是这不影响我们的讨论. 我们最关心的是 Node.js 实际执行的那部分也就是上面的内容.

# 阶段总览

- timer: 此阶段执行由 `setTimeout()` 和 `setInterval()` 设定的回调.
- pending callbacks: 执行被推迟到下一轮循环的 I/O 回调.
- idle, prepare: 仅内部使用.
- poll: 获取新的I/O事件; 执行 I/O 回调(除了 close 回调以及 timer 回调和 `setImmediate` 回调都会在这里执行), node会在适当条件下在这里阻塞.
- check: `setImmediate` 回调将会在次执行.
- close callbacks: 一些执行关闭的函数, 例如 `socket.on('close', ...)`.

Node 会在两次完整的事件循环间检查是否存在 I/O 操作和或者 timer, 如果没有就会退出执行.

# 各阶段中的细节

## timer

timer(计时器) 指定了执行给定回调的阈值时间, 而不是人们所想的准确执行时间. 定时器回调将会在指定的时间到达后尽快的执行, 不过 timer 的执行会受到操作系统调度和其他回调执行的影响被延后.

> 从技术上讲, 决定是否执行 timer 回调是在轮询阶段控制的, 在 timer 阶段才会执行这些回调.

举例来说, 你制定了一个延时 100ms 的 timer, 然后异步进行读取文件花费了 95ms:

```javascript
const fs = require('fs');

function someAsyncOperation(callback) {
  // Assume this takes 95ms to complete
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);


// do someAsyncOperation which takes 95 ms to complete
someAsyncOperation(() => {
  const startCallback = Date.now();

  // do something that will take 10ms...
  while (Date.now() - startCallback < 10) {
    // do nothing
  }
});
```

当事件循环进入轮询队列后, 此时队列是空的(`fs.readFile()` 还未完成), 现在我们等待计时器到达指定的阈值. 过了 95ms 后 `fs.readFile` 读取完毕并且执行回调共花费 10ms. 当回调执行完成, 轮询队列中没有任何内容了, 此时事件循环会看到已经到达阈值的 timer, 然后在 timer阶段去执行回调. 所以在这个例子中的延时函数会在 105ms 后执行.

> 为了防止事件循环被长时间空置,  [libuv](https://libuv.org/) 有一个最大限值(取决于操作系统)用于限制轮询队列的执行次数.

## pending callbacks

系统操作例如: TCP类型错误执行回调会安排在这个阶段执行. 例如当尝试 TCP 连接的时候接收到了一个 `ECONNREFUSED` 错误, 有些 *nix 系统会进行等待而不是立即抛出错误. 这些回调会被添加到队列中在 `pending callbacks` 阶段执行.

## poll

事件轮询阶段主要有两大功能:

1. 计算需要阻塞多长时间, 并且进行I/O轮询, 然后
2. 处理轮询队列中的事件

当事件轮询到了 poll 阶段的时候发现没有计时器到达阈值, 此时会发生两种情况:

1. 如果轮询队列中有内容, 事件循环会遍历轮询队列然后同步调用其中的回掉, 直到队列清空或接近轮询阶段的回调执行上限(上限取决于操作系统).
2. 如果轮询队列为空, 此时
   - 如果存在 `setImmediate()` 任务, 事件循环会结束轮询阶段直接跳入 check 阶段去执行那些 `setImmediate()` 任务.
   - 如果没有需要处理的 `setImmediate()` 任务, 事件循环会在轮询阶段等待新的任务被添加到轮询队列中, 然后立即处理这些添加进来的任务.

轮询队列为空后, 事件循环将检查已达到时间阈值的计时器. 如果有计时器到达阈值, 事件循环会移动到 timer 阶段然后执行那些计时器回调.

## check

这个阶段允许在轮询阶段完成后执行回调. 如果轮询阶段进入等待, 并且有被 `setImmediate()` 设定的回调, 那么事件循环有可能会移动到 check 阶段而不是继续在轮询阶段等待.

`setImmediate()` 实际上是一个特殊的计时器, 在事件循环的一个单独阶段中执行. 它通过 libuv API 在轮询阶段结束后执行由 `setImmediate()` 设定的回调.

通常来说, 随着代码的运行事件循环终将进入事件轮询阶段并在此等待连接的传入或者请求等. 但是如果存在使用 `setImmediate()` 设定的任务且时间轮询进入了等待(idle 阶段), 事件循环会进入到 check 阶段而不是继续等待下去.

## close

如果 socket 或者 handle 突然的关闭, 它们的 `close` 事件会在这个阶段执行. 否则它会经由 `process.nextTick()` 执行.

# `setImmediate()` vs `setTimeout()`

`setImmediate()`  和 `setTimeout()` 很像, 但根据调用时机的差异它们的行为方式有所区别.

- `setImmediate()` 被设计在当前的事件轮询阶段(poll phase)结束后执行脚本一次.
- `setTimeout()` 借助于设定阈值(毫秒)规划脚本的执行.

执行计时器的顺序将根据调用它们的上下文而有所不同. 如果两者都在主模块中运行, 执行的时机会受到进程性能的影响(机器上的其他程序会影响到进程的性能).

例如我们在不受 I/O 循环(例如主模块中)的地方执行下方的代码, 这两个计时器的执行顺序是不确定的, 因为会受到进程性能的影响:

```js
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
```

```bash
$ node timeout_vs_immediate.js
timeout
immediate

$ node timeout_vs_immediate.js
immediate
timeout
```

**译者说明**: 在脚本执行首次执行完成后, `setImmediate` 和 `setTimeout` 被添加到了事件循环中. 在第二轮事件循环中如果进程性能一般已经到达 timer 的阈值了就会在 timer 阶段执行定时器任务, 随后执行 `setImmediate` 设定的任务. 如果线程性能足够就会因为不够计时器阈值跳过 timer 阶段去执行 `setImmediate` 设定的任务.

但是如果你将这两个计时器移动到 I/O 循环中, `setImmediate` 始终会第一个执行:

```javascript
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});
```

```bash
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

**译者说明**: 文件操作是 I/O操作实在 poll 阶段执行的, 回调执行完成后 poll 队列是空着的, 此时 timer 已经在 poll 阶段被设定完成(timer 阶段执行), 此时存在 `setImmediate`  任务所以直接进入到了 check 阶段.

使用 `setImmediate`  的优点是始终在定时器前执行(在 I/O循环中), 而不管设置了多少个定时器.

# `process.nextTick()`

你可能注意到了 `process.nextTick()` 没有出现在之前的图中, 虽然它是异步 API 的组成部分. 从技术角度来看 `process.nextTick()` 并不是事件循环的一部分. `nextTickQueue` 总是在当前操作执行完成后执行

> 水平有限, 有关 "操作" 的定义在原文如下:
>
>  Here, an *operation* is defined as a transition from the underlying C/C++ handler, and handling the JavaScript that needs to be executed.

回到刚才的流程图上， 你可以在图上的任意阶段执行 `process.nextTick()`, 所有通过 `process.nextTick()` 注册的回调都会在事件循环进入到下一个阶段前处理. 这种设计会造成一些不好的情况, 如果你递归调用 `process.nextTick()` 他会 "饿死" I/O, 因为这会阻止事件循环进入到事件轮询阶段.

为什么允许这样的设计？

为什么这样的设计被包含到了 Node.js 中？这是 Node.js 设计理念的一部分, 接口永远应该是异步的即使是它同步也没有问题, 举例来说:

```javascript
function apiCall(arg, callback) {
  if (typeof arg !== 'string'){
    return process.nextTick(
	    	callback,
	    	new TypeError('argument should be string')
    	);
  }
}
```

这段代码会对参数进行检查当类型错误会抛出 error. `process.nextTick()` 在最近的更新中允许传入参数, 然后将参数传入到回调中而不必嵌套一个函数来包装实现类似的功能.

上段代码中我们会向用户通知错误, 但是只有用户的代码执行完成后这个错误才会被执行.  借助于 `process.nextTick()` 我们可以确保 `apiCall()` 调用的 `callback` 永远在当前用户代码执行完成之后以及在事件循环进入下一阶段前执行代码. 为了达到这一点, JS 调用栈允许展开后立即执行那些给定的回调, 这样做允许用户通过  `process.nextTick` 创建递归的代码但是不会造成 V8 引擎的栈溢出错误 `RangeError: Maximum call stack size exceeded from v8`.

> 水平有限, 原文如下:
>
>  To achieve this, the JS call stack is allowed to unwind then immediately execute the provided callback which allows a person to make recursive calls to `process.nextTick()` without reaching a `RangeError: Maximum call stack size exceeded from v8`.

这种理念可能会导致问题出现, 举例来说:

```javascript
let bar;

// 这是拥有异步接口设计的函数, 在其内部确实同步的
function someAsyncApiCall(callback) { callback(); }

// 内部的回调会在 someAsyncApiCall 执行完成前调用
someAsyncApiCall(() => {
  // 由于 someAsyncApiCall 立即执行, 此时的 bar 还未被指定值
  console.log('bar', bar); // undefined
});

bar = 1;
```

用户定义了一个拥有异步接口的函数, 但是内部却是同步的. 当提供给 `someAsyncApiCall` 的回调执行后, 回调和 `someAsyncApiCall`  在事件循环的执行阶段是一样的, 因为 `someAsyncApiCall` 本质上没有做任何的异步操作. 结果是回调试图引用 `bar` 即使在作用域中还没有该变量的值, 因为脚本还未全部解析完成.

通过将回调放入到 `process.nextTick()`, 其余的脚本才会有机会执行完成, 解析所有的函数和变量等等, 以在回调调用前初始化. 在事件循环进入到下一阶段前收到错误也是非常有用的.

这里有一个先前使用 `process.nextTick()` 的示例:

```javascript
let bar;

function someAsyncApiCall(callback) {
  process.nextTick(callback);
}

someAsyncApiCall(() => {
  console.log('bar', bar); // 1
});

bar = 1;
```

这里还有一个实际的应用示例:

```javascript
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```

当端口被传入后, 端口被立即绑定. 所以 `listening` 可能会立即执行. 但问题是此时 `.on('listening')` 回掉还未被注册.

通过使用 `nextTick()` 来将内部的 `listening` 事件排队, 让脚本有机会去执行完成. 这才可以让用户去注册它们想要的监听器.

# `process.nextTick()` vs `setImmediate()`

对于使用者来说这两个接口的功能是类似的, 但是它们的名称却令人难以琢磨.

- `process.nextTick()` 在事件循环的某个阶段中全部执行
- `setImmediate()` 在事件循环的随后的迭代中触发

> 原文:
>
> - `process.nextTick()` fires immediately on the same phase
> - `setImmediate()` fires on the following iteration or 'tick' of the event loop
>

从本质上看, 它们应该交换名称. `process.nextTick()` 从调用到出发所花费的时间比 `setImmediate()` 还要短, 但是这个坑已经被埋了太久了很难再被修复了. 如果要是修改命名会让 npm 上的大部分包挂掉. 随着 npm 上的包越来越多尝试修复的代价也越来越高. 虽然命名有问题, 但是也无法修改了.

我们开发者在所有的情况下都使用 `setImmediate` 因为它更加容易推理(也可以让代码更具兼容性, 比如在浏览器中运行).

# 为什么使用`process.nextTick()`?

主要原因有两个:

1. 运行用户处理错误, 清理不需要的资源, 或者在事件循环进入下一阶段前尝试再次发送请求.
2. 有时需要回掉在栈展开(unwind)后但是事件循环还未进入到下一阶段前执行.

有一个符合用户预期的例子:

```javascript
const server = net.createServer();
server.on('connection', (conn) => { });

server.listen(8080);
server.on('listening', () => { });
```

假设事件循环中第一个运行的是 `listen()`, 但是用于监听的回调是使用 `setImmediate` 设置的. 除非主机名称已经被传入, 否则将立即绑定到端口. 要使事件循环继续, 它必须进入到轮询阶段. 这意味着在 `listening` 前建立的连接会在 `listening` 事件触发前执行 `connection` 事件.

另一个例子是运行一个函数构造函数, 继承自 `EventEmitter` 并且它想在构造函数中调用一个事件.

```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);
  this.emit('event');
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```

你无法在构造函数中立即触发事件, 因为对应的事件监听器还未挂载. 通过使用 `process.nextTick()` 可以在构造函数执行完成后在触发事件, 就可以实现我们的目标了:

```javascript
const EventEmitter = require('events');
const util = require('util');

function MyEmitter() {
  EventEmitter.call(this);

  // use nextTick to emit the event once a handler is assigned
  process.nextTick(() => {
    this.emit('event');
  });
}
util.inherits(MyEmitter, EventEmitter);

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
```

# 参考

> [The Node.js Event Loop, Timers, and process.nextTick()](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#process-nexttick-vs-setimmediate)

> [由setTimeout和setImmediate执行顺序的随机性窥探Node的事件循环机制](https://segmentfault.com/a/1190000013102056)

> [Node探秘之事件循环（2）--setTimeout/setImmediate/process.nextTick的差别](https://www.jianshu.com/p/837b584e1bdd)

> [Node 定时器详解](http://www.ruanyifeng.com/blog/2018/02/node-event-loop.html)

> [nodejs的eventloop，timers和process.nextTick()【译】](https://www.jianshu.com/p/ac64af22d775)