# 简介(题目自底向上学习Web数据操作)

本篇文章主要探讨JavaScript中的数据操作.

JavaScript一直以来给人一种比较低能的感觉,例如无法读取系统上的文件,不能做一些底层的操作.

所以在页面上操作数据会交由服务器处理也就成了主流的做法.

但是很多人没有发现,实际上JavaScript以及在逐步增强这些功能,例如我们已经可以只在浏览器端进行读写文件了.

我个人认为这些功能不流行的原因,除了需求较小外,兼容性,以及不利于理解的API甚至没有过多介绍的文章都在客观上造成了某些影响.

# 涉及的内容

没有非必要的内容,对于文件操作来说以下API都是必须了解的,本文也会渐进式的讨论这些内容.

- Blob
- ArrayBuffer
- TypedArray
- DataView
- FileReader
- File
- URL

# 兼容性

我没有详细考证API的兼容性,不过从MDN提供的数据来看IE10以上的浏览器大部分都是兼容的.

# 总览

一般来说操作一个文件都要经历如下的步骤:
- 知道文件的地址(存放的位置)
- 读取
- 保存到Buffer中,重复上步骤直至结束
- 进行数据编辑
- 知道要写入的地址
- 获取要写入的数据,从Buffer中获取还是所有数据
- 写入
- 写入完成

API名称以及对应的职责:
| 名称 | 职责 |
| ---- | ---- |
| URL  | 制造文件地址 |
| FileReader | 读取文件的接口 |
| Blob | 用于在JavaScript表示文件(本身不包含文件的内容) |
| File | 用于表示文件对象(包含文件内容) |
| ArrayBuffer | 表示Buffer(仅仅提供一片内存空间) |
| TypedArray | 基于数组操作Buffer上的数据(操作的最小单位是数组元素) |
| DataView | 基于字节操作Buffer上的数据 |

上面描述的内容之间的关系很复杂,这里我们逐步来进行分析.

# ArrayBuffer

> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

ArrayBuffer对象用于表示一段缓冲区域(可以理解为一段可控的内存区域),它仅仅表示这片被开辟的区域但是不提供操作方式.

```javascript
const arraybuffer = new ArrayBuffer(8) // 创建一个长度为8字节大小的Buffer
```
默认ArrayBuffer中每一个字节都被填充了0.

利用这个对象我们可以完成如下的操作:
- 获取
  - 该Buffer的大小(字节)
  - 该Buffer的副本(范围)
- 修改
  - 该Buffer的大小
- 判断
  - 给定的数据是否是操作视图(实例方法)
- 异常
  - 当创建的Buffer长度超过`Number.MAX_SAFE_INTEGER`的大小会产生错误

```javascript

const arraybuffer = new ArrayBuffer(8);

console.log(arraybuffer.byteLength); // 获取长度
console.log(arraybuffer.slice(4,8)); // 获取副本
// 截止到2019年2月12日 20:11:05没有浏览器实现该功能
console.log(arraybuffer.transfer(arraybuffer,16));// 修改原有Buffer
console.log(ArrayBuffer.isView({})) // false 是否是视图

```

# DataView
> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView

DataView用于操作ArrayBuffer中的数据,这也是它构造函数中接受一个ArrayBuffer的原因:
```javascript
const arraybuffer = new ArrayBuffer(8);
const dataview = new DataView(arraybuffer); // 默认的视图大小就是buffer的大小
const offset = new DataView(arraybuffer, 0, arraybuffer.byteLength); // 默认的偏移量以及长度
```
利用这个对象我们可以完成如下的操作:
- 获取
  - 被该视图引入的Buffer(只读)
  - 该视图从Buffer中读取的自己长度(只读)
  - 该视图从Buffer中读取的偏移量(只读)
- 异常
  - 如果由偏移（byteOffset）和字节长度（byteLength）计算得到的结束位置超出了 buffer 的长度.
- 写入
  - 使用xxx类型写入(见下方)
- 读取
  - 使用xxx类型读取

可以使用的类型:

