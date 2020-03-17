# 模拟 JavaScript 中的基础操作

## 模拟 call

```javascript
Function.prototype.fakeCall = function fakeCall(context) {

  // 牢记当前 this 的指向是调用者
  // context 是 xxx.call(context) 中的 context

  // 当 context 为 null 是指向 window
  context = context || window;
  context.fn = this;

  var args = [];
  for (var i = 1; i < arguments.length; i++) {
    args.push("arguments[" + i + "]");
  }

  var result = eval("context.fn(" + args + ")");

  delete context.fn;

  return result;
};

var foo = {
  value: 1
};

function bar() {
  console.log(this.value);
  console.log(arguments);
}

bar.fakeCall(foo, "hello world");
```

## 模拟 apply

```javascript
Function.prototype.fakeApply = function fakeApply(context, array) {

  // 牢记 this 指向的是函数调用者
  // 牢记 context == xxx.fakeApply(context) 中的 context
  
  context = context || window;

  context.fn = this;

  var result;

  if (!array) {
    result = context.fn();
  } else {
    var args = [];

    for (var i = 0; i < array.length; i++) {
      args.push("array[" + i + "]");
    }

    result = eval("context.fn(" + args + ")");
  }

  delete context.fn;
  return result;
};

var foo = {
  value: 1
};

function bar() {
  console.log(this.value);
  console.log(arguments);
}

bar.fakeApply(foo, ["hello world"]);
```

## 模拟 bind

```javascript
Function.prototype.fakeBind = function fakeBind(context) {
  // 牢记 this 指向的是函数调用者
  // 牢记 context 是 xxxx.bind(context) 中的 context

  var that = this;
  var args = Array.prototype.slice.call(arguments, 1);

  function fbound() {
    var bindArgs = Array.prototype.slice.call(arguments);

    /**
     * 当 new xxx.fakeBind(context) 的时候等于 nwe fbound
     * 而函数被当作 "构造函数" 的时候 this 所指向的对象已经和 fbound.prototype 建立了联系
     * 所以 this instanceof fbound 会返回 true
     * 反之就视为普通的函数调用即 xxx.fakeBind(context)()
     */
    return this instanceof fbound
      ? this
      : that.apply(context, args.concat(bindArgs));
  }

  // 最后不要忘记了原型绑定此处的 this.prototype
  // 是 xxx.fakeBind(context) 中的 xxx.prototype
  // 因为在 new xxx.fakeBind(context) 和 new xxx 表现一致
  // 会忽略 context 但是会保留 bind 中提供的参数例如
  // xxx.fakeBind(context,arg1)(arg2) === new xxx(arg1,arg2)
  function middle (){}

  middle.prototype = this.prototype;

  // 我们使用 middle 做了一层中转
  // 如果直接使用 fbound.prototype = this.prototype
  // 会导致修改 new xxx.fakeBind() 返回的原型的同时
  // 会修改 this.prototype 因为返回的 fbound 与 this.prototype 相等
  fbound.prototype = new middle;

  return fbound;
};
```

## 模拟 new

```javascript
function ObjectFactory() {
  const obj = Object();

  const constructor = Array.prototype.shift.call(arguments);

  Object.setPrototypeOf(obj, constructor.prototype);

  // 构造函数执行后返回任何继承自 Object 的值
  // 都会被返回, 所以我们要判断执行后是否有返回的内容
  // 如果有我们就返回这个结果
  const result = constructor.apply(obj, arguments);

  if (result instanceof Object) {
    return result;
  }

  return obj;
}
```

# 防抖 debounce

 基本操作之一 bounce(跳动) de(表否定) debounce(防抖).

```javascript
function debounce(fun, wait) {
  let timer;

  return function() {
    if (timer) {
      clearTimeout(timer);
    }

    timer = setTimeout(() => {
      fun.apply(this, arguments);

      timer = undefined;
    }, wait);
  };
}
```

# 节流

```javascript
function throttle(fun, interval) {
  let timeStamp = Date.now();

  return function() {
    let now = Date.now();

    if (now - timeStamp >= interval) {
      fun.apply(this, arguments);
      timeStamp = now;
    }
  };
}
```

