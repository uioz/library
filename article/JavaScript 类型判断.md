# JavaScript 类型判断

通过 typeof 识别的基本类型有:

```javascript
console.log(typeof 123); // number
console.log(typeof '123'); // string
console.log(typeof true); // boolean
console.log(typeof {}); // object
console.log(typeof undefined);// undefined
console.log(typeof Symbol.for(123));// symbol
console.log(typeof BigInt('123'));// bigint
```

另外 null 也是基本类型, 只不过无法通过 typeof 来进行识别, 它识别的结果是 object.

# 基本类型转换

类型转换的操作实际上都是 JavaScript 引擎来完成的, 在 JavaScript 语法层面上, 无论是隐式类型转换还是显式类型转换的本质都是要求 Javascript 引擎去做类型处理.

隐式的转换存在一套看不见的转换过程, 而显式的转换容易理解, 显式的转换背后遵循了几种抽象操作:

- ToNumber
- ToBoolean
- ToString

所谓的 "抽象操作" 实际上是 JavaScript 引擎在背后执行的一套类型转换的逻辑.

另外 JavaScript 中存在 "装箱" 和 "拆包" 的概念, 即在对基本类型做方法调用的时候, JavaScript 引擎会使用对应的类型对象进行包装(装箱), 藉此提供对象的能力. 但是对与对象类型执行类型转换的时候, 会执行拆箱操作, 这个抽象操作被称为 ToPrimitive, 会尝试执行以下操作:

1. 调用对象的 valueOf 方法来获取 "基本类型"
2. 调用 toString 方法来获取对应的字符串(字符串是基本类型)

基本的原则是找到对象对应的基本类型, 然后在执行类型转换.

举个例子:

```
['123'] - 1 // 122
```

为什么会这样, 在 JavaScript 被用于数字的减法, `['123']` 会被执行类型转换操作:

1. 调用 valueOf 方法返回的依然是数组
2. 对该值调用 toString 方法转为字符串 '123'
3. 将 '123' 转为数字 -> 123
4. 执行减法 123-1
5. 获得 122

而另外一方面如果使用加号:

```
['123'] + 1 // 1221
```

为什么因为 + 的两边并不全是数值, 所以 + 会尝试进行字符串连接:

1. [][]调用 valueOf 方法返回的依然是数组
2. 对该值调用 toString 方法转为字符串 '123'
3. 执行加法 '123' + 1
4. '1231'