| 类型名称 | 对应的方法            |
| -------- | --------------------- |
| Int8     | getInt8,setInt8       |
| Uint8    | getUint8,setUint8     |
| Int16    | getInt16,setInt16     |
| Uint16   | getUint16,setUint16   |
| Int32    | getInt32,setInt32     |
| Uint32   | getUint32,setUint32   |
| Float32  | getFloat32,setFloat32 |
| Float64  | getFloat64,setFloat64 |

简单实例:

```javascript
        const arraybuffer = new ArrayBuffer(1); // 一个字节
        const dataview = new DataView(arraybuffer); // 默认的视图大小就是buffer的大小
        
        dataview.setInt8(0,127) // 从0开始写入一个int8(8位无符号整形,一个字节)
        dataview.getInt8(0) // 从偏移0开始读取一个int8(8位无符号整形,一个字节)
        console.log(dataview.getInt8(0));
        dataview.setInt16(0,65535); // 错误超出了ArrayBuffer的空间 int16占两个字节
```

## 字节序

简单来讲-使用DataView:
- 在读写时不用考虑平台字节序问题。

> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView#%E5%AD%97%E8%8A%82%E5%BA%8F
>
> https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F

可以利用这个函数来进行判断:
```javascript
var littleEndian = (function() {
  var buffer = new ArrayBuffer(2);
  new DataView(buffer).setInt16(0, 256, true /* 设置值时使用小端字节序 */);
  // Int16Array 使用系统字节序，由此可以判断系统是否是小端字节序
  return new Int16Array(buffer)[0] === 256;
})();
console.log(littleEndian); // true or false
```

# TypedArray

> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray

在上面一节中我们使用get和set的方式基于数据类型来读写内存(ArrayBuffer)中的数据.

而所谓的TypedArray就是使用类似于操作数组的方式来操作我们的Buffer可以理解为数组中的每一个元素都是不同类型的数据,这样一来我们可以使用数组上的很多方法,相较于干巴巴的使用get和set更加灵活一些,少掉点头发.

名字叫做TypedArray的这个对象或者全局构造函数并不存在于JavaScript中.因为类型数组并不只有一个,但是TypedArray代指的这些内容拥有统一的构造函数,统一的属性统一的方法,不同的只是他们的名字以及所对应的数据类型.

```
TypedArray()指的是以下的其中之一： 

Int8Array(); 
Uint8Array(); 
Uint8ClampedArray();
Int16Array(); 
Uint16Array();
Int32Array(); 
Uint32Array(); 
Float32Array(); 
Float64Array();
```

看到这里我们立马联想到了之前DataView上不同的Get和Set,概念是一样的,不同于ArrayBuffer的是,这里的最小数据单位是数组中的元素,不同类型元素所占用的空间是不同的,但是我们不需要考虑在字节层面上进行控制.

接下来我们利用Int8Array来进行讨论:

- 构造函数
  - 传入一个数值来表示类型数组中元素的数量
  - 传入任意一个类型数组在保留其原有的长度上进行数据类型转换
- 方法(静态)
  - Int8Array.from()通过可迭代对象创建一个类型数组
  - Int8Array.of()通过可变参数创建一个类型数组

例子:

```javascript
// 32无符号能表示的最大的数值 占4个字节
const int32 = new Int32Array(1); // 使用length
int32[0] = 4294967295;

// 8位无符号能表示最大的内容是127 占1个字节
const int8 = new Int8Array(int32); // 使用另外一个类型数组
console.log(int8[0]) // -1 32位转8位要确保,32位的值在8位的范围内否则无法保证精度

const from = Int8Array.from([0,127]);
console.log(from.length === 2) // true

const of = Int8Array.of(0,127);
console.log(of.length === 2)// true
```

- 属性(静态)
  - [`TypedArray.BYTES_PER_ELEMENT`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/BYTES_PER_ELEMENT)
  - *TypedArray*.length
  - [`TypedArray.name`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/name)
  - [`get TypedArray[@@species\]`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/@@species)
  - [`TypedArray.prototype`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/prototype)
