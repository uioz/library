# 响应式对象的基本实现

```javascript
let Target = null;

function $watch(exp, fn) {
  Target = fn;
  data[exp];
}

function walk(obj) {

  for (const key of Object.keys(obj)) {
    
    const dep = [];
    // 事先缓存对于键的值, get set 获取以及修改的是这个局部变量
    // 而不是真正的 obj 上的属性, 避免递归调用导致的栈溢出
    let oldValue = obj[key];

    Object.defineProperty(obj, key, {
      set(newValue) {

        // 新旧数据相等直接返回
        if (newValue === oldValue){
          return;
        }

        oldValue = newValue;

        for (const fn of dep) {
          fn();
        }

      },
      get() {

        dep.push(Target)

        return oldValue;
      }
    })
  }

}
```

执行如下代码:

```javascript
const data = {
  a: 10
};

walk(data);

// 添加观察
$watch('a', () => {
  console.log('change1')
});

$watch('a', () => {
  console.log('change2')
});

// 修改数据触发观察
data.a = 20;
```

输出:

```
change1
change2
```

# 嵌套对象的响应式

上面的例子中只能监听简单对象, 对于嵌套对象无能为例:

```javascript
const data = {
  a: 10,
  b: {
    c: 10
  }
};

data.a = 20 // 可以触发
data.b.c = 20 // 没有效果
```

可以通过递归来为嵌套对象定义响应式属性:

```javascript
function isObject(value) {
  return Object.prototype.toString.call(value) === '[object Object]';
}

function walk(obj) {
  
  for (const key of Object.keys(obj)) {
    
    const dep = [];

    // 事先缓存对于键的值, get set 获取以及修改的是这个局部变量
    // 而不是真正的 obj 上的属性, 避免递归调用导致的栈溢出
    let oldValue = obj[key];

    // 如果该键对应的值是 object 那么递归定义
    if (isObject(oldValue)){
      walk(oldValue);
    }

    Object.defineProperty(obj, key, {
      set(newValue) {

        // 新旧数据相等直接返回
        if (newValue === oldValue) {
          return;
        }

        oldValue = newValue;

        for (const fn of dep) {
          fn();
        }

      },
      get() {

        dep.push(Target)

        return oldValue;
      }
    })
  }

}
```

然后升级我们的监听器添加方法允许它以 `data.b.c` 的方式进行监听:

```javascript
const data = {
  a: 10,
  b: {
    c: 10
  }
};

let Target = null;

function $watch(exp, fn) {

  Target = fn;
  let obj = data;

  // 将 a.b.c 转为 ['a','b','c']
  // a 转为 ['a']
  for (const key of exp.split('.')) {
    obj = obj[key];
  }

}
```

执行如下代码进行测试:

```javascript
walk(data);

// 添加观察
$watch('b', () => {
  console.log('change b')
});
$watch('b.c', () => {
  console.log('change b.c')
});
// 修改数据触发观察
data.b.c = 20;
```

输出:

```
change b.c
```

只有第二个监听器被调用了, 这也说明了为什么在 `Vue` 中这样的修改是不会触发响应式更新的:

```javascript
const vm = new Vue({
  data: {
    b: {
      c: 1
    }
  }
});

vm.b.c = 20; // 不会响应式更新视图
```

# 依赖收集

`$watch` 函数会通过获取对象属性来触发 `get` 拦截器, 然后将 `Target` 也就是监听器(函数)收集到该属性的依赖 `dep` 中.

如果视图层的渲染工作是一个函数, 当响应式对象改变后自动调用渲染函数, 该如何实现呢? 很简单, 让函数的首次渲染来触发依赖收集, 当数据发生变化再次调用渲染函数进行渲染就可以了, 例如有如下数据以及对应的渲染函数:

```javascript
const data = {
  i:'hello',
  you:''
};

function render() {
  console.log(`我说 ${data.i} 你说 ${data.you}`);
}
```

为了达到这种目的, 我们需要重写 `$watch` 让其允许接收函数作为参数:

```javascript
function $watch(exp, fn) {

  Target = fn;

  // 如果 exp 是函数, 调用函数, 让这个函数来触发依赖收集
  // 如果这个函数依赖的数据改变, fn 由于被收集到了依赖中
  // fn 如果是 exp 的话就会再次调用 exp ,请考虑 exp 是渲染函数的情况
  if(typeof exp === 'function'){
    exp();
  }else{
    let obj = data;

    // 将 a.b.c 转为 ['a','b','c']
    // a 转为 ['a']
    for (const key of exp.split('.')) {
      obj = obj[key];
    }
  }

  // 依赖收集完成后清空 Target 的引用
  // 避免重复收集
  Target = null;

}
```

现在我们把他们结合起来:

```javascript
walk(data);

$watch(render,render);
```

输出:

```
我说 hello 你说 
我说 hello 你说 world
```