- 属性(实例)
  - [`TypedArray.prototype.buffer`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/buffer)
  - [`TypedArray.prototype.byteLength`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/byteLength)
  - [`TypedArray.prototype.byteOffset`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/byteOffset)
  - [`TypedArray.prototype.length`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/length)
- 方法(实例)
  - 方法是在是太多了Array上的方法TypedArray基本都有,例举太多都是照搬MDN,给个贴上大家自行查阅吧:joy:.
  - [方法列表](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray#%E6%96%B9%E6%B3%95_2)

例子(类数组操作):

```javascript
const int8 = new Int8Array(2);
int8[0] = 0;
int8[1] = 127;

int8.forEach((value)=>console.log(value));

for (const elem of int8) {
	console.log(elem);
}
```

# Blob

> https://developer.mozilla.org/zh-CN/docs/Web/API/Blob
>
> Blob` 对象表示一个不可变、原始数据的类文件对象。Blob 表示的不一定是JavaScript原生格式的数据

这说明了什么意思,类似于ArrayBuffer一样,ArrayBuffer本身没有为了达到某种目的而提供具体的操作方法,他的存在就类似于一个占位符一样,Blob对象也是类似的概念,在JavaScript中我们使用Blob对象来表示一个文件,当这个文件需要进行操作的时候我们在利用其他途径对这个Blob对象进行操作.(个人理解)

Blob的API和ArrayBuffer非常相似,因为他们有着非常密切的联系,创建Blob对象有两种方式,对应着两种具体的需求:

- 直接调用构造函数传入JavaScript中的数据结构
- 使用File对象创建,用于表示文件

这里我们不讨论由File对象创建的情况,这部分留到下节中讨论.

- 构造函数
  - 你可以利用现有的JavaScript数据结构来创建一个Blob对象
  - 你可以选择这个Blob对象的MIME类型
  - 你可以控制这个Blob对象中的换行符在系统中表现的行为
  - [具体参考](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/Blob)
- 属性(实例)
  - size - Blob对象所包含的数据大小
  - type - Blob对象所描述的MIME类型
- 方法(实例)
  - slice()类似于ArrayBuffer.slice()从原有的Blob中分离出一部分组成新的Blob对象

例子:

```javascript
        const blob1 = new Blob([JSON.stringify({
                content: 'success'
            })], {
                type: 'application/json'
        });

        const blob2 = new Blob(['<a id="a"><b id="b">hey!</b></a>'],{
            type:'text/html'
        });
```

**注意**:Blob对象接受的第一个参数是一个数组.

Blob对象还可以根据其他数据结构进行创建:

- ArrayBuffer
- ArrayBufferView(TypedArray)
- Blob

> https://developer.mozilla.org/zh-CN/docs/Web/API/Blob/Blob

乍一看Blob对象看似很鸡肋,不过在JavaScript中能装载数据还可以指定MIME类型,多半都是用于和外部进行交互.

回顾前面的内容,我们知道了如何创建一片内存中的区域,还知道了如何利用不同的工具来对这篇内存进行操作,最重要的一个用于描述文件Blob对象接受ArrayBuffer和TypedArray,那么还能玩出什么花样呢?

# File

> 文件（**File**）接口提供有关文件的信息，并允许网页中的 JavaScript 访问其内容。
>
> https://developer.mozilla.org/zh-CN/docs/Web/API/File

File对象用于描述文件,这个对象虽然可以利用构造函数自行创建,但是大多数情况下都是利用浏览器上的`<input>`元素拖拽API来提供的.

File对象继承Blob对象,所以继承了Blob对象上的原型方法和属性,和Blob纯粹表示文件不同,File更加接地气一点,他还拥有了我们操作系统上常见的一些特征:

- 属性(实例)
  - lastModified 最后修改时间
  - name 文件名称
  - size 文件大小
  - type MIME类型
  - [详细介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/File#Properties)
- 构造函数
  - [详细介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/File/File)

# FileReader

# 这些API在其他API中应用

// TODO URL 对象可以使用在很多地方